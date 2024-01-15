# Ingress-nginx controller Installation

## Installation using Helm

- helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginxs
- helm repo update
- helm install nginx-tcp ingress-nginx/ingress-nginx -n ingress-nginx-tcp --create-namespace

## Setup externalIPs

- Define externalIPs at `values.yml`

## Expose TCP connections

> Ref: <https://mailazy.com/blog/exposing-tcp-udp-services-ingress/>

- Expose tcp connections with syntax:

  `<expose_port>`:`<namespace>`/`<service>`:`<service_port>`

- For example:

  ```yaml
  tcp:
    1883: default/emqx:1883
    6379: ot-operators/redis-replication:6379
  ```

- Apply configuration

  ```bash
     helm upgrade --install --create-namespace -n ingress-nginx-tcp nginx-tcp ingress-nginx/ingress-nginx --values values.yml
  ```

- Verify

  ```bash
  kubectl -n ingress-nginx-tcp get svc
  ```

- Uninstall
  ```bash
    helm uninstall -n ingress-nginx-tcp nginx-tcp
  ```
