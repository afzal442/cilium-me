While addressing the issue https://github.com/cilium/cilium/issues/36518, I came up with the following preflight check.

## IP Address Planning – Avoid Overlapping Ranges

Before installing Cilium, ensure that your Kubernetes network ranges are correctly planned and do not overlap unless explicitly supported by your IPAM mode.

Incorrectly configured CIDR ranges can result in connectivity issues, including failure of pod-to-service communication, routing problems, or silent traffic drops.

## Non-overlapping CIDR ranges required

In most deployments (including the default Cilium IPAM modes), the following CIDR ranges must not

| Range type                 | Description                                                   | Example          |
| -------------------------- | ------------------------------------------------------------- | ---------------- |
| **Pod CIDR**               | Range from which Pod IPs are allocated                        | `10.0.0.0/16`    |
| **Service CIDR**           | Range Kubernetes uses for `clusterIP` services                | `10.96.0.0/12`   |
| **Node / VPC / Host CIDR** | Network used by Kubernetes nodes or underlying infrastructure | `192.168.0.0/24` |

Ensure each of these CIDR blocks is unique and non-overlapping unless using an external IPAM mode that explicitly allows overlap (see below).

Example of safe configuration:
```
Pod CIDR:      10.0.0.0/16
Service CIDR:  10.96.0.0/12
Node network:  192.168.0.0/24
```

Example of misconfiguration:

Pod CIDR:      10.0.0.0/16
Service CIDR:  10.0.0.0/20    # ❌ Overlaps Pod CIDR
Node network:  10.0.5.0/24    # ❌ Overlaps both ranges
