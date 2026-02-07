# envoy

The Helm Chart deploys Envoy Gateway API's controller and CRDs. The rest of the `deploy.yaml` deploys EnvoyProxy as DaemonSet using the `hostNetwork`, GatewayClass targeting this EnvoyProxy and Gateway exposing everything on HTTP port using the aforementioned GatewayClass.
