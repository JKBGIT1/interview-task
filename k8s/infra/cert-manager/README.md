# cert-manager

The cert-manager is needed to generate the certificates used by Linkerd for mTLS (although I haven't set it up for any application in K8s) and to generate TLS certificate to expose the `nginx-web-app` on HTTPS.

Currently deployed as a YAML manifest in https://github.com/cert-manager/cert-manager/releases/tag/v1.19.3 assets.

I had to patch the cert-manager Deployment to support Gateway API.
