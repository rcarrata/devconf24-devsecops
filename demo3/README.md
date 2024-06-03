# Demo 3 - Using Cosign to Check vulnerability in Container Images

## Quay.io Creds Setup

```bash
USERNAME=<Robot_Account_Username>
QUAY_TOKEN=<Robot_Account_Token>
```

## Generate Sigstore KeyPairs

* Generate a pki key-pair for signing with cosign:

```bash
export COSIGN_PASSWORD="changeme"
cosign generate-key-pair k8s://${NAMESPACE}/cosign

kubectl get secret -n ${NAMESPACE} cosign -o jsonpath="{.data.cosign\.key}" | base64 -d >> cosign.key
```

## Testing Cosign

* Pull image from quay.io (public repo), and push it to the personal one:

```bash
podman pull quay.io/centos7/httpd-24-centos7:20220713
podman tag quay.io/centos7/httpd-24-centos7:20220713 quay.io/${USERNAME}/centos7/httpd-24-centos7:0.1
podman push  quay.io/${USERNAME}/httpd-24-centos7:0.1
```

* Sign and verify the container image signed with the Sigstore Keypairs:

```bash
cosign sign --key cosign.key quay.io/${USERNAME}/httpd-24-centos7:0.1

cosign verify --key cosign.pub quay.io/${USERNAME}/httpd-24-centos7:0.1
```

