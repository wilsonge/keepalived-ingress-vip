# Keepalived Ingress VIP

![Hero Banner](https://raw.githubusercontent.com/janeczku/keepalived-ingress-vip/master/img/banner-top.png)

This is a HA/IP failover solution for Kubernetes Ingress Controllers, such as [NGINX Ingress](https://kubernetes.github.io/ingress-nginx/) and mainly useful for on-premises environments that lack infrastructure load balancing capabilities.

It provides a virtual IP for terminating ingress traffic on the cluster and provides sub-second failover in scenarios such as node crashing, node being drained during cluster upgrades or node becoming network partitioned from Kubernetes control plane.

The Virtual IP is managed using the [Virtual Router Redundancy Protocol (VRRP) implementation of Keepalived](https://keepalived.readthedocs.io/en/latest/case_study_failover.html) and - instead of relying on Kubernetes API state (which would be to slow for this purpose) - the readiness of the ingress nodes is determined by running probes against local HTTP health check endpoints (Ingress, Kubelet).

## Prerequisites

- Network infrastructure must permit the use of VRRP protocol and multicast traffic (which excludes most public clouds)
- Tested on a vSphere 6.7u3 environment with stock vSwitch and port group setup. Other non-cloud infrastructure providers should work.
- The Chart included in the repo requires Helm >= v3.1

### Installing the Chart using Rancher

In the Rancher GUI, navigate to __Cluster->System Project->Tools->Catalog__ and click on __Add Catalog__:

- Name: <Some Name>
- Catalog URL: `https://github.com/janeczku/keepalived-ingress-vip`
- Helm Version: `Helm v3`

Afterwards, you can launch the chart from the System project's __Apps__ page providing the configuration variables documented below.

### Installing the Chart using Helm CLI

To run the application using the virtual IP `172.16.135.2/21` on the host interface named `ens160`:

```bash
$ helm install -n vip --name keepalived-ingress-vip ./chart \
  --set keepalived.vrrpInterfaceName=ens160 \
  --set keepalived.vipInterfaceName=ens160 \
  --set keepalived.vipAddressCidr="172.16.135.2/21"
```

### Uninstalling the Chart using Helm CLI

To uninstall the `keepalived-ingress-vip` deployment:

```bash
$ helm delete -n vip keepalived-ingress-vip
```

### Configuration

The following table lists the configurable parameters of this chart and their default values.

| Parameter                           | Description                                                      | Default                                             |
| ----------------------------------- | -----------------------------------------------------------------| --------------------------------------------------- |
| `keepalived.enableDebug`            | Enable verbose logging                                           | `false`                                             |
| `keepalived.vrrpInterfaceName`      | The host network interface name to use for VRRP traffic.         | `eth0`                                              |
| `keepalived.vipInterfaceName`       | The host network interface name to attach the VIP to.            | `eth0`                                              |
| `keepalived.vipAddressCidr`         | The Virtual IP address to use (in CIDR notation, e.g. `192.168.11.2/24`) | ``                                          |
| `keepalived.virtualRouterId`        | A unique numeric Keepalived Router ID.                           | `10`                                                |
| `keepalived.ingressHealthzUrl`      | The URL to poll to determine Ingress health (expect HTTP status code 200) | `http://127.0.0.1:10254/healthz`           |
| `keepalived.kubeletHealthzUrl`      | The URL to poll to determine Kubelet health (expect HTTP status code 200) | `http://127.0.0.1:10248/healthz`           |
| `image.repository`                  | Image repository to pull from                                    | `janeczku/keepalived-ingress-vip`                   |
| `image.tag`                         | Image tag to pull                                                | `v0.1.1`                                            |
| `image.pullPolicy`                  | Image pull policy                                                | `IfNotPresent`                                      |
| `rbac.create`                       | Whether to create the required RBAC resources                    | `true`                                              |
| `rbac.pspEnabled`                   | Whether to create the required PodSecurityPolicy                 | `false`                                             |
| `serviceAccount.name`               | Use an existing service account instead of creating a new one.   | ``                                                  |
| `pod.replicas`                      | The number of Keepalived instances to run in the cluster         | `2`                                                 |
| `pod.extraEnv`                      | Additional pod environment variables                             | `[]`                                                |
| `pod.resources.requests.cpu`        | CPU resource requests                                            | 80m                                                 |
| `pod.resources.limits.cpu`          | CPU resource limits                                              |                                                 |
| `pod.resources.requests.memory`     | Memory resource requests                                         | 6Mi                                                 |
| `pod.resources.limits.memory`       | Memory resource limits                                           | 12Mi                                                |
| `pod.nodeSelector`                  | Node selector                                                    | `{}`                                                |
| `pod.tolerations`                   | Additional pod taint tolerations                                 | `[]`                                                |
| `pod.affinity`                      | Additional pod affinity configuration                            | `{}`                                                |
| `pod.imagePullSecrets`              | Array of image Pull Secrets                                      | `[]`                                                |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.