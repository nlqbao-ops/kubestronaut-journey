#### cluster setup
- network security policies to restrict cluster level access
- CIS benchmark to review security config of k8s components
- properly setup ingress object with TLS
- protect node metadata and endpoints
- verify platform binaries before deploying

#### minimize microservices vulnerabilities
- pod security standards
- k8s secrets
- isolation techniques
- pod-to-pod encryption ( cilium, istio)

#### cluster hardening 
- RBAC
- caution in service accounts
- restrict access to kubernetes API
- upgrade k8s to avoid vulnerabilities

#### supply chain security 
- minimize base image footprint
- understand your supply chain ( SBOM, CICD, artifact repo)
- secure your supply chain
- perform static analytis of user workloads and container images 

#### system hardening
- minimize host OS footprint
- least-privilege identity and access mgmt
- minimize external access to the network 
- appropriate use kernel hardening tool

#### monitoring, logging and runtime security 
- behavioral analytics to detect malicious activities
- detect threat within physical infra, apps, network, data, users and workloads
- investigate and identify phases of attack and bad actors within env
- ensure immutability of containers at runtime
- user k8s audit logs to monitor access 