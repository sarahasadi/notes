## Use Your Own Certificates with Traefik

### What I'm Using
- Kubernetes 1.29.2
- Traefik chart: https://github.com/traefik/traefik-helm-chart
- Version: v26.1.0
- Domain: test.local


### Setting up Traefik
First things, you’ll need your certificate (.crt or pem) and private key (.key). For this example I’m storing them in \certs on my local machine.

Next create secrect file:
```
kubectl -n traefik-internal create secret tls test-local --key="privkey.key" --cert="fullchain.pem"

```
To add / remove TLS certificates, even when Traefik is already running, their definition can be added to created configmap named "traefik-configs"  in the [[tls.certificates]] section.

```
insecureSkipVerify: true

tls:
  stores:
    default:
      defaultCertificate:
        certFile: /certs/tls.crt
        keyFile: /certs/tls.key
  certificates:
    - certFile: /certificates/test.local/tls.crt
      keyFile: /certificates/test.local/tls.key
```
Delete and recreate the traefik-configs configmap in the traefik-internal namespace, and load the traefik-configs.yaml file as the dynamic configuration for Traefik.
```
kubectl -n traefik-internal delete cm traefik-configs
kubectl -n traefik-internal create cm traefik-configs --from-file traefik-configs.yaml
```
Used these values in values.yaml
```
volumes:
  - name: test-local
    mountPath: "/certificates/test.local/"
    type: secret
```
Then upgrade Traefik helm chart:
```
helm upgrade -n traefik-internal traefik-internal traefik

```
You need to verify that your certificates are valid and secure:
```
openssl s_client -connect kidzy.land:443

```
