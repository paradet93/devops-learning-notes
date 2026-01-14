# prepare host
- disable swap
	- `sudo swapoff -a`(temporarily)
	- edit /etc/fstab for permanent change
	- KI instruction:
```
	swapoff -a
# Damit es nach Reboot so bleibt, kommentiere die swap Zeile in fstab aus:
sed -i '/swap/ s/^/#/' /etc/fstab
```
> ? was macht sed, wie verwendet man es?

## install container runtime
- per default kubernetes uses the Container Runtime Interface (CRI)
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/
	1. enable IPv4 Packet forwarding
	2. install [[cgroup]] driver
		1. here: systemd
	3. 
	