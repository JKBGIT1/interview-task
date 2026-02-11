# vagrant

Vagrant by default provisions 3 VMs for K8s, 1 VM for DNS server and 1 VM for LB. It uses VirtualBox as an underlying technology for virtualization.

`Vagrantfile` is configured to automatically update `kubeone/kubeone.yaml` and `nginx/nginx.conf` based on the number of provisioned VMs.

By default, the root login and password authentication to VMs' SSH is forbiden.

In the case of DNS server and the LB VM, Vagrant sets up some basic firewall using `ufw`. On VM used by DNS server vagrant installs and sets up the DNS server leveraging `dnsmasq`. The `*.app.test` records points to LB VM. On VM used as a LB vagrant installs docker and starts NGINX container defined in the `/lbs/docker-compose.yaml`
