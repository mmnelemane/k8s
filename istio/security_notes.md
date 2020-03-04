SSL/TLS related settings for upstream connections (common to both HTTP and TCP upstreams)

# Example: Configuring a client to use mTLS for connections to a database cluster

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: db-mtls
spec:
  host: mydbserver.prod.svc.cluster.local
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```


