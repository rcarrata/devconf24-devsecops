# Demo 1 - Installing DevSecOps tooling with GitOps in ARO

## Install OpenShift GitOps and OpenShift Pipelines

* Install OpenShift Pipelines / Tekton:

```bash
until kubectl apply -k bootstrap/; do sleep 2; done
```

* After couple of minutes check the OpenShift GitOps and Pipelines:

```
ARGOCD_ROUTE=$(kubectl get route openshift-gitops-server -n openshift-gitops -o jsonpath='{.spec.host}{"\n"}')
curl -ks -o /dev/null -w "%{http_code}" https://$ARGOCD_ROUTE
```

## Install Stackrox / RHACS using GitOps

Use GitOps to install Stackrox / ACS, using the [ACS GitOps repository](https://github.com/rcarrata/rhacs-gitops/tree/main/apps/acs) as the Git repository used in this example:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: acs-install
  namespace: openshift-gitops
spec:
  destination:
    namespace: openshift-gitops
    server: https://kubernetes.default.svc
  project: default
  source:
    path: apps/acs
    repoURL: https://github.com/rcarrata/acs-gitops
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    - PruneLast=true
EOF
```

* After the Argo App is fully synched and finished properly, check the Stackrox / ACS route:

```
ACS_ROUTE=$(k get route -n stackrox central -o jsonpath='{.spec.host}')

curl -Ik https://${ACS_ROUTE}
```

NOTE: Check that you're getting a 200.

## Quay.io Repository Setup

* Add a Public Quay Repository in Quay.io (I've used pipelines-vote-api repository):

<img align="center" width="650" src="../assets/quay1.png">

* In Settings, Add Robot Account and assign Write or Admin permissions to this Quay Repository

<img align="center" width="650" src="../assets/quay2.png">

* Grab the QUAY_TOKEN and the USERNAME that is provided:

```
USERNAME=<Robot_Account_Username>
QUAY_TOKEN=<Robot_Account_Token>
```

## Configure Quay creds and RBAC

* Export the token for the Quay Registry:

```bash
export QUAY_TOKEN=""
export EMAIL="xxx"
export USERNAME="rcarrata+acs_integration"
export NAMESPACE="demo-sign"
```

* Create the namespace for the demo-sign:

```bash
kubectl create ns ${NAMESPACE}
```

* Generate a docker-registry secret with the credentials for Quay Registry to push/pull the images and signatures:

```bash
kubectl create secret docker-registry regcred --docker-server=quay.io --docker-username=${USERNAME} --docker-email=${EMAIL}--docker-password=${QUAY_TOKEN} -n ${NAMESPACE}
```

* Add the imagePullSecret to the ServiceAccount “pipeline” in the namespace of the demo:

```
export SERVICE_ACCOUNT_NAME=pipeline
kubectl patch serviceaccount $SERVICE_ACCOUNT_NAME \
 -p "{\"imagePullSecrets\": [{\"name\": \"regcred\"}]}" -n $NAMESPACE

oc secrets link pipeline regcred -n demo-sign
#oc secrets link default regcred -n demo-sign
```

## Deploy Tekton Pipeline and Tasks

* Deploy all the Tekton Tasks and Pipelines for run this demo:

```bash
kubectl apply -k manifests/
```