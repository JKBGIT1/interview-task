# interview-task

## Overview

A local Kubernetes infrastructure built on VirtualBox VMs, provisioned with Vagrant and KubeOne. The cluster runs a GitOps pipeline (FluxCD + ArgoCD), a service mesh (Linkerd), TLS termination (cert-manager), Gateway API routing (Traefik), DNS resolution (dnsmasq), and vulnerability scanning (Trivy).

## Architecture

### Network diagram

| VM       | IP           | Role          |
|----------|--------------|---------------|
| dns-1    | 192.168.56.2 | DNS Server    |
| lb-1     | 192.168.56.3 | Load Balancer |
| master-1 | 192.168.56.4 | Control Plane |
| worker-1 | 192.168.56.5 | Worker Node   |
| worker-2 | 192.168.56.6 | Worker Node   |

### Traffic flow

```
Host (*.app.test) → DNS Server (dnsmasq) → LB (NGINX stream proxy) → Worker NodePorts (31080/31443) → Traefik Gateway → App Pods
```

### Deployment flow

```
Vagrant → KubeOne → FluxCD → (cert-manager, Linkerd, Traefik, ArgoCD, Trivy) → ArgoCD deploys web apps
```

### Repository structure

```
.
├── .github/workflows/       # CI pipeline (Trivy vulnerability scanning)
├── k8s/
│   ├── infra/               # Infrastructure components (FluxCD-managed)
│   │   ├── argocd/          # ArgoCD deployment + app definitions
│   │   ├── cert-manager/    # TLS certificate management
│   │   ├── flux-system/     # FluxCD installation + GitOps sync
│   │   ├── linkerd/         # Service mesh with mTLS
│   │   ├── traefik/         # Gateway API controller
│   │   └── trivy/           # Runtime vulnerability scanning
│   └── web-apps/
│       └── nginx-web-app/   # Demo application with security hardening
├── kubeone/                 # KubeOne cluster configuration
├── lbs/                     # Load balancer (NGINX + Docker)
└── vagrant/                 # VM provisioning (Vagrantfile)
```

## Tool choices

The rationale behind the tool choices will be explained at the meeting.

KubeOne, FluxCD, ArgoCD, Linkerd, Traefik, Cilium, cert-manager, Trivy, dnsmasq

## Prerequisites

- **OS**: Debian-based Linux recommended (Vagrant installation commands are Debian-specific)
- **CPU**: at least 8 cores (5 VMs: 1+1+2+2+2)
- **RAM**: at least 14 GB (5 VMs: 1+1+4+4+4 GB)

## Installations

### VirtualBox

Install VirtualBox 7.2.6 following https://www.virtualbox.org/wiki/Downloads

### Vagrant

Download Vagrant by running the commands below on a debian based linux. Otherwise follow https://developer.hashicorp.com/vagrant/install#linux. and don't forget to install version 2.4.9.

```
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant=2.4.9-1
```

Verify the vagrant version running

```
vagrant -v
```

You should expect the following output

```
Vagrant 2.4.9
```

### KubeOne

Download the kubeone CLI from https://github.com/kubermatic/kubeone/releases/tag/v1.12.3

### Helm

Download helm CLI from https://github.com/helm/helm/releases/tag/v3.18.4

### Kubectl

```
$ kubectl version
Client Version: v1.33.0
Kustomize Version: v5.6.0
Server Version: v1.34.1
```

### Kustomize

Download kustomize CLI from https://github.com/kubernetes-sigs/kustomize/releases/tag/kustomize%2Fv5.4.3

### TalosOS

The show stopper https://github.com/hashicorp/vagrant/issues/13619 for TalosOS on VirtualBox using Vagrant. In a nutshell, missing support for `config.vm.communicator = :none`.

## Set up cluster

Follow the steps below once you finished installation of all required tools to build the cluster.

0. Update the /etc/systemd/resolved.conf by adding

```
DNS=192.168.56.2
```

1. `cd ./vagrant && vagrant up` - Vagrant creates VMs and sets up `/kubeone/kubeone.yaml` which will be used to spin up K8s. ATM, the downscaling doesn't work. You can scale up by changing values in `workers_count` and `masters_count`.
2. `cd ./kubeone && kubeone apply -m ./kubeone.yaml` - Spins up K8s cluster on spawned VMs from previous step.
3. `kubectl get no --kubeconfig ./kubeone/brightpick-interview-task-kubeconfig`

Expected output below

```
NAME       STATUS   ROLES           AGE   VERSION
master-1   Ready    control-plane   23m   v1.34.1
worker-1   Ready    <none>          22m   v1.34.1
worker-2   Ready    <none>          22m   v1.34.1
```

4. You have to perform the initial deployment of FluxCD manually.

```
kustomize build ./k8s/infra/flux-system | kubectl create -f -
```

If some applications aren't deployed, rerun the command above but using `kubeclt apply -f -`

Watch for the successful sync.

```
$ kubectl get kustomizations.kustomize.toolkit.fluxcd.io -n flux-system --watch
NAME    AGE   READY   STATUS
infra   11m   True    Applied revision: feat/deploy-web-app-and-set-up-gitops@sha1:e3e6327d056d123d1a80cec55438a02b62b0202e
```

It can take a while until the FluxCD deploys the applications in the `k8s/infra/`. In my case, it took around 6 minutes ever since I hit the `kubectl apply`.

5. Sync ArgoCD has been deployed by FluxCD, you can port forward its dashboard

```
kubectl port-forward service/argocd-server -n argocd 8080:443
```

6. Visit http://localhost:8080 and accept the certificate

7. Get the ArgoCD admin password running

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
8. Resolve the brightpick.app.test

```
$ dig brightpick.app.test

; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> brightpick.app.test
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63523
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;brightpick.app.test.           IN      A

;; ANSWER SECTION:
brightpick.app.test.    30      IN      A       192.168.56.3

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Feb 11 19:07:32 CET 2026
;; MSG SIZE  rcvd: 64
```

9. Access the application

```
$ curl https://brightpick.app.test
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## How to debug

If FluxCD doesn't deploy the applications in the `k8s/infra`, check the Kustomization, HelmRepositories and HelmReleases.

```

```

Then check the `kustomize-controller` logs.

```
kubectl logs -l app=kustomize-controller -n flux-system
```


## How to scale up the worker nodes

1. Increase the `workers_count` variable in the `vagrant/Vagrantfile`.
2. `vagrant up`
2. `kubeone apply -m ./kubeone/kubeone.yaml`

## How to scale down the worker nodes

Unfortunately, in this case, you have to do a bit more manual work with precision.

1. Remove the last item in the `staticWorkers.hosts` in the `kubeone/kubeone.yaml`.
2. `kubectl drain <lastly-joined-worker-node> --ignore-daemonsets --delete-emptydir-data`
3. `kubectl delete <lastly-joined-worker-node>`
4. `vagrant ssh <lastly-provisioned-worker-node>`
    - 4.1. `sudo kubeadm reset`
5. `kubeone apply -m ./kubeone/kubeone.yaml` (just in case)
6. `vagrant destroy <lastly-provisioned-worker-node>`
7. Decrease the `workers_count` variable in the `vagrant/Vagrantfile` by 1
8. `vagrant provision` (to update the `kubeone/kubeone.yaml` + `lbs/nginx/nginx.conf` and restart the nginx on LB VM)

## How to scale up/down the master nodes

Should work the same as scaling up/down the worker nodes. However, it haven't been tested.

## Teardown

1. Destroy all VMs:

```
cd ./vagrant && vagrant destroy -f
```

2. Optionally clean up generated files:

```
rm -f kubeone/kubeone.yaml kubeone/brightpick-interview-task-kubeconfig lbs/nginx/nginx.conf
```

## CI/CD

A GitHub Actions pipeline at `.github/workflows/scan-vulnerabilities.yaml` runs on every PR to `main` that touches `k8s/web-apps/**`.

It extracts container images from the Kubernetes manifests and scans each one with [Trivy](https://github.com/aquasecurity/trivy). The pipeline fails if any CRITICAL or HIGH vulnerabilities are found.

## Basic security hardening

- Check the README.md in the `k8s/web-apps/nginx-web-app` folder.
- In a nutshell, the K8s isn't secured. It's missing secure supply chain, IDS (e.g., Falco), and maybe Kyverno to enforce policies (basically the ones mentioned in the `k8s/web-apps/nginx-web-app/README.md`). In a nutshell, I have just enabled the Kube API audit logs.
- I have just disabled the root login and password authentication. The VMs used in K8s don't have any firewall, although allowing only the ports mentioned in https://kubernetes.io/docs/reference/networking/ports-and-protocols/ should work. However, I didn't want to jump into that rabbit hole. The LB and DNS server VMs have some basic firewall enforced by `ufw`.

## Known limitations

- Downscaling worker nodes requires manual steps (no automated downscaling)
- Master node scaling is untested
- TalosOS on VirtualBox blocked by [vagrant#13619](https://github.com/hashicorp/vagrant/issues/13619)
- Single master node (no HA control plane)
- No persistent storage provisioner

## Further reading

- [vagrant/README.md](vagrant/README.md) - VM provisioning details
- [kubeone/README.md](kubeone/README.md) - KubeOne cluster configuration
- [lbs/README.md](lbs/README.md) - Load balancer setup
- [k8s/infra/README.md](k8s/infra/README.md) - Infrastructure components overview
- [k8s/infra/argocd/README.md](k8s/infra/argocd/README.md) - ArgoCD deployment
- [k8s/infra/cert-manager/README.md](k8s/infra/cert-manager/README.md) - cert-manager configuration
- [k8s/infra/flux-system/README.md](k8s/infra/flux-system/README.md) - FluxCD setup
- [k8s/infra/linkerd/README.md](k8s/infra/linkerd/README.md) - Linkerd service mesh
- [k8s/infra/traefik/README.md](k8s/infra/traefik/README.md) - Traefik Gateway API
- [k8s/infra/trivy/README.md](k8s/infra/trivy/README.md) - Trivy vulnerability scanning
- [k8s/web-apps/nginx-web-app/README.md](k8s/web-apps/nginx-web-app/README.md) - Web app security hardening
