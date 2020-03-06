This file highlights security concepts for Istio and lists steps for installation and operation with Istio
The attempt is to be able to demonstrate key security features of Istio.

## Setting up Security for Istio

```
$ istioctl manifest apply --set profile=demo --set values.global.mtls.auto=true --set values.global.mtls.enabled=false
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Citadel...
- Applying manifest for component EgressGateway...
- Applying manifest for component Policy...
- Applying manifest for component Prometheus...
- Applying manifest for component Kiali...
- Applying manifest for component IngressGateway...
- Applying manifest for component Galley...
- Applying manifest for component Injector...
- Applying manifest for component Telemetry...
- Applying manifest for component Tracing...
- Applying manifest for component Pilot...
- Applying manifest for component Grafana...
✔ Finished applying manifest for component Kiali.
✔ Finished applying manifest for component Prometheus.
✔ Finished applying manifest for component Citadel.
✔ Finished applying manifest for component Policy.
✔ Finished applying manifest for component IngressGateway.
✔ Finished applying manifest for component Pilot.
✔ Finished applying manifest for component Tracing.
✔ Finished applying manifest for component Galley.
✔ Finished applying manifest for component Injector.
✔ Finished applying manifest for component EgressGateway.
✔ Finished applying manifest for component Grafana.
✔ Finished applying manifest for component Telemetry.


✔ Installation complete
```

## An Example


* Full

```
$ kubectl create ns full
namespace/full created

$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n full 
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created


$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n full              
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
```

##Partial

```
$ kubectl create ns partial
namespace/partial created

$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n partial
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created


$ cat << EOF | kubectl apply -n partial -f -
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: httpbin-nosidecar
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: httpbin
>   template:
>     metadata:
>       labels:
>         app: httpbin
>         version: nosidecar
>     spec:
>       containers:
>       - image: docker.io/kennethreitz/httpbin
>         imagePullPolicy: IfNotPresent
>         name: httpbin
>         ports:
>         - containerPort: 80
> EOF
deployment.apps/httpbin-nosidecar created
```

## Legacy

```
$  kubectl create ns legacy
namespace/legacy created

$ kubectl apply -f samples/httpbin/httpbin.yaml -n legacy
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created

$ kubectl apply -f samples/sleep/sleep.yaml -n legacy
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
```

## kube injection

```
$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n full
serviceaccount/sleep unchanged
service/sleep unchanged
deployment.apps/sleep configured

$ kubectl apply -f samples/sleep/sleep.yaml -n legacy
serviceaccount/sleep unchanged
service/sleep unchanged
deployment.apps/sleep unchanged
```

## Verify setup

```
$ kubectl get pods -n full
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-7fbcc7bc74-h87qk   2/2     Running   0          5m25s
sleep-6b7b8df4bc-m6rmw     2/2     Running   0          5m9s

$ kubectl get pods -n partial
NAME                                 READY   STATUS    RESTARTS   AGE
httpbin-7fbcc7bc74-hsm8s             2/2     Running   0          4m14s
httpbin-nosidecar-6bc65f66df-cjnlc   1/1     Running   0          3m47s

$ kubectl get pods -n legacy
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-64776bf78d-cd9ll   1/1     Running   0          3m15s
sleep-666475687f-tc4cr     1/1     Running   0          3m3s
```

## Check default policies

```
$ kubectl get policies.authentication.istio.io --all-namespaces
NAMESPACE      NAME                          AGE
istio-system   grafana-ports-mtls-disabled   65m

$ kubectl get meshpolicies -o yaml | grep ' mode'
        mode: PERMISSIVE
```



#############################################################
# Verifying Security features using mTLS on istio
#############################################################

```
$ istioctl manifest apply --set values.global.mtls.enabled=true,values.security.selfSigned=false
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Citadel...
- Applying manifest for component Galley...
- Applying manifest for component Policy...
- Applying manifest for component Injector...
- Applying manifest for component Prometheus...
- Applying manifest for component IngressGateway...
- Applying manifest for component Pilot...
- Applying manifest for component Telemetry...
- Pruning objects for disabled component Tracing...
- Pruning objects for disabled component EgressGateway...
- Pruning objects for disabled component Grafana...
- Pruning objects for disabled component Kiali...
✔ Finished pruning objects for disabled component EgressGateway.
✔ Finished applying manifest for component Citadel.
✔ Finished pruning objects for disabled component Grafana.
✔ Finished pruning objects for disabled component Kiali.
✔ Finished applying manifest for component IngressGateway.
✔ Finished applying manifest for component Prometheus.
✔ Finished applying manifest for component Policy.
✔ Finished applying manifest for component Injector.
✔ Finished applying manifest for component Galley.
✔ Finished applying manifest for component Pilot.
✔ Finished applying manifest for component Telemetry.
✔ Finished pruning objects for disabled component Tracing.


✔ Installation complete
```

```
$ kubectl delete secret istio.default
secret "istio.default" deleted

$ kubectl label namespace default istio-injection=enabled
namespace/default labeled

$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created

$ kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.102.23.249    <none>        9080/TCP   20s
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    14d
productpage   ClusterIP   10.110.128.226   <none>        9080/TCP   14s
ratings       ClusterIP   10.109.97.209    <none>        9080/TCP   19s
reviews       ClusterIP   10.102.95.183    <none>        9080/TCP   18s

$ kubectl get pods
NAME                             READY   STATUS            RESTARTS   AGE
details-v1-c5b5f496d-jflzm       0/2     PodInitializing   0          25s
productpage-v1-c7765c886-jsx8w   0/2     PodInitializing   0          17s
ratings-v1-f745cf57b-6pdkm       0/2     PodInitializing   0          25s
reviews-v1-75b979578c-wz2ff      0/2     PodInitializing   0          22s
reviews-v2-597bf96c8f-jhsjc      0/2     PodInitializing   0          22s
reviews-v3-54c6c64795-p6tcq      0/2     PodInitializing   0          21s

$ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>

$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

```
$ kubectl get gateway
NAME               AGE
bookinfo-gateway   6s

$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.111.91.69   <pending>     15020:31172/TCP,80:30857/TCP,443:32432/TCP,15029:32247/TCP,15030:30302/TCP,15031:31817/TCP,15032:30876/TCP,15443:30942/TCP   14d

$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ echo $INGRESS_PORT
30857
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
$ export INGRESS_HOST=$(minikube ip)

$ minikube ip
192.168.39.147

$ echo $INGRESS_HOST
192.168.39.147

$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

$ echo $GATEWAY_URL
192.168.39.147:30857
```

```
$ curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

```
$ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
```


```
$ kubectl get destinationrules -o yaml
apiVersion: v1
items:
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"details","namespace":"default"},"spec":{"host":"details","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"}]}}
    creationTimestamp: "2020-03-04T10:31:20Z"
    generation: 1
    name: details
    namespace: default
    resourceVersion: "255107"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/details
    uid: 49c7348a-5e03-11ea-b4e6-a8b61109fa9e
  spec:
    host: details
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"productpage","namespace":"default"},"spec":{"host":"productpage","subsets":[{"labels":{"version":"v1"},"name":"v1"}]}}
    creationTimestamp: "2020-03-04T10:31:20Z"
    generation: 1
    name: productpage
    namespace: default
    resourceVersion: "255103"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/productpage
    uid: 49a74c41-5e03-11ea-b4e6-a8b61109fa9e
  spec:
    host: productpage
    subsets:
    - labels:
        version: v1
      name: v1
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"host":"ratings","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v2-mysql"},"name":"v2-mysql"},{"labels":{"version":"v2-mysql-vm"},"name":"v2-mysql-vm"}]}}
    creationTimestamp: "2020-03-04T10:31:20Z"
    generation: 1
    name: ratings
    namespace: default
    resourceVersion: "255106"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/ratings
    uid: 49c0cb12-5e03-11ea-b4e6-a8b61109fa9e
  spec:
    host: ratings
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v2-mysql
      name: v2-mysql
    - labels:
        version: v2-mysql-vm
      name: v2-mysql-vm
- apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"host":"reviews","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v3"},"name":"v3"}]}}
    creationTimestamp: "2020-03-04T10:31:20Z"
    generation: 1
    name: reviews
    namespace: default
    resourceVersion: "255104"
    selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/reviews
    uid: 49b36ffc-5e03-11ea-b4e6-a8b61109fa9e
  spec:
    host: reviews
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v3
      name: v3
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```


```
$ kubectl get policies.authentication.istio.io --all-namespaces
NAMESPACE      NAME                          AGE
istio-system   grafana-ports-mtls-disabled   13d
```

```
$ istioctl manifest apply --set profile=demo --set values.global.mtls.auto=true --set values.global.mtls.enabled=false
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Prometheus...
- Applying manifest for component Kiali...
- Applying manifest for component EgressGateway...
- Applying manifest for component Citadel...
- Applying manifest for component Galley...
- Applying manifest for component IngressGateway...
- Applying manifest for component Injector...
- Applying manifest for component Tracing...
- Applying manifest for component Pilot...
- Applying manifest for component Policy...
- Applying manifest for component Telemetry...
- Applying manifest for component Grafana...
✔ Finished applying manifest for component Prometheus.
✔ Finished applying manifest for component Citadel.
✔ Finished applying manifest for component IngressGateway.
✔ Finished applying manifest for component Galley.
✔ Finished applying manifest for component Policy.
✔ Finished applying manifest for component Pilot.
✔ Finished applying manifest for component EgressGateway.
✔ Finished applying manifest for component Kiali.
✔ Finished applying manifest for component Injector.
✔ Finished applying manifest for component Grafana.
✔ Finished applying manifest for component Tracing.
✔ Finished applying manifest for component Telemetry.


✔ Installation complete
```


```
$ kubectl get destinationrules.networking.istio.io --all-namespaces -o yaml | grep "host:"
    host: details
    host: productpage
    host: ratings
    host: reviews
    host: kubernetes.default.svc.cluster.local
    host: '*.local'
    host: '*.global'
    host: istio-policy.istio-system.svc.cluster.local
    host: istio-telemetry.istio-system.svc.cluster.local

$ kubectl exec $(kubectl get pod -l app=sleep -n full -o jsonpath={.items..metadata.name}) -c sleep -n full -- curl http://httpbin.full:8000/headers  -s  -w "response %{http_code}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$'
URI=spiffe://cluster.local/ns/full/sa/sleep
response 200
```


```
$ for from in "full" "legacy"; do for to in "full" "partial" "legacy"; do echo "sleep.${from} to httpbin.${to}";kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/headers  -s  -w "response code: %{http_code}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$';  echo -n "\n"; done; done
sleep.full to httpbin.full
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.partial
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.legacy
response code: 503
\nsleep.legacy to httpbin.full
response code: 200
\nsleep.legacy to httpbin.partial
response code: 200
\nsleep.legacy to httpbin.legacy
response code: 200
```


```
$ for i in `seq 1 10`; do kubectl exec $(kubectl get pod -l app=sleep -n full -o jsonpath={.items..metadata.name}) -c sleep -nfull  -- curl http://httpbin.partial:8000/headers  -s  -w "response code: %{http_code}e}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$';  echo -n "\n"; done
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nURI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\n
```


```
$ cat <<EOF | kubectl apply -n full -f -
> apiVersion: "authentication.istio.io/v1alpha1"
> kind: "Policy"
> metadata:
>   name: "httpbin"
> spec:
>   targets:
>   - name: httpbin
>   peers:
>   - mtls: {}
> EOF
policy.authentication.istio.io/httpbin created
```


```
$ for from in "full" "legacy"; do for to in "full" "partial" "legacy"; do echo "sleep.${from} to httpbin.${to}";kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/headers  -s  -w "response code: %{http_code}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$';  echo -n "\n"; done; done
sleep.full to httpbin.full
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.partial
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.legacy
response code: 503
\nsleep.legacy to httpbin.full
response code: 000
command terminated with exit code 56
\nsleep.legacy to httpbin.partial
response code: 200
\nsleep.legacy to httpbin.legacy
response code: 200
\n
```

```
$ cat <<EOF | kubectl apply -n full -f -
> apiVersion: "authentication.istio.io/v1alpha1"
> kind: "Policy"
> metadata:
>   name: "httpbin"
> spec:
>   targets:
>   - name: httpbin
> EOF
policy.authentication.istio.io/httpbin configured
```

```
$ for from in "full" "legacy"; do for to in "full" "partial" "legacy"; do echo "sleep.${from} to httpbin.${to}";kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/headers  -s  -w "response code: %{http_code}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$';  echo -n "\n"; done; done
sleep.full to httpbin.full
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.partial
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.legacy
response code: 503
\nsleep.legacy to httpbin.full
response code: 200
\nsleep.legacy to httpbin.partial
response code: 200
\nsleep.legacy to httpbin.legacy
response code: 200
\n
```

```
$ cat <<EOF | kubectl apply -n full -f -
> apiVersion: "networking.istio.io/v1alpha3"
> kind: "DestinationRule"
> metadata:
>   name: "httpbin-full-mtls"
> spec:
>   host: httpbin.full.svc.cluster.local
>   trafficPolicy:
>     tls:
>       mode: ISTIO_MUTUAL
> EOF
destinationrule.networking.istio.io/httpbin-full-mtls created
```

```
$ for from in "full" "legacy"; do for to in "full" "partial" "legacy"; do echo "sleep.${from} to httpbin.${to}";kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl http://httpbin.${to}:8000/headers  -s  -w "response code: %{http_code}\n" | egrep -o 'URI\=spiffe.*sa/[a-z]*|response.*$';  echo -n "\n"; done; done
sleep.full to httpbin.full
response code: 503
\nsleep.full to httpbin.partial
URI=spiffe://cluster.local/ns/full/sa/sleep
response code: 200
\nsleep.full to httpbin.legacy
response code: 503
\nsleep.legacy to httpbin.full
response code: 200
\nsleep.legacy to httpbin.partial
response code: 200
\nsleep.legacy to httpbin.legacy
response code: 200
```
