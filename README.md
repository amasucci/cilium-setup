# cilium-setup

This is how to setup cilium on K0S.

Install K0S using `k0sctl`

Generate k0s cluster config with the command:

`k0sctl init --k0s -n "k0s" -u "root" -i "~/.ssh/id_rsa" -C 1 "10.0.0.92" > k0s-cluster-config.yaml`

Make sure the public key is whitelisted in the `authorized_keys` file of the root user.

Edit the `k0s-cluster-config.yaml` and modify the following fields:

`spec.hosts.[].role` set them to `controller` `worker` `controller+worker` or `single`
`spec.k0s.config.spec.network.kubeProxy.disabled` set it to `true`
`spec.k0s.config.spec.network.provider` set it to `custom`

## Create the cluster

To create the cluster run:

`k0sctl apply --config k0s-cluster-config.yaml --no-wait`

Generate a kubeconfig using: `k0sctl kubeconfig --config k0s-cluster-config.yaml > ~/.kube/config`


To install cilium run:
`cilium install -f values.yaml`

```yaml
kubeProxyReplacement: "strict"

k8sServiceHost: 10.0.2.15
k8sServicePort: 6443

global:
  encryption:
    enabled: true
    nodeEncryption: true

hubble:
  metrics:
    #serviceMonitor:
    #  enabled: true
    enabled:
    - dns:query;ignoreAAAA
    - drop
    - tcp
    - flow
    - icmp
    - http

  ui:
    enabled: true
    replicas: 1
    ingress:
      enabled: true
      hosts:
        - hubble.k3s.intra
      annotations:
        cert-manager.io/cluster-issuer: ca-issuer
      tls:
      - secretName: ingress-hubble-ui
        hosts:
        - hubble.k3s.intra

  relay:
    enabled: true


operator:
  replicas: 1

ipam:
  mode: "cluster-pool"
  operator:
    clusterPoolIPv4PodCIDRList: ["10.43.0.0/16"]
    clusterPoolIPv4MaskSize: 24
    clusterPoolIPv6PodCIDRList: ["fd00::/104"]
    clusterPoolIPv6MaskSize: 120

prometheus:
  enabled: true
  # Default port value (9090) needs to be changed since the RHEL cockpit also listens on this port.
  port: 19090
```
