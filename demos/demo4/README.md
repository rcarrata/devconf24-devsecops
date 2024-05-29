# Demo 4 - DevSecOps Image Signature Verification Pipeline

## Add Signature Integration within Stackrox/ACS

* Add the Cosign signature into the Stackrox / ACS integrations. Go to Integrations, Signature, New Integration and add the following:

```
Integration Name - Cosign-signature
Cosign Public Name - cosign-pubkey
Cosign Key Value - Content of cosign.pub generated before
```

<img align="center" width="450" src="../assets/acs2.png">

* For more information around this check the [Stackrox / ACS official guide around signature verification](https://docs.openshift.com/acs/3.70/operating/verify-image-signatures.html#configure-signature-integration_verify-image-signatures)

## Add Policy Image Signature Verification

* Create the ACS Policy for the image verification importing the Trusted Signature Image Policy json into the ACS console. Go to Platform Configuration -> Policy Management -> Import Policy.

* Copy and paste the content of the [ACS Policy pre-generated](https://raw.githubusercontent.com/rcarrata/ocp4-network-security/sign-acs/sign-images/policies/signed-image-policy.json) (or upload the json file):

<img align="center" width="450" src="../assets/acs3.png">

* After imported, check the policy generated and select the response method as Inform and enforce:

<img align="center" width="650" src="../assets/acs4.png">

* In the policy scope restrict the Policy Scope of the Policy to the specific cluster and namespace (in my case demo-sign) and save the policy:

<img align="center" width="650" src="../assets/acs5.png">

* Check with the roxctl image check command the new image against the cosign public key generated:

```
roxctl --insecure-skip-tls-verify image check --endpoint $ACS_ROUTE:443 --image  quay.io/centos7/httpd-24-centos7:centos7 | grep -A2 Trusted
| Trusted_Signature_Image_Policy |   HIGH   |      -       | Alert on Images that have not  |      - Image signature is      | All images should be signed by |
|                                |          |              |          been signed           |           unverified           |   our cosign-demo signature    |
+--------------------------------+----------+--------------+--------------------------------+--------------------------------+--------------------------------+
```

Now the new policy is generating an alert in our Stackrox / ACS cluster, checking that the image is not signed with the Cosign public key that we defined before.

* For more information check the [Stackrox / ACS Security Policy guide](https://docs.openshift.com/acs/3.70/operating/manage-security-policies.html#create-policy-from-system-policies-view_manage-security-policies)

## Integrate and configure Quay.io registry into ACS

* For Quay.io use the Generic Docker Integration integration to add Quay registry credentials into Stackrox / ACS:

<img align="center" width="350" src="../assets/quay4.png">

* For more information around integrate Image Registries such as Quay into ACS check the [Integration with Image Registries guide](https://docs.openshift.com/acs/3.70/integration/integrate-with-image-registries.html#manual-configuration-image-registry-ocp_integrate-with-image-registries) in ACS docs.

## Run Signed Pipeline

* Run the pipeline for build the image, push to the Quay registry, sign the image with cosign, push the signature of the image to the Quay registry:

```bash
kubectl create -f run/sign-images-pipelinerun.yaml
```

* This pipeline will deploy the signed image and also will be validated against ACS/Stackrox System policy:

```bash
k get deploy -n workshop pipelines-vote-api
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
pipelines-vote-api   1/1     1            1           29h
```

* The steps will be as depicted below:

<img align="center" width="770" src="../assets/signed-1.png">

- Clone Repository
- Build Image and Push to Quay
- Sign the image with the Cosign Private Key and push the signature the Quay
- Deploy the k8s deployment for the application
- Update the k8s deployment for the application with the signed image

* Check in Quay that effectively the image is signed properly as you can check in the signature:

<img align="center" width="870" src="../assets/signed-2.png">