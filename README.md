# Playing with cert-manager and let's encrypt on DO

I've experimentet with [cert-manager](https://github.com/jetstack/cert-manager) and [ingress-nginx](https://github.com/haugom/do-ingress-nginx) on digital ocean using the loadbalancer to his ingress-nginx.

I can update add a DNS A record of *.my.domain and point it to the load balancer on digital ocean.

With 
* ingress-nginx
* cert-manager (running with let's encrypt integration)
* A DNS record pointing to DO loadbalancer

I can submit a certificate request that looks like this:

```yaml
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: my-name
  namespace: my-namespace
spec:
  secretName: tls-myTls
  issuerRef:
    name: letsencrypt-prod
  commonName: my.domain
  dnsNames:
  - my.domain
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - my.domain
    - http01:
        ingress: my-ingress
      domains:
      - my.domain
```

and an ingress-rule like this:

```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
  namespace: my-namespace
spec:
  tls:
    - hosts:
      - my.domain
      secretName: tls-myTls
  rules:
    - host: my.domain
      http:
        paths:
        - path: /
          backend:
            serviceName: my-svc
            servicePort: 80
```
With the above combination, cert-manager fetches certificate from let's encrypt and makes it ready as a tls-secret which ingress-nginx can use to display the certificate.
