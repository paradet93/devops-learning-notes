Primary [[Node]] agent, that runs on each node.
- one of 2 main components on a worker node (next to [[kube-proxy]])
- Linux Daemon
	- background process


# Functions
- registers node with [[api-server]]
- manages [[Container]]s on node
	- receives [[podspec]]s from api-server
	- ensures containers are running and healthy
		- monitors health
		- manages lifecycle (starting, stopping, restarting)
		- ensures node cpu and memory resources are not exhausted
	- interacts with container runtime (by default [[containerd]])
# How to Troubleshoot