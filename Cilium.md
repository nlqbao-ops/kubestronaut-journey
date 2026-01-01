#### architecture
- role of Cilium in k8s env
- cilium architecture
- IP address mgmt with cilium
- cilium component roles
- datapath model

#### installation and conifg
- cilium CLI to query and modify config
- cilium CLI to install cilium, run connectivity tests, monitor status

#### netpol
- interpret cilium netpol and intent
- cilium's identity-based network security model 
- policy enforcement modes 
- policy rule structure 
- k8s network policies vs cilium network policies 

#### cluster mesh 
- understand benefit for cluster mesh for multi-cluster connectivity 
- achieve service discovery and load balancing across cluster with cluster mesh

#### service mesh
- how to use ingress/gateway api for ingress routing 
- service mesh use cases
- benefits of gateway api over ingress
- encrypting traffic in transit with cilium
- sidecar-based vs sidecarless architectures

#### eBPF
- the role of eBPF in cilium eBPF key benefits
- eBPF-based platforms vs IPtables-based platform

#### network observability 
- observability capabilities of hubble
- enabling layer 7 protocol visibility 
- use hubble from cli or hubble ui

#### BGP and external networking
- egress connectivity requirements 
- options to connect cilium-managed cluster to external network 