# trivy

The static vulnerabilities scanning isn't enough because some CVEs can be found only during the runtime of the application. Also, the `.github/scan-vulnerabilities.yaml` pipeline scans only the images in the `k8s/web-apps` folder.

The `deploy.yaml` was taken from https://raw.githubusercontent.com/aquasecurity/trivy-operator/refs/tags/v0.29.0/deploy/static/trivy-operator.yaml.

Once FluxCD deploys the trivy-operator it will take some time until the trivy scans all images used in the K8s cluster. Therefore, you will see a lot of `scan-vulnerability-report-<hash>-<hash>` in the `trivy-system` namespace at first.

Once the vulnerability reports finish, run the command below to see them all.

```
kubeclt get vulnerabilityreports.aquasecurity.github.io -A
```

You can pick one and run the command below to see the details.

```
kubeclt describe vulnerabilityreports.aquasecurity.github.io -n <vulnerability-report-namespace> <vulnerability-report-name>
```

The trivy operator is deployed by FluxCD
