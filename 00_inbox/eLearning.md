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


# mock exam 3
- how to expose multiple pods one service
- custom columns & sort
- can you choose columns with k get customresourcedefinitions?
- HPA
- persistentvolumeclaim podspec
- ready/live probes for troubleshooting

## q1
> read carefully, did not change number for request

## q5
> did not specify namespace 

q8
> did not specify correct amount of storage request (150 instead of 100)

# q9
> you need to specify the namespce for the gateway which is the parentref

# q10
learn command kubectl get event
> otherwise correct, looks like a bug

# q11 
> again wrong capacity on pvc. BUT it is requested to get 100Mi, is has a capacity of 150Mi ?!

mine:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      initContainers:
        - name: busybox
          image: busybox
          restartPolicy: Always
          command: ['sh', '-c', 'sleep 3600']
          volumeMounts:
            - name: python-data
              mountPath: /usr/src
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
  volumeName: olive-pv-cka10-str
  storageClassName: olive-stc-cka10-str

### solution:

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: olive-stc-cka10-str
  volumeName: olive-pv-cka10-str
  resources:
    requests:
      storage: 100Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      - name: busybox
        image: busybox
        command:
          - "bin/sh"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: python-data
            mountPath: "/usr/src"
            readOnly: true
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str

---
apiVersion: v1
kind: Service
metadata:
  name: olive-svc-cka10-str
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 32006
  selector:
    app: olive-app-cka10-str
```

> Service does not matter

# q14
> lets learn what endpointslices are ig

Solve this question on: `ssh cluster3-controlplane`  
  
  
We have an **external** webserver running on `student-node` which is exposed at port `9999`. We have created a service called `external-webserver-cka03-svcn` that can connect to our local webserver from within the kubernetes **cluster3**, but at the moment, it is not working as expected.  
  
  
  
Fix the issue so that other pods within **cluster3** can use **external-webserver-cka03-svcn** service to access the webserver.  
  

Solution

##### SSH into the `cluster3-controlplane` host

```sh
ssh cluster3-controlplane
```

Let's check if the webserver is working or not:

```
cluster3-controlplane  ~ ➜  curl student-node:9999
...
<h1>Welcome to nginx!</h1>
...
```

Now we will check if service is correctly defined:

```
cluster3-controlplane  ~ ➜  kubectl describe svc -n kube-public external-webserver-cka03-svcn 
Name:              external-webserver-cka03-svcn
Namespace:         kube-public
.
.
Endpoints:         <none> # there are no endpoints for the service
...
```

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

First, obtain the IP address of the student node, easiest way is to ping it:

```
root@student-node ~ ✦  ping student-node
PING student-node (192.168.222.128) 56(84) bytes of data.
64 bytes from student-node (192.168.222.128): icmp_seq=1 ttl=64 time=0.023 ms
64 bytes from student-node (192.168.222.128): icmp_seq=2 ttl=64 time=0.030 ms
```

In this example : student-node IP is `192.168.222.128`

Next, use the IP address to create the EndpointSlice:

```
cluster3-controlplane  ~ ➜ kubectl  apply -f - <<EOF
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-webserver-cka03-svcn
  namespace: kube-public
  labels:
    kubernetes.io/service-name: external-webserver-cka03-svcn
addressType: IPv4
ports:
  - protocol: TCP
    port: 9999
endpoints:
  - addresses:
      - 192.168.222.128   # IP of student node
EOF
```

Finally check if the `curl test` works now:

```
cluster3-controlplane  ~ ➜  kubectl run -n kube-public --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-cka03-svcn
...
<title>Welcome to nginx!</title>
...
```


# q15 
> did not change the egress part

Solve this question on: `ssh cluster1-controlplane`  
  
  
An nginx-based pod called `cyan-pod-cka28-trb` is running under the `cyan-ns-cka28-trb` namespace and is exposed within the cluster using the `cyan-svc-cka28-trb` service.  
  
This is a `restricted` pod, so a network policy called `cyan-np-cka28-trb` has been created in the same namespace to apply some restrictions on this pod.  
  
  
Two other pods called `cyan-white-cka28-trb` and `cyan-black-cka28-trb` are also running in the `default` namespace.  
  
  
The nginx-based app running on the `cyan-pod-cka28-trb` pod is exposed internally on the default nginx port (`80`).  
  

**Expectation:** This app should `only` be accessible from the `cyan-white-cka28-trb` pod.  
  
  
**Problem:** This app is not accessible from anywhere.  
  
  
Troubleshoot this issue and fix the connectivity as per the requirement listed above.  
  
  
**Note:** You can exec into `cyan-white-cka28-trb` and `cyan-black-cka28-trb` pods and test connectivity using the `curl` utility.  
  
  
You may update the network policy, but make sure it is not deleted from the `cyan-ns-cka28-trb` namespace.

Solution

##### SSH into the `cluster1-controlplane` host

```sh
ssh cluster1-controlplane
```

  
  
Let's look into the network policy

```sh
kubectl edit networkpolicy cyan-np-cka28-trb -n cyan-ns-cka28-trb
```

Under `spec:` -> `egress:` you will notice there is not `cidr:` block has been added, since there is no restrcitions on `egress` traffic so we can update it as below. Further you will notice that the port used in the policy is `8080` but the app is running on default port which is `80` so let's update this as well (under `egress` and `ingress`):

- Change `port: 8080` to `port: 80`

```yaml
- ports:
  - port: 80
    protocol: TCP
  to:
  - ipBlock:
      cidr: 0.0.0.0/0
```

Now, lastly notice that there is no POD selector has been used in `ingress` section but this app is supposed to be accessible from `cyan-white-cka28-trb` pod under `default` namespace. So let's edit it to look like as below:

```yaml
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: default
    podSelector:
      matchLabels:
        app: cyan-white-cka28-trb
```

Now, let's try to access the app from `cyan-white-pod-cka28-trb`

```sh
kubectl exec cyan-white-cka28-trb -- sh -c 'curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local'
```

Also make sure its not accessible from the other pod(s)

```sh
kubectl exec cyan-black-cka28-trb -- sh -c 'curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local'
```

It should not work from this pod. So its looking good now.


# mock exam 4
- Probes

## Q1
-> learn more about probes
-> check solution

### Q2
-> check solution

#### question + solution
Solve this question on: `ssh cluster3-controlplane`  
  
  
Utilize **helm** to **search** for the **repository URL** of the Bitnami version of the **nginx** **repository**. Ensure that you save the **repository URL** in the file located at `/root/nginx-helm-url.txt` on the `cluster3-controlplane`.

---
#### SSH into the Control Plane

Run the following command to access **cluster3-controlplane**:

```sh
ssh cluster3-controlplane
```

#### 1. Check repo url for the nginx official repository

Run the Helm command below to list the repository URLs:

```
helm search hub nginx --list-repo-url | head -n15
URL                                                     CHART VERSION   APP VERSION                             DESCRIPTION                                             REPO URL                                          
https://artifacthub.io/packages/helm/krakazyabr...      1.0.0           1.19.0                                  Nginx Helm chart for Kubernetes                         https://krakazyabra.github.io/microservices       
https://artifacthub.io/packages/helm/jfrog/nginx        15.1.5          1.25.2                                  NGINX Open Source is a web server that can be a...      https://charts.jfrog.io                           
https://artifacthub.io/packages/helm/ashu-nginx...      0.1.0           1.16.0                                  A Helm chart for Kubernetes                             https://redashu.github.io/test-helm/              
https://artifacthub.io/packages/helm/zrepo-test...      5.1.5           1.16.1                                  Chart for the nginx server                              http://pqbbvd.natappfree.cc/charts/index.yaml     
https://artifacthub.io/packages/helm/wiremind/n...      2.1.1                                                   An NGINX HTTP server                                    https://wiremind.github.io/wiremind-helm-charts   
https://artifacthub.io/packages/helm/dhinesh/nginx      19.0.2          1.27.4                                  NGINX Open Source is a web server that can be a...      oci://registry-1.docker.io/bitnamicharts/nginx    
https://artifacthub.io/packages/helm/dysnix/nginx       7.1.8           1.19.4                                  Chart for the nginx server                              https://dysnix.github.io/charts                   
https://artifacthub.io/packages/helm/bitnami-ak...      13.2.12         1.23.2                                  NGINX Open Source is a web server that can be a...      https://marketplace.azurecr.io/helm/v1/repo       
https://artifacthub.io/packages/helm/cloudnativ...      3.2.0           1.16.0                                  Chart for the nginx server                              https://cloudnativeapp.github.io/charts/curated/  
https://artifacthub.io/packages/helm/shubhamtat...      0.1.12          1.19.6                                  Nginx Helm chart for Kubernetes                         https://shubhamtatvamasi.github.io/helm           
https://artifacthub.io/packages/helm/test-nginx...      0.1.0           1.16.0                                  A Helm chart for Kubernetes                             https://vizarg.github.io/helm-chart-nginx/nginx   
https://artifacthub.io/packages/helm/bitnami/nginx      19.0.2          1.27.4                                  NGINX Open Source is a web server that can be a...      https://charts.bitnami.com/bitnami                
https://artifacthub.io/packages/helm/matic-insu...      1.1.1           1.1.1                                   A Nginx Helm chart for Kubernetes                       https://matic-insurance.github.io/helm-charts     
https://artifacthub.io/packages/helm/onurs-repo...      0.2.0           1.20.2                                  Helm chart for nginx-1.20.2                             https://rezoruno.github.io/helm-charts
```

#### Capture the Repository URL

Store the official Nginx repository URL by executing the following command:

```sh
echo "https://charts.bitnami.com/bitnami" > /root/nginx-helm-url.txt
```
### q3

### Q9
-> my solution
```bash title='my solution' fold
cluster1-controlplane ~ ➜  etcdutl --data-dir /root/default.etcd snapshot restore /opt/cluster1_backup_to_restore.db 
2026-02-02T20:12:47Z    info    snapshot/v3_snapshot.go:265     restoring snapshot      {"path": "/opt/cluster1_backup_to_restore.db", "wal-dir": "/root/default.etcd/member/wal", "data-dir": "/root/default.etcd", "snap-dir": "/root/default.etcd/member/snap", "initial-memory-map-size": 10737418240}
2026-02-02T20:12:47Z    info    membership/store.go:141 Trimming membership information from the backend...
2026-02-02T20:12:47Z    info    membership/cluster.go:421       added member    {"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
2026-02-02T20:12:47Z    info    snapshot/v3_snapshot.go:293     restored snapshot       {"path": "/opt/cluster1_backup_to_restore.db", "wal-dir": "/root/default.etcd/member/wal", "data-dir": "/root/default.etcd", "snap-dir": "/root/default.etcd/member/snap", "initial-memory-map-size": 10737418240}

cluster1-controlplane ~ ➜  
```
--> worked!
### Q10

did not specify correct name of HPA ...

### Q13
> is "recreate" just delete and create? Or something else?

> correct way to wait for container to start?

```yaml title='my solution(wrong)' fold
apiVersion: v1
kind: Pod
metadata:
  name: elastic-app-cka02-arch
spec:
  containers:
  - name: elastic-app
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      mkdir /var/log;
      i=0;
      while true;
      do
        echo "$(date) INFO $i" >> /var/log/elastic-app.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  initContainers:
  - name: busybox
    image: busybox
    restartPolicy: Always
    command: ['sh', '-c', 'sleep 20;tail -f /var/log/elastic-app.log']
    volumeMounts:
      - name: varlog
        mountPath: /var/log/
  volumes:
  - name: varlog
    emptyDir: {}
```

Solve this question on: `ssh cluster3-controlplane`  
  
  
A pod called `elastic-app-cka02-arch` is running in the `default` namespace. The `YAML` file for this pod is available at `/root/elastic-app-cka02-arch.yaml` on the `cluster3-controlplane`. The single application container in this pod writes logs to the file `/var/log/elastic-app.log`.  
  
  
One of our logging mechanisms needs to read these logs to send them to an upstream logging server, but we don't want to increase the read overhead for our main application container. So, you need to `recreate` this POD with an additional co-located container named `busybox` that will run along with the application container and print to the `STDOUT` by running the command `tail -f /var/log/elastic-app.log`. You can use the `busybox` image for this container.

---
Solution

##### SSH into the `cluster3-controlplane` host

```sh
ssh cluster3-controlplane
```

Recreate the pod with a new container called `busybox`. Update the `/root/elastic-app-cka02-arch.yaml` YAML file as shown below:  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: elastic-app-cka02-arch
spec:
  containers:
  - name: elastic-app
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      mkdir /var/log; 
      i=0;
      while true;
      do
        echo "$(date) INFO $i" >> /var/log/elastic-app.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: busybox
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -f  /var/log/elastic-app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

  
  
  
Next, recreate the pod:  

```sh
student-node ~ ➜ kubectl replace -f /root/elastic-app-cka02-arch.yaml --force
pod "elastic-app-cka02-arch" deleted
pod/elastic-app-cka02-arch replaced

student-node ~ ➜ 
```

### Q14
> how does coreDNS work, what do i need to know??

> what is the solution??



Solve this question on: `ssh cluster3-controlplane`  
  
  
The **CoreDNS** configuration in the cluster needs to be updated:

Update the **CoreDNS** configuration in the cluster so that **DNS resolution** for `cka.local` works exactly like `cluster.local` and in addition to it.

Test your configuration using the `jrecord/nettools` image by executing the following commands:

```
nslookup kubernetes.default.svc.cluster.local
nslookup kubernetes.default.svc.cka.local
```

Solution

##### SSH into the `cluster3-controlplane` host

```sh
ssh cluster3-controlplane
```

#### 1. Update the CoreDNS Configuration

The CoreDNS configuration is stored in a ConfigMap. To edit this configuration, use the following command:

```
kubectl edit cm -n kube-system coredns
```

Below is the structure of the CoreDNS configuration:

```
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cka.local cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
```

#### 2. Restart the CoreDNS Deployment

To apply the changes, restart the CoreDNS deployment using the command:

```
kubectl rollout restart deploy -n kube-system coredns
```

#### 3. Verify the Changes

To verify the updates, execute the following commands:

```
kubectl run test --rm -it --image=jrecord/nettools --restart=Never -- nslookup kubernetes.default.svc.cluster.local
kubectl run test --rm -it --image=jrecord/nettools --restart=Never -- nslookup kubernetes.default.svc.cka.local
```
### -Q16
> correct way?
yes, done
### Q17 
> correct way?

forget sysctl enable

solution: 

Solve this question on: `ssh cluster5-controlplane`  
  
  
A **debian package** for **cri-dockerd** `cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb` is located in the **/root folder** on the `cluster5-controlplane`. As part of **cluster initialization**, install the package and make sure that **cri-docker service** is **up** and **enabled** on the system. Additionally, **enable IP forwarding** on the server.

Solution

#### SSH into the Control Plane

To access **cluster5-controlplane**, execute the following command:

```sh
ssh cluster5-controlplane
```

#### 1. Install cri-dockerd

The Debian package for cri-dockerd is available at `cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb`. Install it using the following command:

```sh
dpkg -i cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb 
```

#### 2. Enable and Start the Service

To enable and start the cri-dockerd service, use the following command:

```
sudo systemctl enable --now cri-docker.service
```

#### 3. Enable IP Forwarding

To ensure that IP forwarding changes are persistent:

1. Create a configuration file:

```sh
   vi /etc/sysctl.d/k8s.conf
```

2. Add the following line to the file:

```sh
   net.ipv4.ip_forward=1
```

3. Apply the changes:

```sh
   sysctl -p
```

To set the changes temporarily, execute the following command:

```sh
sysctl -w net.ipv4.ip_forward=1
```