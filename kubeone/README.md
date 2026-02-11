# kubeone

In order to avoid setting up the VMs for K8s, running `kubeadm init/join`, installing CNI and other inconveniences, I've leveraged the `kubeone`. In a nutshell, it spins up the K8s cluster on the VMs provisoned by vagrant.

I've enabled K8s auditing to static files, since the `nginx-web-app` namespace uses auditing thanks to `pod-security.kubernetes.io/audit: restricted`. The `audit-policy.yaml` was taken from https://oneuptime.com/blog/post/2026-01-30-kubernetes-audit-policies/view

The Kube API is exposed on the master's node IP. That being said, even if you scale up the control plane (I haven't tried that on my own), the generated kubeconfig can interact with Kube API only on the first master node.

After successful run of `kubeone apply -m ./kubeone.yaml`, you should see the kubeconfig in the this folder named as `brightpick-interview-task-kubeconfig`.
