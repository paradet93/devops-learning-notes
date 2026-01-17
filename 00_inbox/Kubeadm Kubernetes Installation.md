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
	cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Sysctl params required by setup, params persist across reboots
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sysctl --system

# Abhängigkeiten installieren
apt-get update
apt-get install -y ca-certificates curl gnupg

# Docker GPG Key hinzufügen (Docker stellt containerd bereit)
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Repo hinzufügen
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update
apt-get install -y containerd.io

# Standard Config erstellen
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Suche "SystemdCgroup = false" und ändere es zu "true"
# Wir nutzen wieder sed für den "Profi-Modus":
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Neustart
systemctl restart containerd

## Install Kubeadm, Kubectl, Kubelet

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## install CNI 
-> calico, anhand der onprem doku

