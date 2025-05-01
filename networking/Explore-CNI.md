# Kubernetes CNI Configuration Guide

This guide documents how to inspect and identify Container Network Interface (CNI) configurations in a Kubernetes cluster.

## Container Runtime Endpoint

To inspect the kubelet service and identify the container runtime endpoint value:

```bash
ps -aux | grep -i kubelet | grep container-runtime
```

## CNI Binaries Path

The path configured with all binaries of CNI supported plugins:

```
/opt/cni/bin
```

## Available CNI Plugins

To check which plugins are available on the host:

```bash
cd /opt/cni/bin
ls -la | grep -E "dhcp|vlan|cisco|bridge"
```

**Note:** In our inspection, we found that the `cisco` plugin is not available in the list of CNI plugins on this host.

## Configured CNI Plugin

To identify which CNI plugin is configured to be used on the Kubernetes cluster:

```bash
ls /etc/cni/net.d/
```

Output:

```
controlplane /opt/cni/bin ➜ ls /etc/cni/net.d/
10-flannel.conflist
```

## CNI Configuration

The binary executable file that will be run by kubelet after a container and its associated namespace are created can be found in the CNI configuration file:

```bash
cat /etc/cni/net.d/10-flannel.conflist
```

The configuration reveals the following JSON structure:

```json
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

## YAML Representation

The same configuration in YAML format:

```yaml
name: cbr0
cniVersion: 0.3.1
plugins:
  - type: flannel
    delegate:
      hairpinMode: true
      isDefaultGateway: true
  - type: portmap
    capabilities:
      portMappings: true
```

This configuration shows that `flannel` is the primary CNI plugin in use, with `portmap` as a chained plugin for port mapping capabilities.
