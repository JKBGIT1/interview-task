# trivy

The static vulnerabilities scanning isn't enough because some CVEs can be found only during the runtime of the application. Also, the `.github/scan-vulnerabilities.yaml` pipeline scans only the images in the `k8s/web-apps` folder.

The `deploy.yaml` was taken from https://raw.githubusercontent.com/aquasecurity/trivy-operator/refs/tags/v0.29.0/deploy/static/trivy-operator.yaml
