# lbs

The content of this folder is mounted to the VM used as a LB. `docker-compose.yaml` starts NGINX. NGINX routes traffic on ports 80 and 443 to the Traefik running on the K8s worker nodes.

The `nginx/nginx.conf` file is automatically updated based on the `nginx/nginx.conf.erb` (template) by running vagrant cmds (e.g., `vagrant up`, `vagrant provision`).
