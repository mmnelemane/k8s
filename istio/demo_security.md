# This file highlights security concepts for Istio and lists steps for installation and operation with Istio
# Tha attempt is to be able to demonstrate key security features of Istio.

** Setting up Security for Istio

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

** An Example


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

* Partial

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

* Legacy

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

* kube injection

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

* Verify setup

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

*  Check default policies

```
$ kubectl get policies.authentication.istio.io --all-namespaces
NAMESPACE      NAME                          AGE
istio-system   grafana-ports-mtls-disabled   65m

$ kubectl get meshpolicies -o yaml | grep ' mode'
        mode: PERMISSIVE
```

