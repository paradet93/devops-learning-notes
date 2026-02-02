- A service defines a logical set of endpoints (usually [[Pods]]) along with a policy about hot to make these endpoints acessible

### Selector 
- The set of endpoints is assigned by the selector (`spec.selector`) defined in the service. The controller for that service scans periodically for resources (Pods) that match the selctor

### Service types
- ClusterIP (Default)
- NodePort

### Port Definitions

> by default is `targetPort` set to the same value as `port` if not defined. This is for convenience reasons

- `targetPort` can be defined using names
	- if the port definition inside of a pod is named, targetport can be set to this name.
	- e.g. `pod.spec.containers.ports.name: http-web-svc`enables you to set `.service.spec.ports.targetPort: http-web-svc` 
		- complete yaml: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
	- you can use the same name inside podspec for different ports. E.g. in pod a listens http-web-svc on port 80, in pod b it listens on port 8080
	- 

#### Services without selectors

If used with a selector Kubernetes automatically creates endpoint slices. If you are not using a selector in your service definition those endpoit slices will not be created.
This opens up the possibility to route services to endpoints outside of your cluster, e.g. a database or a webserver hosted somewhere else. 
For this you have to define the endpoint slice by hand. In the endpointSlice Object definition, you map the service to the network address and port where the designated service runs. for example: 
```yaml title='endpointslice yaml example' fold
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: my-service-1 # by convention, use the name of the Service
                     # as a prefix for the name of the EndpointSlice
  labels:
    # You should set the "kubernetes.io/service-name" label.
    # Set its value to match the name of the Service
    kubernetes.io/service-name: my-service
addressType: IPv4
ports:
  - name: http # should match with the name of the service port defined above
    appProtocol: http
    protocol: TCP
    port: 9376
endpoints:
  - addresses:
      - "10.4.5.6"
  - addresses:
      - "10.1.2.3"
```
