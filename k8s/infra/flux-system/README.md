# flux-system

The `install.yaml` file was taken from https://github.com/fluxcd/flux2/releases/download/v2.7.5/install.yaml.

The `main-repo-sync.yaml` deploys the GitRepository CR (monitoring this repo), and Kustomization CR (deploying the application defined in `k8s/infra/kustomization.yaml` to K8s).
