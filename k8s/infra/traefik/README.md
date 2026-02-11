# traefik

Traefik runs as DaemonSet in K8s. It serves as a Gateway API controller to expose the applications outside the K8s cluster. The traefik pods are exposed outside the K8s cluster thanks to the NodePort Service (`31080` HTTP and `31443` HTTPS). The NodePort service is targeted by the NGINX running on the LB VM.

The traefik is deployed by Helm Chart thanks to HelmRepository and HelmRelease managed by FluxCD.
