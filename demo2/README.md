# Demo 2 - Using StackRox to Check Vulnerabilities of container images

## Generate API Token within Stackrox

* Generate an API Token within Stackrox, go to Platform Configuration -> Integrations -> Authentication Tokens -> API Token and generate new API Token:

<img align="center" width="450" src="../assets/acs1.png">

* Grab the token generated, and export into the ROX_API_TOKEN variable:

```
export ROX_API_TOKEN="xxx"
```

* [Install the roxctl cli](https://docs.openshift.com/acs/3.66/cli/getting-started-cli.html#installing-cli-on-linux_cli-getting-started) and use the roxctl check image to verify if the API Token is working properly:

```
roxctl image check --insecure-skip-tls-verify --endpoint $ACS_ROUTE:443 --image quay.io/centos7/httpd-24-centos7:centos7
```

The output of the command will show that two policies are violated, so the roxctl image check is working as expected:

```
WARN:   A total of 2 policies have been violated
ERROR:  failed policies found: 1 policies violated that are failing the check
ERROR:  Policy "Fixable Severity at least Important" - Possible remediation: "Use your package manager to update to a fixed version in future builds or speak with your security team to mitigate the vulnerabilities."
ERROR:  checking image failed after 3 retries: failed policies found: 1 policies violated that are failing the check
```

NOTE: For further information check the [ACS Integration with CI Systems](https://docs.openshift.com/acs/3.70/integration/integrate-with-ci-systems.html#cli-authentication_integrate-with-ci-systems)

## Create ACS API Token Secrets for Tekton Pipeline to integrate in ACS

* To be able to authenticate from the Tekton Pipelines towards the Stackrox / ACS API, the roxctl tasks used in the pipelines needs to have both ROX_API_TOKEN (generated in one step before) and the ACS Route as well:

**Linux**
```
echo $ROX_API_TOKEN
echo $ACS_ROUTE

cat > /tmp/roxsecret.yaml << EOF
apiVersion: v1
data:
  rox_api_token: "$(echo $ROX_API_TOKEN | tr -d '\n' | base64 -w 0)"
  rox_central_endpoint: "$(echo $ACS_ROUTE:443 | tr -d '\n' | base64 -w 0)"
kind: Secret
metadata:
  name: roxsecrets
  namespace: ${NAMESPACE}
type: Opaque
EOF

kubectl apply -f /tmp/roxsecret.yaml
```

**MacOS**
```
cat > /tmp/roxsecret.yaml << EOF
apiVersion: v1
data:
  rox_api_token: "$(echo $ROX_API_TOKEN | tr -d '\n' | base64)"  
  rox_central_endpoint: "$(echo $ACS_ROUTE:443 | tr -d '\n' | base64)"  
kind: Secret
metadata:
  name: roxsecrets
  namespace: ${NAMESPACE}
type: Opaque
EOF
```

TODO: check the -w0 because base64 adds one new line

* Check the [roxctl cli docs](https://docs.openshift.com/acs/3.70/cli/getting-started-cli.html#cli-authentication_cli-getting-started) for more information.