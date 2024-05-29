# Demo 5 -  "Hacking" Tekton Pipeline with unsigned Container Images

## Run Unsigned Pipeline

* Run the pipeline for build the image and push to the Quay registry, but this time without sign with cosign private key:

```bash
kubectl create -f run/unsigned-images-pipelinerun.yaml
```

* The pipeline will fail because will detect that a unsigned image is used for the deployment:

<img align="center" width="570" src="../assets/unsigned-1.png">

* As we can see in the logs, the step of check-image failed, because Stackrox / ACS policy blocked the pipeline due to a policy failure (enforced by the Trusted Signature Image Policy).

<img align="center" width="770" src="../assets/unsigned-2.png">

* As you can check in the pipeline we have the full output of the image check with the rationale of the policy violation:

<img align="center" width="870" src="../assets/unsigned-3.png">