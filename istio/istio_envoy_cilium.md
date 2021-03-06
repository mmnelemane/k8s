Istio in summary provides the following functionality for an Envoy-Kubernetes-System.

Ingress -> Config -> Check & Report -> Secure -> Egress

# Cilium and BPF for Istio

* Istio abstracts networking at Layer 7 (Application protocol layer)
   - Cilium datapath allows L7 functionality in the kernel
* Multiple levels of Integration is possible between Istio and Cilium
  1. As a simple policy offload irrespective service management proxy.
  2. As a combination of Service Management with policy offload.

* Istio runs Envoy in a sidecar configuration inside of the application pod
* Cilium runs Envoy outside of the application pod and configures separate listeners for individual pods.

# Istio Advantages for Cilium:

* Auth and Identity enforcement on top of Cilium using Certificates on Network polices.
* Export Telemetry from Cilium to Istio
* Offload Mixer functionality in Cilium thereby reducing proxy-related latency.

# Few useful links:

* Blogs about Cilium and Istio: 
  - https://cilium.io/blog/istio/
  - https://cilium.io/blog/2018/08/07/istio-10-cilium/

* A paper on kTLS  (accelerating service mesh security): 
  - https://netdevconf.info/1.2/papers/ktls.pdf
    
* Minikube setup for Cilium as policy offload for Istio:
  - https://docs.cilium.io/en/stable/gettingstarted/istio/

