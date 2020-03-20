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


