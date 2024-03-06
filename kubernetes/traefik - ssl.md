## Utilize Your Custom Certificates with Traefik

### What I'm Using
- Kubernetes 1.29.2
- Traefik chart: https://github.com/traefik/traefik-helm-chart - Version: v26.1.0
- Domain: test.local


### Setting up Traefik
To integrate Traefik with your custom certificates, follow these steps:

**1. Prepare Certificates**: Begin by gathering your certificate (.crt or .pem) and private key (.key). For this example I’m storing them in \certs on my local machine.

**2. Create secrect:**
```
kubectl -n traefik-internal create secret tls test-local --key="privkey.key" --cert="fullchain.pem"

```
**3.** To add / remove TLS certificates, even when Traefik is already running, their definition can be added to created configmap named "traefik-configs"  in the [[tls.certificates]] section.

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
**4.** Delete and recreate the traefik-configs configmap in the traefik-internal namespace, and load the traefik-configs.yaml file as the dynamic configuration for Traefik.
```
kubectl -n traefik-internal delete cm traefik-configs
kubectl -n traefik-internal create cm traefik-configs --from-file traefik-configs.yaml
```
**5.** Specify values `in values.yaml`:
```
volumes:
  - name: test-local
    mountPath: "/certificates/test.local/"
    type: secret
```
**6.** Then upgrade Traefik helm chart:
```
helm upgrade -n traefik-internal traefik-internal traefik

```
**7.** You need to verify that your certificates are valid and secure:
```
openssl s_client -connect kidzy.land:443

```
