# interview-task

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

### Kubectl

### Kustomize

### TalosOS

Put aside the Talos OS on VirtualBox spawned by Vagrant due to the missing support for `config.vm.communicator = :none` https://github.com/hashicorp/vagrant/issues/13619

## Set up cluster

Follow the steps below once you finished installation of all required tools to build the cluster.

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
kustomize build ./k8s/infra/flux-system | k apply -f -
```

Watch first successful sync

```
$ k get kustomizations.kustomize.toolkit.fluxcd.io -n flux-system --watch
NAME    AGE   READY   STATUS
infra   11m   True    Applied revision: feat/deploy-web-app-and-set-up-gitops@sha1:e3e6327d056d123d1a80cec55438a02b62b0202e
```

5. Sync ArgoCD has been deployed by FluxCD, you can port forward its dashboard

```
kubectl port-forward service/argocd-server -n argocd 8080:443
```

6. Visit http://localhost:8080 and accept the certificate

7. Get the ArgoCD admin password running

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
8. Update the /etc/systemd/resolved.conf by adding

```
DNS=192.168.56.2
```

You can now resolve the brightpick.app.test without using the `@192.168.56.2`

```
$ dig brightpick.app.test
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> brightpick.app.test
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43636
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;brightpick.app.test.           IN      A

;; ANSWER SECTION:
brightpick.app.test.    24      IN      A       192.168.56.5
brightpick.app.test.    24      IN      A       192.168.56.4

;; Query time: 1 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sun Feb 08 09:32:20 CET 2026
;; MSG SIZE  rcvd: 80
```

9. Access the application

```
$ curl http://brightpick.app.test:30080
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
