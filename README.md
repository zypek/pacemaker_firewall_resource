# pacemaker_firewall_resource
# Firewall Resource Agent

This repository provides an OCF-compliant resource agent for managing firewall rules using either `iptables` or `nftables`. It ensures that specified TCP ports and/or source IPs are rejected based on the provided configuration. The agent supports single-node or multi-node deployments within a Pacemaker cluster.

## Features

- Detects and uses `nftables` by default if both `iptables` and `nftables` are available.
- Allows configuration of:
  - Ports to block (`ports`).
  - Source IPs to block (`source_ips`).
  - The network interface to apply the rules (`interface`).
- Supports monitoring to ensure that the configured rules are active.
- Provides comprehensive error handling and logging for easy debugging.

## Parameters

| Parameter       | Required | Description                                                                 |
|-----------------|----------|-----------------------------------------------------------------------------|
| `ports`         | Yes      | Comma-separated list of TCP ports to reject.                                |
| `source_ips`    | No       | Comma-separated list of source IPs to reject.                               |
| `interface`     | No       | The network interface where the firewall rules should be applied.           |

## Actions

| Action         | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `start`        | Configures and applies the firewall rules.                                  |
| `stop`         | Removes the configured firewall rules.                                      |
| `monitor`      | Verifies the presence of the configured firewall rules.                     |
| `validate-all` | Checks the provided parameters and validates the configuration.             |
| `meta-data`    | Outputs metadata about the resource agent in XML format.                    |

## Examples

### Create a Firewall Resource
To create a firewall resource in a Pacemaker cluster, use the following `pcs` command:

```bash
sudo pcs resource create firewall ocf:network:firewall \
  ports="80,443" \
  source_ips="192.168.1.100,192.168.1.101" \
  interface="eth0" \
  op monitor interval=20s timeout=10s
```

### Set Resource to Run on One Node Only
To ensure the resource is not cloned across multiple nodes, use:

```bash
sudo pcs resource meta firewall clone-max=1 clone-node-max=1
```

### Verify Resource Status
After creating the resource, verify its status using:

```bash
sudo pcs status
```

### Delete the Resource
To remove the resource from the cluster:

```bash
sudo pcs resource delete firewall
```

## How It Works

1. The resource agent checks for the availability of `nftables` and `iptables`. If both are installed, it defaults to `nftables`.
2. Rules are added to the firewall tool for:
   - Blocking specified TCP ports.
   - Blocking traffic from specified source IPs.
3. The `monitor` action validates that the configured rules are in place and active.
4. On `stop`, all rules configured by the agent are removed.

---------------------------------------------------------------------------------------------------------------

# Firewall OCF Resource Agent

A **multi-state Pacemaker resource agent** for managing firewall rules using `iptables` or `nftables`.  
This resource agent allows a cluster to enforce different firewall rules depending on whether a node is in **Master (Promoted)** or **Slave (Unpromoted)** state.

## Features

✅ Supports **both `iptables` and `nftables`**  
✅ Implements **multi-state (Master-Slave) behavior**  
✅ **Unpromoted (Slave) nodes apply firewall rules** to block specified traffic  
✅ **Promoted (Master) nodes remove firewall rules**, allowing unrestricted access  
✅ **Dynamic rule management** when nodes are promoted or demoted  
✅ **Integrated with Pacemaker clustering**  

## How It Works

| **Cluster Role**      | **Firewall Behavior**                         |
|----------------------|-----------------------------------------------|
| **Unpromoted (Slave)** | Applies firewall rules to restrict access.   |
| **Promoted (Master)** | Removes firewall rules, allowing unrestricted access. |
| **Demotion (Master → Slave)** | Reapplies firewall rules.   |
| **Promotion (Slave → Master)** | Removes firewall rules.   |
| **Monitoring** | Ensures rules are correctly applied or removed. |

## Installation & Usage

### 1⃣ Copy the Script to `/usr/lib/ocf/resource.d/`
```bash
sudo mkdir -p /usr/lib/ocf/resource.d/custom
sudo cp firewall.sh /usr/lib/ocf/resource.d/custom/firewall
sudo chmod +x /usr/lib/ocf/resource.d/custom/firewall
```

### 2⃣ Create the Resource in Pacemaker
```bash
pcs resource create firewall ocf:custom:firewall \
    ports="80,443" source_ips="0.0.0.0/0" promotable \
    op start timeout=20s \
    op stop timeout=20s \
    op monitor interval=10s on-fail=restart \
    op monitor interval=20s role=Unpromoted on-fail=restart
```

### 3⃣ Set Migration Threshold (Optional)
This ensures the resource **fails over after `2` failures**.
```bash
pcs resource meta firewall migration-threshold=2
```

### 4⃣ Check the Resource Status
```bash
pcs status
```

## Parameters

| Parameter     | Description |
|--------------|-------------|
| **`ports`** | Comma-separated list of TCP ports to block when Unpromoted. |
| **`source_ips`** | Comma-separated list of source IPs to block. |
| **`state`** | Path to state file (used internally). |
| **`notify_delay`** | Delay before notifying Pacemaker of a state change. |

## Resource Actions

| Action | Description |
|--------|-------------|
| **start** | Starts the firewall rules in Unpromoted mode (block traffic). |
| **stop** | Stops the firewall rules (removes all rules). |
| **promote** | Promotes a node to Master (removes rules). |
| **demote** | Demotes a node to Slave (reapplies rules). |
| **monitor** | Checks whether the rules are correctly applied. |
| **meta-data** | Displays resource metadata. |

## Monitoring & Logs

### Check Resource Status
```bash
pcs resource show firewall
```

### Check System Logs
```bash
journalctl -u pacemaker --no-pager -n 50
```

### Check Pacemaker Logs
```bash
grep "firewall" /var/log/pacemaker.log
```

## Troubleshooting

### Resource Not Starting?
Check the logs for errors:
```bash
pcs resource debug-start firewall
```

### Rules Not Applied Correctly?
Manually check the firewall rules:

#### nftables
```bash
nft list ruleset
```

#### iptables
```bash
iptables -L -n -v
```

### Force Failover
Manually move the Master node:
```bash
pcs resource move firewall
```

## Contributing

Feel free to submit **issues or pull requests** to improve this resource agent.

## License

This script is licensed under the **GNU GPLv2 or later**.
