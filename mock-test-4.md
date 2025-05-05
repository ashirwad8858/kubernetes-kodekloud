Solve this question on: ssh cluster1-controlplane


A template to create a Kubernetes pod is stored at /root/red-probe-cka12-trb.yaml on the cluster1-controlplane. However, using this template as is resulting in an error.


Fix the issue with this template and use it to create the pod. Once created, watch the pod for a minute or two to ensure it is stable, i.e., it's not crashing or restarting.


Do not update the args: section of the template.
--------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Try to apply the template
kubectl apply -f red-probe-cka12-trb.yaml 

You will see error:

error: error validating "red-probe-cka12-trb.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): unknown field "command" in io.k8s.api.core.v1.HTTPGetAction, ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): missing required field "port" in io.k8s.api.core.v1.HTTPGetAction]; if you choose to ignore these errors, turn validation off with --validate=false

From the error you can see that the error is for liveness probe, so let's open the template to find out:

vi red-probe-cka12-trb.yaml

Under livenessProbe: you will see the type is httpGet however the rest of the options are command based so this probe should be of exec type.

Change httpGet to exec
Try to apply the template now
kubectl apply -f red-probe-cka12-trb.yaml 

Cool it worked, now let's watch the POD status, after few seconds you will notice that POD is restarting. So let's check the logs/events

kubectl get event --field-selector involvedObject.name=red-probe-cka12-trb

You will see an error like:

21s         Warning   Unhealthy   pod/red-probe-cka12-trb   Liveness probe failed: cat: can't open '/healthcheck': No such file or directory

So seems like Liveness probe is failing, lets look into it:

vi red-probe-cka12-trb.yaml

Notice the command - sleep 3 ; touch /healthcheck; sleep 30;sleep 30000 it starts with a delay of 3 seconds, but the liveness probe initialDelaySeconds is set to 1 and failureThreshold is also 1. Which means the POD will fail just after first attempt of liveness check which will happen just after 1 second of pod start. So to make it stable we must increase the initialDelaySeconds to at least 5

vi red-probe-cka12-trb.yaml

Change initialDelaySeconds from 1 to 5 and save apply the changes.
Delete old pod:

kubectl delete pod red-probe-cka12-trb

Apply changes:

kubectl apply -f red-probe-cka12-trb.yaml

###############################################################################
###############################################################################
Solve this question on: ssh cluster3-controlplane


Utilize helm to search for the repository URL of the Bitnami version of the nginx repository. Ensure that you save the repository URL in the file located at /root/nginx-helm-url.txt on the cluster3-controlplane.
-----------------------------------------------------------------------
Solution
SSH into the Control Plane
Run the following command to access cluster3-controlplane:

ssh cluster3-controlplane

1. Check repo url for the nginx official repository
Run the Helm command below to list the repository URLs:


helm search hub nginx --list-repo-url | head -n15
URL                                                     CHART VERSION   APP VERSION                             DESCRIPTION                                             REPO URL                                          
https://artifacthub.io/packages/helm/krakazyabr...      1.0.0           1.19.0                                  Nginx Helm chart for Kubernetes                         https://krakazyabra.github.io/microservices       
https://artifacthub.io/packages/helm/onurs-repo...      0.2.0           1.20.2                                  Helm chart for nginx-1.20.2                             https://rezoruno.github.io/helm-charts

Capture the Repository URL
Store the official Nginx repository URL by executing the following command:

echo "https://charts.bitnami.com/bitnami" > /root/nginx-helm-url.txt
############################################################################
############################################################################
Solve this question on: ssh cluster4-controlplane
Identify and fix the issue that occurs while running kubectl commands on cluster4?
-----------------------------------------------------------------------
Step 1: SSH into the Control Plane Node
ssh cluster4-controlplane

Step 2: Inspect the kube-apiserver Pod
Check the status of the kube-apiserver static pod:

crictl ps | grep kube-apiserver

Get the logs:

crictl logs <container_id>

If the container is continuously restarting or the API server is unreachable via kubectl, the logs will help identify the issue.

Step 3: Notice Connection Errors in Logs
You will observe repeated errors similar to:

grpc: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2379"}.
Err: connection error: desc = "error reading server preface: EOF"

This indicates the API server is attempting to connect to etcd but failing to establish a valid connection.

Step 4: Inspect the Static Pod Manifest
Open the static pod manifest for the API server:

sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

Look for the --etcd-server flag in the container command section. If you see:

- --etcd-server=http://127.0.0.1:2379

That is incorrect. The flag name is missing an "s", and the protocol should be HTTPS (not HTTP).

Fix it:
- --etcd-servers=https://127.0.0.1:2379

Save and exit the file. The API server will auto-restart because static pods are monitored by the kubelet.

Step 5: Validate CA Certificate Path
If the logs now show a TLS error like:

transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate signed by unknown authority

This means the API server is reaching etcd, but the certificate authority (CA) being used is incorrect.

Open the manifest again:

sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

Look for this line:

- --etcd-cafile=/etc/kubernetes/pki/ca.crt

This is wrong. The API server should use the etcd-specific CA certificate.

Fix it:
- --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt

Save and exit.

Step 6: Verify Everything is Working
Check node and pod status:

kubectl get nodes
kubectl get pods -A

Monitor the kube-apiserver container to ensure it is not restarting:

crictl ps | grep kube-apiserver

You should no longer see error logs, and the control plane should be fully operational.
####################################################################################
####################################################################################
Solve this question on: ssh cluster1-controlplane


A deployment called nginx-dp-cka04-trb has been used to deploy a static website. The access to this website can be tested by running: curl http://cluster1-controlplane:30002. However, it is not working at the moment.


Troubleshoot and fix it.
----------------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

First list the available pods:

kubectl get pod

Look into the nginx-dp-xxxx POD logs

kubectl logs -f <pod-name>

You may not see any logs so look into the kubernetes events for <pod-name> POD

Look into the POD events

kubectl get event --field-selector involvedObject.name=<pod-name>

You will see an error something like:

70s         Warning   FailedMount   pod/nginx-dp-cka04-trb-767b767dc-6c5wk   Unable to attach or mount volumes: unmounted volumes=[nginx-config-volume-cka04-trb], unattached volumes=[index-volume-cka04-trb kube-api-access-4fbrb nginx-config-volume-cka04-trb]: timed out waiting for the condition

From the error we can see that its not able to mount nginx-config-volume-cka04-trb volume
Check the nginx-dp-cka04-trb deployment

kubectl get deploy nginx-dp-cka04-trb -o=yaml

Under volumes: look for the configMap: name which is nginx-configuration-cka04-trb. Now lets look into this configmap.

kubectl get configmap nginx-configuration-cka04-trb

The above command will fail as there is no configmap with this name, so now list the all configmaps.

kubectl get configmap

You will see an configmap named nginx-config-cka04-trb which seems to be the correct one.
Edit the nginx-dp-cka04-trb deployment now

kubectl edit deploy nginx-dp-cka04-trb

Under configMap: change nginx-configuration-cka04-trb to nginx-config-cka04-trb. Once done, wait for the POD to come up.

Try to access the website now:

curl http://cluster1-controlplane:30002
#################################################################
#################################################################
Solve this question on: ssh cluster1-controlplane


There is a requirement to share a volume between two containers that are running within the same pod. Use the following instructions to create the pod and related objects:


- Create a pod named grape-pod-cka06-str.

- The main container should use the nginx image and mount a volume called grape-vol-cka06-str at path /var/log/nginx.

- The sidecar container can use busybox image, you might need to add a sleep 7200 command to this container to keep it running. Next, mount the same volume called grape-vol-cka06-str at the path /usr/src.

- The volume should be of type emptyDir.

-------------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create a YAML file as below:
apiVersion: v1
kind: Pod
metadata:
  name: grape-pod-cka06-str
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
      - name: grape-vol-cka06-str
        mountPath: "/var/log/nginx"
  - name: busybox
    image: busybox
    command:
      - "bin/sh"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: grape-vol-cka06-str
        mountPath: "/usr/src"
  volumes:
  - name: grape-vol-cka06-str
    emptyDir: {}

Apply the template:
kubectl apply -f <template-file-name>.yaml
#############################################################################
#############################################################################
Solve this question on: ssh cluster1-controlplane


There is an existing persistent volume called orange-pv-cka13-trb. A persistent volume claim called orange-pvc-cka13-trb is created to claim storage from orange-pv-cka13-trb.


However, this PVC is stuck in a Pending state. As of now, there is no data in the volume.


Troubleshoot and fix this issue, ensuring that orange-pvc-cka13-trb PVC is in the Bound state.
-----------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

List the PVC to check its status
kubectl  get pvc

So we can see orange-pvc-cka13-trb PVC is in Pending state and its requesting a storage of 150Mi. Let's look into the events

kubectl get events --sort-by='.metadata.creationTimestamp' -A

You will see some errors as below:

Warning   VolumeMismatch            persistentvolumeclaim/orange-pvc-cka13-trb          Cannot bind to requested volume "orange-pv-cka13-trb": requested PV is too small

Let's look into orange-pv-cka13-trb volume

kubectl get pv

We can see that orange-pv-cka13-trb volume is of 100Mi capacity which means its too small to request 150Mi of storage.
Let's edit orange-pvc-cka13-trb PVC to adjust the storage requested.

kubectl get pvc orange-pvc-cka13-trb -o yaml > /tmp/orange-pvc-cka13-trb.yaml
vi /tmp/orange-pvc-cka13-trb.yaml

Under resources: -> requests: -> storage: change 150Mi to 100Mi and save.
Delete old PVC and apply the change:

kubectl delete pvc orange-pvc-cka13-trb
kubectl apply -f /tmp/orange-pvc-cka13-trb.yaml 
######################################################################
######################################################################
Solve this question on: ssh cluster2-controlplane


Identify the available CPU and memory resources available on cluster2-node01 node and save the results in /root/cluster2-node01-cpu.txt and /root/cluster2-node01-memory.txt, respectively, on the cluster2-controlplane.

Store the values in the following format:

<Resource-name>: <Value>
----------------------------------------------------------------------
SSH into the Control Plane
To access cluster2-controlplane, execute the following command:

ssh cluster2-controlplane

1. Retrieve Resource Capacity of the Node
To list the resource capacity of the node, run the command below:

kubectl describe node cluster2-node01 

You will see output similar to the following:

...
Capacity:
  cpu:                16
  ephemeral-storage:  772706776Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             65838280Ki
  pods:               110
...

Record the Resource Values
To store the official Nginx repository URL, execute the following commands:

echo "cpu: 16" > cluster2-node01-cpu.txt
echo "memory: 65838280Ki" > cluster2-node01-memory.txt

#############################################################################
#############################################################################
Solve this question on: ssh cluster1-controlplane


A pod called check-time-cka03-trb is continuously crashing. Figure out what is causing this and fix it.




Ensure that the check-time-cka03-trb POD is in the running state.

This pod prints the current date and time at a pre-defined frequency and saves it to a file. Ensure that it continues this operation once you have fixed it.
------------------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Look into the POD logs
kubectl logs -f check-time-cka03-trb

You may not see any logs so look into the kubernetes events for check-time-cka03-trb POD

Look into the POD events

kubectl get event --field-selector involvedObject.name=check-time-cka03-trb

You will see an error something like:

Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/bin/bash": stat /bin/bash: no such file or directory: unknown

From the error we can see that its not able to execute /bin/bash so let's try /bin/sh

Edit the pod
kubectl get pod check-time-cka03-trb -o=yaml > check-time-cka03-trb.yaml

Make the required changes in check-time-cka03-trb.yaml template
vi check-time-cka03-trb.yaml

Under spec: -> containers: -> command: change /bin/bash to /bin/sh and save the file.
Delete the old pod.
kubectl delete pod check-time-cka03-trb

Apply the updated template
kubectl apply -f check-time-cka03-trb.yaml
##########################################################################
##########################################################################
Solve this question on: ssh cluster1-controlplane


An etcd backup is already stored at the path /opt/cluster1_backup_to_restore.db on the cluster1-controlplane node. Use /root/default.etcd as the --data-dir and restore it on the cluster1-controlplane node itself.
------------------------------------------------------------------------
SSH into cluster1-controlplane node:

student-node ~ ➜ ssh root@cluster1-controlplane




Install etcd utility (if not installed already) and restore the backup:

cluster1-controlplane ~ ➜ cd /tmp
cluster1-controlplane ~ ➜ export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep tag_name | cut -d '"' -f 4)
cluster1-controlplane ~ ➜ wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
cluster1-controlplane ~ ➜ tar xvf etcd-${RELEASE}-linux-amd64.tar.gz ; cd etcd-${RELEASE}-linux-amd64
cluster1-controlplane ~ ➜ mv etcd etcdctl etcdutl /usr/local/bin/
cluster1-controlplane ~ ➜ etcdutl snapshot restore --data-dir /root/default.etcd /opt/cluster1_backup_to_restore.db 

Details
#########################################################################
############################################################################
Solve this question on: ssh cluster3-controlplane


Create an HPA named webapp-hpa for a deployment named webapp-deployment in the default namespace with service webapp-service associated to it.

The HPA should scale the deployment based on CPU utilization, maintaining an average CPU usage of 50% across all pods with a minimum of 1 and a maximum of 3 replicas.

Configure the HPA to scale down pods cautiously by setting a stabilization window of 300 seconds to prevent rapid fluctuations in pod count.
--------------------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane

Verify that the webapp-deployment exists:

kubectl get deployment webapp-deployment

Create a file named webapp-hpa.yaml with the following content:

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300

Apply the HPA configuration:

kubectl apply -f webapp-hpa.yaml

Verify that the HPA has been created with the correct configuration:

kubectl get hpa webapp-hpa
kubectl describe hpa webapp-hpa

The output should show the target CPU utilization of 50% and the scale-down stabilization window of 300 seconds.

Understanding Stabilization Windows in HPA
Stabilization windows help prevent rapid fluctuations in the number of replicas during scaling events, particularly when metrics fluctuate around the threshold.

When you set a stabilizationWindowSeconds value for scaling down:

The HPA controller keeps track of the desired number of replicas over a period of time (the stabilization window)
When deciding whether to scale down, it uses the highest recommendation within that window
This prevents "flapping" where pods would be repeatedly created and deleted in response to brief metric changes
In this case, the 300-second (5-minute) stabilization window means that the HPA will only scale down if the recommendation to use fewer replicas has been consistent for at least 5 minutes. This helps maintain system stability and prevents unnecessary pod terminations.

You can also set a separate stabilization window for scaling up by configuring the scaleUp.stabilizationWindowSeconds parameter. When not specified, Kubernetes uses default values of 300 seconds for scale-down and 0 seconds for scale-up.

#######################################################################
#######################################################################
Solve this question on: ssh cluster3-controlplane


A manifest file is available at the /root/app-wl03/ on the student-node node. The file has some issues; we couldn't deploy a pod on the cluster3-controlplane node.

After fixing the issues, deploy the pod, and it should be in a running state.


NOTE: - Ensure that the existing limits are unchanged.
-------------------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane



Use the cd command to move to the given directory: -

cd /root/app-wl03/

While creating the resource, you will see the error output as follows: -

kubectl create -f app-wl03.yaml 
The Pod "app-wl03" is invalid: spec.containers[0].resources.requests: Invalid value: "1Gi": must be less than or equal to memory limit

In the spec.containers.resources.requests.memory value is not configured as compare to the memory limit.

As a fix, open the manifest file with the text editor such as vim or nano and set the value to 100Mi or less than 100Mi.

It should be look like as follows: -

resources:
     requests:
       memory: 100Mi
     limits:
       memory: 100Mi

Final, create the resource from the kubectl create command: -

kubectl create -f app-wl03.yaml 
pod/app-wl03 created
#############################################################
#############################################################
Solve this question on: ssh cluster1-controlplane


John is setting up a two-tier application stack that is supposed to be accessible using the service curlme-cka01-svcn. To test that the service is accessible, he is using a pod called curlpod-cka01-svcn. However, at the moment, he is unable to get any response from the application.



Troubleshoot and fix this issue to ensure the application stack is accessible.



While you may delete and recreate the service curlme-cka01-svcn, please do not alter it in anyway.
------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Test if the service curlme-cka01-svcn is accessible from pod curlpod-cka01-svcn or not.


kubectl exec curlpod-cka01-svcn -- curl curlme-cka01-svcn

.....
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0




We did not get any response. Check if the service is properly configured or not.


kubectl describe svc curlme-cka01-svcn 

....
Name:              curlme-cka01-svcn
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          run=curlme-ckaO1-svcn
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.45.180
IPs:               10.109.45.180
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>




The service has no endpoints configured. As we can delete the resource, let's delete the service and create the service again.

To delete the service, use the command kubectl delete svc curlme-cka01-svcn.
You can create the service using imperative way or declarative way.


Using imperative command:
kubectl expose pod curlme-cka01-svcn --port=80



Using declarative manifest:


apiVersion: v1
kind: Service
metadata:
  labels:
    run: curlme-cka01-svcn
  name: curlme-cka01-svcn
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: curlme-cka01-svcn
  type: ClusterIP




You can test the connection from curlpod-cka-1-svcn using following.


kubectl exec curlpod-cka01-svcn -- curl curlme-cka01-svcn
####################################################################################
####################################################################################
Solve this question on: ssh cluster3-controlplane


A pod called elastic-app-cka02-arch is running in the default namespace. The YAML file for this pod is available at /root/elastic-app-cka02-arch.yaml on the student-node. The single application container in this pod writes logs to the file /var/log/elastic-app.log.


One of our logging mechanisms needs to read these logs to send them to an upstream logging server, but we don't want to increase the read overhead for our main application container. So, you need to recreate this POD with an additional sidecar container that will run along with the application container and print to the STDOUT by running the command tail -f /var/log/elastic-app.log. You can use the busybox image for this sidecar container.
--------------------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane

Recreate the pod with a new container called sidecar. Update the /root/elastic-app-cka02-arch.yaml YAML file as shown below:

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
  - name: sidecar
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -f  /var/log/elastic-app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}




Next, recreate the pod:

student-node ~ ➜ kubectl replace -f /root/elastic-app-cka02-arch.yaml --force
pod "elastic-app-cka02-arch" deleted
pod/elastic-app-cka02-arch replaced

student-node ~ ➜ 
###################################################################
###################################################################
Solve this question on: ssh cluster3-controlplane


The CoreDNS configuration in the cluster needs to be updated:

Update the CoreDNS configuration in the cluster so that DNS resolution for cka.local works exactly like cluster.local and in addition to it.

Test your configuration using the jrecord/nettools image by executing the following commands:

nslookup kubernetes.default.svc.cluster.local
nslookup kubernetes.default.svc.cka.local
---------------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane

1. Update the CoreDNS Configuration
The CoreDNS configuration is stored in a ConfigMap. To edit this configuration, use the following command:

kubectl edit cm -n kube-system coredns

Below is the structure of the CoreDNS configuration:

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

2. Restart the CoreDNS Deployment
To apply the changes, restart the CoreDNS deployment using the command:

kubectl rollout restart deploy -n kube-system coredns

3. Verify the Changes
To verify the updates, execute the following commands:

kubectl run test --rm -it --image=jrecord/nettools --restart=Never -- nslookup kubernetes.default.svc.cluster.local
kubectl run test --rm -it --image=jrecord/nettools --restart=Never -- nslookup kubernetes.default.svc.cka.local

##################################################################################
##################################################################################
Solve this question on: ssh cluster1-controlplane


We have a deployment named external-app and a service associated with it named external-service in the external namespace already created.

There is also an existing Ingress resource named external-ingress in the external namespace.

Create HTTPRoute to achieve the same behavior as external-ingress.

Part 1

Create an HTTP Gateway named web-gateway in the nginx-gateway namespace, which uses the nginx gateway class, and listens on port 80, and allows routes from all namespaces.
Part 2

Create an HTTPRoute named external-route in the external namespace that binds to the web-gateway in the default namespace and routes traffic to the external-service in the external namespace.
Part 3

Remove the existing Ingress resource from the external namespace.
Hit curl on the gateway to test the service.

# This is the node port where gatewayclass is attached to
curl localhost:30080
-----------------------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Step 1: Create the GatewayClass named nginx and create a Gateway in the default namespace
Create a file named nginx-gateway.yaml:

# Create the Gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All

Note: The allowedRoutes.namespaces.from: All setting explicitly allows routes from all namespaces to bind to this Gateway, which is required for our cross-namespace routing scenario.

Apply it:

kubectl apply -f nginx-gateway.yaml

Step 2: Create the HTTPRoute in the external namespace
First, review the existing ingress configuration:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  creationTimestamp: "2025-03-24T05:21:34Z"
  generation: 1
  name: external-ingress
  namespace: external
  resourceVersion: "6592"
  uid: 2ec75a22-fa6d-45b8-90d8-29273a8fbd0c
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: external-service
            port:
              number: 80
        path: /
        pathType: Prefix

The ingress configuration routes all traffic to the external-service on port 80.

Now, create a file named external-route.yaml with the following content:

apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: external-route
  namespace: external
spec:
  parentRefs:
  - name: web-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: external-service
      port: 80

Then, apply the HTTPRoute configuration:

kubectl apply -f external-route.yaml

Step 3: Remove the existing ingress resource
To delete the existing ingress resource, execute the following command:

kubectl delete -n external ingress external-ingress 

Step 4: Verify that the HTTPRoute is created and configured correctly
Describe the HTTPRoute to see its configuration:

kubectl describe httproute external-route -n external

Step 5: Test the routing functionality
Get the external IP of the Gateway:

kubectl get gateway web-gateway -n nginx-gateway

Test accessing the service using curl:

curl localhost:30080

You should see traffic being routed to the external-service in the external namespace.
########################################################################
########################################################################
Solve this question on: ssh cluster2-controlplane


Utilize the official flannel definition file, located at https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml, to deploy the flannel CNI on the cluster using the CIDR 172.17.0.0/16. Ensure pods can communicate after the CNI is installed.

------------------------------------------------------------------------
SSH into the cluster2-controlplane host
ssh cluster2-controlplane

1. Install the Flannel CNI
Download the Flannel manifest file:
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

Edit the Flannel manifest to set the CIDR to 172.17.0.0/16:
vi kube-flannel.yml

Below is the structure of the CoreDNS configuration:

apiVersion: v1
data:
  cni-conf.json: |
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
  net-conf.json: |
    {
      "Network": "172.17.0.0/16",   #Edited
      "EnableNFTables": false,
      "Backend": {
        "Type": "vxlan"
      }
    }
kind: ConfigMap
metadata:
  labels:
    app: flannel
    k8s-app: flannel
    tier: node
  name: kube-flannel-cfg
  namespace: kube-flannel

Apply the manifest by running the following command:
kubectl apply -f kube-flannel.yml

2. Test the pod-to-pod communication.
Run an nginx image:
kubectl run web-app --image nginx

Retrieve the IP address of the pod:
kubectl get pod web-app -o jsonpath='{.status.podIP}'

Test the connection:
kubectl run test --rm -it -n kube-public --image=jrecord/nettools --restart=Never -- curl <IP>

Details
####################################################################
#####################################################################
Solve this question on: ssh cluster5-controlplane


A debian package for cri-dockerd cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb is located in the /root folder on the cluster5-controlplane. As part of cluster initialization, install the package and make sure that cri-docker service is up and enabled on the system. Additionally, enable IP forwarding on the server.
----------------------------------------------------------------
SSH into the Control Plane
To access cluster5-controlplane, execute the following command:

ssh cluster5-controlplane

1. Install cri-dockerd
The Debian package for cri-dockerd is available at cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb. Install it using the following command:

dpkg -i cri-dockerd_0.3.16.3-0.ubuntu-jammy_amd64.deb 

2. Enable and Start the Service
To enable and start the cri-dockerd service, use the following command:

sudo systemctl enable --now cri-docker.service

3. Enable IP Forwarding
To ensure that IP forwarding changes are persistent:

Create a configuration file:
   vi /etc/sysctl.d/k8s.conf

Add the following line to the file:
   net.ipv4.ip_forward=1

Apply the changes:
   sysctl -p

To set the changes temporarily, execute the following command:

sysctl -w net.ipv4.ip_forward=1
