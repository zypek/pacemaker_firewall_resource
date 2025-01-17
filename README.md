# pacemaker_firewall_resource
Firewall Resource Agent
This repository provides an OCF-compliant resource agent for managing firewall rules using either iptables or nftables. It ensures that specified TCP ports and/or source IPs are rejected based on the provided configuration. The agent supports single-node or multi-node deployments within a Pacemaker cluster.

Features
Detects and uses nftables by default if both iptables and nftables are available.
Allows configuration of:
Ports to block (ports).
Source IPs to block (source_ips).
The network interface to apply the rules (interface).
Supports monitoring to ensure that the configured rules are active.
Provides comprehensive error handling and logging for easy debugging.
Parameters
Parameter	Required	Description
ports	Yes	Comma-separated list of TCP ports to reject.
source_ips	No	Comma-separated list of source IPs to reject.
interface	No	The network interface where the firewall rules should be applied.
Actions
Action	Description
start	Configures and applies the firewall rules.
stop	Removes the configured firewall rules.
monitor	Verifies the presence of the configured firewall rules.
validate-all	Checks the provided parameters and validates the configuration.
meta-data	Outputs metadata about the resource agent in XML format.
Examples
Create a Firewall Resource
To create a firewall resource in a Pacemaker cluster, use the following pcs command:

bash
Copy
Edit
sudo pcs resource create firewall ocf:network:firewall \
  ports="80,443" \
  source_ips="192.168.1.100,192.168.1.101" \
  interface="eth0" \
  op monitor interval=20s timeout=10s
Set Resource to Run on One Node Only
To ensure the resource is not cloned across multiple nodes, use:

bash
Copy
Edit
sudo pcs resource meta firewall clone-max=1 clone-node-max=1
Verify Resource Status
After creating the resource, verify its status using:

bash
Copy
Edit
sudo pcs status
Delete the Resource
To remove the resource from the cluster:

bash
Copy
Edit
sudo pcs resource delete firewall
How It Works
The resource agent checks for the availability of nftables and iptables. If both are installed, it defaults to nftables.
Rules are added to the firewall tool for:
Blocking specified TCP ports.
Blocking traffic from specified source IPs.
The monitor action validates that the configured rules are in place and active.
On stop, all rules configured by the agent are removed.
