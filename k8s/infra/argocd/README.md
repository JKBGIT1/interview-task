# argocd

The `apps` folder contains the ArgoCD applications that deploy the applications defined in `k8s/web-apps`. 

The `deploy.yaml` deploys ArgoCD HelmRepository and HelmRelease (managed by FluxCD). I couldn't deploy the ArgoCD CRDs by HelmRelease because the ArgoCD applications defined in the `k8s/web-apps` uses the `argoproj.io/v1alpha1` Application. As a result, the FluxCD couldn't reconcile due to the missing CRD. As a workaround, I'm depoying the ArgoCD CRDs using the `crds.yaml`.
