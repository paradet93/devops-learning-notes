# CKA Mock exam 2
## where i needed the docs
- lookup storage class definition
- pv definition
	- how to mount localpath
	- add node affinity
		- matchexpression
		- matchfield
		- cluster1-controlplane ~ ➜  k apply -f pv.yaml 
Error from server (BadRequest): error when creating "pv.yaml": PersistentVolume in version "v1" cannot be handled as a PersistentVolume: json: cannot unmarshal object into Go struct field NodeSelectorTerm.spec.nodeAffinity.required.nodeSelectorTerms.matchExpressions of type []v1.NodeSelectorRequirement
apiVersion: v1
kind: PersistentVolume
metadata:
  name: orange-pv-cka07-str
spec:
  capacity:
    storage: 150Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: orange-stc-cka07-str
  hostPath:
    path: /opt/orange-data-cka07-str
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
     _     key: kubernetes.io/hostname_
          operator: In 
          values:
          - cluster1-controlplane
  - pvc definition
> how to send a command over a number of nodes via ssh if i have to look up information on different hosts?

- VPA 
	- everything to this question: "Deploy a **Vertical Pod Autoscaler (VPA)** named `analytics-vpa` for a deployment named `analytics-deployment` in the `cka24456` namespace. The **VPA** should automatically adjust the **CPU** and **memory** requests of the pods to optimize resource utilization. Ensure that the **VPA** operates in `Auto` mode, allowing it to `evict` and `recreate` pods with updated resource requests as needed."
	- https://kubernetes.io/docs/concepts/workloads/autoscaling/vertical-pod-autoscale/
	- solution: apiVersion: autoscaling.k8s.io/v1 kind: VerticalPodAutoscaler metadata: name: analytics-vpa namespace: cka24456 spec: targetRef: apiVersion: apps/v1 kind: Deployment name: analytics-deployment updatePolicy: updateMode: "Auto"
> how to check pods/applications with curl?- how to use curl inside troubleshoot pod?
- difference of port types in service
- reread: how to services select their pods (labels and stuff)
- configure env (from secret) (and read secrets page)
> question 12 solution?
	- - Under `rules:` -> `host:` change `example.com` to `kodekloud-ingress.app`
- Under `backend:` -> `service:` -> `name:` Change `example-service` to `nodeapp-svc-cka08-trb`
_- Change `port:` -> `number:` from `80` to `3000`_ <--- this was still wrong

- API Gateway
	- configure https/tls for gateway
	- how to check functionality of resources? (question 14)
- troubleshoot nodes
> question 16 answer?
**- i was missing the correct kulebet version**
whole answer:
#### SSH into the Control Plane

Run the following command to access **cluster2-controlplane**:

```sh
ssh cluster2-controlplane
```

#### 1. Check Cluster Status

Run `kubectl get nodes` to check the cluster status:

```sh
cluster2-controlplane ~ ➜  kubectl get nodes
The connection to the server cluster2-controlplane:6443 was refused - did you specify the right host or port?
```

This indicates that the **API server is down**.

---

#### 2. Verify Kubernetes Containers Are Running

Check if all the Kubernetes-related containers are running:

```sh
cluster2-controlplane ~ ➜ crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
fbbf1db37defb       ead0a4a53df89       5 hours ago         Running             coredns                   0                   4be392af8ae56       coredns-69f9c977-lsjwc
b7a2fc42b8b83       ead0a4a53df89       5 hours ago         Running             coredns                   0                   8fdd89e10d6d7       coredns-69f9c977-bfsfw
63f27f8d81c29       a0eed15eed449       5 hours ago         Running             etcd                      0                   3e093d91c4361       etcd-cluster2-controlplane
f07cb1acaa26f       0824682bcdc8e       5 hours ago         Exited              kube-controller-manager   0                   fba0d299aab89       kube-controller-manager-cluster2-controlplane
2f36c95ff0b7e       7ace497ddb8e8       5 hours ago         Exited              kube-scheduler            0                   11e422ba2d28b       kube-scheduler-cluster2-controlplane
```

The `kube-apiserver` container **is missing**.

---

#### 3. Check `kubelet` Logs

Since `kube-apiserver` is missing, check if `kubelet` is running:

```sh
cluster2-controlplane ~ ➜ systemctl status kubelet

Unit kubelet.service could not be found.
```

**This confirms that `kubelet` is missing or not installed.**

---

#### 4. Reinstall `kubelet`

Since `kubelet` is missing, we need to **install it manually**.

- Find the Correct Kubelet Version

```sh
cluster2-controlplane ~ ➜ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.0", GitCommit:"70d3cc986aa8221cd1dfb1121852688902d3bf53", GitTreeState:"clean", BuildDate:"2024-12-11T18:04:20Z", GoVersion:"go1.23.3", Compiler:"gc", Platform:"linux/amd64"}
```

The **`kubelet` version must match `kubeadm`**.

- Install `kubelet` (Matching Version)

```sh
sudo apt install -y kubelet=1.32.0-1.1
```

- Start the `kubelet` Service**

```sh
sudo systemctl start kubelet
```

---

#### 5. Verify That the Cluster is Restored

Wait a few moments for `kubelet` to bring back `kube-apiserver`, then check the cluster status:

```sh
cluster2-controlplane ~ ➜ kubectl get nodes

NAME                    STATUS   ROLES           AGE   VERSION
cluster2-controlplane   Ready    control-plane   27m   v1.32.0
cluster2-node01         Ready    <none>          26m   v1.32.0
```