# argocd

The `apps` folder contains the ArgoCD applications deploying the applications in `k8s/web-apps`. I couldn't deploy the ArgoCD apps directly (in the `kustomization.yaml` together with the `deploy.yaml`), because the Helm Chart defined in this folder haven't deployed the CRDs yet. As a result, the FluxCD reconciliation has been failing. 

As a workaround I'm deploying the ArgoCD CRDs separately referencing the https://raw.githubusercontent.com/argoproj/argo-cd/refs/tags/v3.3.0/manifests/crds/application-crd.yaml in the `kustomization.yaml` `resources`.
