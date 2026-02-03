# Kubernetes Pods — Quick Notes

## Pods
- Smallest deployable unit: one or more containers sharing network namespace, IP, ports, volumes.
- Spec key fields: containers, initContainers, volumes, restartPolicy, nodeName/nodeSelector, tolerations, affinity, hostNetwork, hostPID.
- Lifecycle: Pending → Running → Succeeded/Failed; use liveness/readiness/startup probes to manage health.
- Volume mounts allow persistent/shared data between containers.

## Labels
- Key/value metadata for selection and grouping.
- Used by Services, ReplicaSets, Deployments, NetworkPolicies.
- Immutable only by pattern if used in selectors; design stable labels for selectors.

## Annotations
- Arbitrary metadata (non-identifying); not used for selection.
- Good for tooling, controllers, large or binary data.
- Read by kubelet, admission controllers, custom controllers.

## Environment (env)
- container.spec.env: set individual vars (value / valueFrom).
- envFrom: populate from ConfigMap or Secret.
- valueFrom options: secretKeyRef, configMapKeyRef, fieldRef (Downward API), resourceFieldRef.
- Prefer secrets as mounted volumes where possible; avoid logging env containing secrets.

## Resources (CPU/Memory)
- requests: scheduler uses to place pods.
- limits: kubelet enforces (OOMKilled for memory, CPU throttled).
- QoS classes: Guaranteed (requests==limits), Burstable, BestEffort.
- Set realistic requests to help bin-packing and stability.

## ConfigMaps & Secrets
- ConfigMap: plain-text configuration data.
- Secret: base64-encoded (not encrypted by default) sensitive data; enable encryption at rest in cluster.
- Consumption: mounted as files or env vars (envFrom / env).
- Mark Secrets immutable if static; rotate secrets via controllers or rolling updates.

## IP / Networking concepts related to Pods
- Pod IP: unique IP per pod (shared among containers in pod). Pod-to-Pod direct IPing within cluster (no NAT) if CNI supports it.
- Pod CIDR: IP range allocated per node or per pod depending on CNI.
- Node IP: host machine IP; node may host multiple pods.
- HostNetwork: pod uses node network namespace (uses node IP).
- HostPort: expose specific host port for a container (binds on node IP).
- Service ClusterIP: stable virtual IP for group of pods (internal cluster load-balancing).
- Headless Service (clusterIP: None): DNS-based pod discovery, no virtual IP.
- NodePort: exposes service on each node on a port (for external access).
- LoadBalancer: cloud LB provisioning (external IP) that fronts Service.
- ExternalIP / ExternalName: ways to reference external endpoints.
- kube-proxy modes: userspace, iptables, ipvs — implement Service traffic routing.
- CNI plugins: implement pod networking (flannel, calico, weave, cilium); enforce pod IP allocation, routing, and NetworkPolicy enforcement.
- NetworkPolicy: controls allowed ingress/egress between pods (namespace/label selectors).
- DNS: CoreDNS provides service/pod name resolution; pods use kube-dns by default.
- Dual-stack: IPv4/IPv6 support (if enabled); pod and service IP families configured cluster-wide.
- NAT: Services often NAT traffic (cluster IP → pod IP) via kube-proxy; some CNIs enable direct routing (no SNAT).
- Viewing IPs: kubectl get pods -o wide, kubectl describe pod.

## Best practices (brief)
- Use labels for selectors; keep annotations for metadata.
- Configure requests and limits; monitor QoS.
- Keep Secrets out of logs and Git; prefer mounted secrets over env for large secrets.
- Use NetworkPolicy deny-by-default for multi-tenant security.
- Prefer service discovery (ClusterIP) + Ingress/LoadBalancer rather than HostPort when possible.
