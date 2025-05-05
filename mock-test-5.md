Solve this question on: ssh cluster1-controlplane


​A pod called nginx-cka01-trb is running in the default namespace. Inside this pod, there is a container called nginx-container running that uses the image nginx:latest. Another sidecar container called logs-container runs in this pod.

For some reason, this pod is continuously crashing. Identify the issue and fix it. Make sure that the pod is in a running state, and you can access the website using the curl http://cluster1-controlplane:30001 command on the controlplane node of cluster1.

---------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Check the container logs:
kubectl logs -f nginx-cka01-trb -c nginx-container

You can see that its not able to pull the image.

Edit the pod
kubectl edit pod nginx-cka01-trb -o yaml

Change image tag from nginx:latst to nginx:latest
Let's check now if the POD is in Running state

kubectl get pod

You will notice that its still crashing, so check the logs again:

kubectl logs -f nginx-cka01-trb -c nginx-container

From the logs you will notice that nginx-container is looking good now so it might be the sidecar container that is causing issues. Let's check its logs.

kubectl logs -f nginx-cka01-trb -c logs-container

You will see some logs as below:

cat: can't open '/var/log/httpd/access.log': No such file or directory
cat: can't open '/var/log/httpd/error.log': No such file or directory

Now, let's look into the sidecar container

kubectl get pod nginx-cka01-trb -o yaml

Under containers: check the command: section, this is the command which is failing. If you notice its looking for the logs under /var/log/httpd/ directory but the mounted volume for logs is /var/log/nginx (under volumeMounts:). So we need to fix this path:

kubectl get pod nginx-cka01-trb -o yaml > /tmp/test.yaml
vi /tmp/test.yaml

Under command: change /var/log/httpd/access.log and /var/log/httpd/error.log to /var/log/nginx/access.log and /var/log/nginx/error.log respectively.

Delete the existing POD now:

kubectl delete pod nginx-cka01-trb

Create new one from the template

kubectl apply -f /tmp/test.yaml

Let's check now if the POD is in Running state

kubectl get pod

It should be good now. So let's try to access the app.

curl http://cluster1-controlplane:30001

You will see error

curl: (7) Failed to connect to cluster1-controlplane port 30001: Connection refused

So you are not able to access the website, et's look into the service configuration.

Edit the service
kubectl edit svc nginx-service-cka01-trb -o yaml 

Change app label under selector from httpd-app-cka01-trb to nginx-app-cka01-trb
You should be able to access the website now.
curl http://cluster1-controlplane:30001
#######################################################################
#####################################################################
Solve this question on: ssh cluster2-controlplane


Inspect the user-defined priority classes on the cluster and output the highest user-defined priority class value to /root/highest-user-prio.txt on cluster2-controlplane.

NOTE: Only the value for the priority class should be recorded.
---------------------------
SSH into the cluster2-controlplane host
ssh cluster2-controlplane

Step 1: Find the highest user-defined priority classes
Run the below command to list all the priority classes in cluster.

student-node ~ ✖ kubectl get priorityclasses.scheduling.k8s.io 
NAME                      VALUE        GLOBAL-DEFAULT   AGE
bronze-tier               997          false            3m2s
burst-mode                998          false            3m2s
default-tier              500          true             3m2s
gold-tier                 998          false            3m2s
silver-tier               999          false            3m2s
system-cluster-critical   2000000000   false            74m
system-node-critical      2000001000   false            74m

The silver-tier priority class has highest user defined class.

Step 2: Store the value
Run the below command

echo "999" > /root/highest-user-prio.txt
###################################################################
####################################################################
Solve this question on: ssh cluster5-controlplane


You are an administrator preparing your environment to deploy a kubernetes cluster using kubeadm. Adjust the following network parameters on the system to the following values, and make sure your changes persist reboots:

net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
net.ipv4.conf.all.forwarding = 1
----------------------------------
SSH into the Control Plane
To access cluster5-controlplane, execute the following command:

ssh cluster5-controlplane

1. Setup Network Configuration for Kubernetes Cluster
To ensure consistent network changes across boots, create a configuration file by executing:

vi /etc/sysctl.d/k8s.conf

Add the following values to the file:

net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.ipv4.conf.all.forwarding        = 1

To apply the changes, run the following command:

sysctl --system
###################################################################
###############################################################
Solve this question on: ssh cluster2-controlplane


In the cka-multi-containers namespace, proceed to create a pod named cka-sidecar-pod that adheres to the following specifications:

The first container, labeled main-container, is required to run the nginx:1.27, which writes the current date along with a greeting message Hi I am from Sidecar container to /log/app.log.
The second container, identified as sidecar-container, must use the nginx:1.25 image and serve the app.log file as a webpage located at /usr/share/nginx/html.
Note: Do not rename app.log to index.html. The file name should remain app.log and be available at /app.log via the Nginx server.
-----------------------------------------------------------
SSH into the Control Plane
To access cluster2-controlplane, please execute the following command:

ssh cluster2-controlplane

Step 1: Deploy the cka-sidecar-pod
Utilize the YAML configuration provided below to create the desired pod:

apiVersion: v1
kind: Pod
metadata:
  name: cka-sidecar-pod
  namespace: cka-multi-containers
spec:
  containers:
    - name: main-container
      image: nginx:1.27
      command: ["/bin/sh"]
      args:
        - -c
        - |
          while true; do
            echo "$(date) Hi I am from Sidecar container" >> /log/app.log;
            sleep 5;
          done
      volumeMounts:
        - name: shared-logs
          mountPath: /log
    - name: sidecar-container
      image: nginx:1.25
      volumeMounts:
        - name: shared-logs
          mountPath: /usr/share/nginx/html
  volumes:
    - name: shared-logs
      emptyDir: {}

Step 2: Verify the Pod
After waiting for the pod to become ready, execute the following command inside the pod to ensure that the sidecar-container is properly exposing the logs:

kubectl exec -it -n cka-multi-containers cka-sidecar-pod -c sidecar-container -- curl http://localhost:80/app.log

You should observe output similar to the following:

Tue Mar 25 05:31:19 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:24 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:29 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:34 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:39 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:44 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:49 UTC 2025 Hi I am from Sidecar container
Tue Mar 25 05:31:54 UTC 2025 Hi I am from Sidecar container
########################################################################
########################################################################
Solve this question on: ssh cluster1-controlplane


Create a VPA named api-vpa in Auto Mode for a deployment named api-deployment in the services namespace. The VPA should automatically adjust CPU and memory requests but must ensure that the CPU requests do not exceed 1 cores and memory requests do not exceed 1Gi. Additionally, set a minimum CPU request of 600m and a minimum memory request of 600Mi.

The containerName in VPA should explicitly match the container name inside api-deployment.
--------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Next, check the deployment in the "services" namespace by running:

kubectl get deploy -n services

To create a Vertical Pod Autoscaler (VPA) in auto mode that scales the application within the specified resource constraints, utilize the following configuration:

apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
  namespace: services
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: api-container
      minAllowed:
        cpu: "600m"
        memory: "600Mi"
      maxAllowed:
        cpu: "1"
        memory: "1Gi"
		
######################################################################
###################################################################
Solve this question on: ssh cluster3-controlplane


On cluster3, there is a web application pod running inside the default namespace. This pod is part of a deployment called webapp-color-wl10 and uses an environment variable that can change constantly. Add this environment variable to a configmap and configure the pod in the deployment to make use of this config map.

Use the following specs-


1. Create a new configMap called webapp-wl10-config-map with the key and value as - APP_COLOR=red.

2. Update the deployment to make use of the newly created configMap name.

3. Delete and recreate the deployment if necessary.
---------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane



Inspect the given pod in the default namespace: -

kubectl get pods -n default



Let's create a new configMap in the default namespace as follows:-

kubectl create configmap webapp-wl10-config-map --from-literal=APP_COLOR=red



And now configure the newly created configmap to the web application pod's template: -

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webapp-color-wl10
  name: webapp-color-wl10
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp-color-wl10
  template:
    metadata:
      labels:
        app: webapp-color-wl10
    spec:
      containers:
      - image: kodekloud/webapp-color
        name: webapp-color-wl10
        envFrom:
        - configMapRef: 
            name: webapp-wl10-config-map



NOTE: - It will terminate the old pods and will recreate the new pods with configMap.



You can inspect the newly created pod to check the configMap: -

kubectl describe po webapp-color-wl10-d79c6f76c-4gjmz



Note: - In your lab, pod name could be different.
############################################################
#############################################################
Solve this question on: ssh cluster1-controlplane



One co-worker deployed a fluent-bit helm chart on the cluster1 server called lvm-crystal-apd. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.
After updating the helm chart, upgrade its version to 0.48.5 and increase the replica count to 3.
NOTE: - We have to perform this task on the cluster1-controlplane node.
You can SSH into the cluster1 using ssh cluster1-controlplane command.
---------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane



In this task, we will use the kubectl and helm commands. Here are the steps: -



Log in to the cluster1-controlplane node first and use the helm ls command to list all the releases installed using Helm in the Kubernetes cluster.

helm ls -A



Here -A or --all-namespaces option lists all the releases of all the namespaces.



Identify the namespace where the resources get deployed.
Use the helm repo ls command to list the helm repositories.
helm repo ls 

Now, update the helm repository with the following command: -

helm repo update lvm-crystal-apd -n crystal-apd-ns



The above command updates the local cache of available charts from the configured chart repositories.


The helm search command searches for all the available charts in a specific Helm chart repository. In our case, it's the fluent-bit helm chart.
helm search repo lvm-crystal-apd/fluent-bit -n crystal-apd-ns -l | head -n30



The -l or --versions option is used to display information about all available chart versions.



Upgrade the helm chart to 0.48.5 and also, increase the replica count of the deployment to 3 from the command line. Use the helm upgrade command as follows: -

helm upgrade lvm-crystal-apd lvm-crystal-apd/fluent-bit -n crystal-apd-ns --version=0.48.5 --set replicaCount=3 --set kind=Deployment



After upgrading the chart version, you can verify it with the following command: -

helm ls -n crystal-apd-ns



Look under the CHART column for the chart version.



Use the kubectl get command to check the replicas of the deployment: -
kubectl get deploy -n crystal-apd-ns



The available count 3 is under the AVAILABLE column.
##########################################################################
######################################################################
Solve this question on: ssh cluster2-controlplane


We recently deployed a DaemonSet called logs-cka26-trb under kube-system namespace in cluster2 for collecting logs from all the cluster nodes including the controlplane node. However, at this moment, the DaemonSet is not creating any pod on the controlplane node.


Troubleshoot the issue and fix it to make sure the pods are getting created on all nodes including the controlplane node.
--------------------------------------------------
SSH into the cluster2-controlplane host
ssh cluster2-controlplane

Check the status of DaemonSet

kubectl get ds logs-cka26-trb -n kube-system

Verify the Number of Nodes

kubectl get nodes

You will observe that the values for DESIRED, CURRENT, READY, etc., are not equal to 2. This indicates that a total of two pods were expected to be created; however, only one pod has been created. You can confirm this by listing the pods:

kubectl get pod  -n kube-system

You can check on which nodes these are created on

kubectl get pod <pod-name> -n kube-system -o wide

Under NODE you will find the node name, so we can see that its not scheduled on the controlplane node which is because it must be missing the reqiured tolerations. Let's edit the DaemonSet to fix the tolerations

kubectl edit ds logs-cka26-trb -n kube-system

Under tolerations: add below given tolerations as well

- key: node-role.kubernetes.io/control-plane
  operator: Exists
  effect: NoSchedule

Wait for some time PODs should schedule on all nodes now including the controlplane node.

#################################################################################
###############################################################################
Solve this question on: ssh cluster3-controlplane


There is a deployment nginx-deployment-cka04-svcn in cluster3 which is exposed using service nginx-service-cka04-svcn.



Create an ingress resource nginx-ingress-cka04-svcn to load balance the incoming traffic with the following specifications:



pathType: Prefix and path: /

Backend Service Name: nginx-service-cka04-svcn

Backend Service Port: 80

ssl-redirect is set to false

------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane




Now apply the ingress resource with the given requirements:



kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress-cka04-svcn
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service-cka04-svcn
            port:
              number: 80
EOF




Check if the ingress resource was successfully created:



student-node ~ ➜  kubectl get ingress
NAME                       CLASS    HOSTS   ADDRESS       PORTS   AGE
nginx-ingress-cka04-svcn   <none>   *       172.25.0.10   80      13s




As the ingress controller is exposed on cluster3-controlplane using traefik service, check if the ingress resource works properly:



student-node ~ ➜  ssh cluster3-controlplane

cluster3-controlplane:~# curl -I 10.43.178.15
HTTP/1.1 200 OK
...

##################################################################
##################################################################
Solve this question on: ssh cluster1-controlplane


A storage class called coconut-stc-cka01-str was created earlier.


Use this storage class to create a persistent volume called coconut-pv-cka01-str as per below requirements:


- Capacity should be 100Mi.

- The volume type should be hostpath and the path should be /opt/coconut-stc-cka01-str.

- Use coconut-stc-cka01-str storage class.

- This volume must be created on cluster1-node01 (the /opt/coconut-stc-cka01-str directory already exists on this node).

- It must have a label with key: storage-tier with value: gold.


Also, create a persistent volume claim with the name coconut-pvc-cka01-str as per the below specs:


- Request 50Mi of storage from coconut-pv-cka01-str PV. It must use matchLabels to use the PV.

- Use coconut-stc-cka01-str storage class.

- The access mode must be ReadWriteMany.
-------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create a yaml template as below:
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: coconut-pv-cka01-str
  labels: 
    storage-tier: gold
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /opt/coconut-stc-cka01-str
  storageClassName: coconut-stc-cka01-str
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - cluster1-node01
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: coconut-pvc-cka01-str
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: coconut-stc-cka01-str
  selector: 
    matchLabels:
      storage-tier: gold

Apply the template.
kubectl apply -f <template-name>.yaml

###################################################################
###################################################################

Solve this question on: ssh cluster2-controlplane


A Kustomize configuration is located at /root/web-dashboard-kustomize on cluster2-controlplane. This application is designed to monitor the pods in the default namespace. It has been deployed using the following command:

kubectl kustomize /root/web-dashboard-kustomize/overlays/dev | kubectl apply -f -

The application is currently unable to monitor the pods due to insufficient permissions. Modify the Kustomize overlays/dev configuration to ensure the application is operational.

The application is currently unable to monitor the pods due to insufficient permissions. Adjust the Kustomize configuration accordingly to ensure the application is operational.
--------------------------------------------------------
SSH into the Control Plane
To access cluster2-controlplane, please execute the following command:

ssh cluster2-controlplane

Step 1: Understand Kustomize Configuration
To review the resources that Kustomize deploys for the application, execute the command below:

kubectl kustomize /root/web-dashboard-kustomize/overlays/dev

This configuration deploys a deployment, service, role, and role binding.

Step 2: Check the Application Logs
To investigate the reason for the application's failure, run the following command:

cluster2-controlplane ~ ➜  kubectl logs deploy/web-dashboard 

You may see an error message resembling the following:

🔄 Checking pods...
❌ Error: (403)
Reason: Forbidden
HTTP response headers: HTTPHeaderDict({'Audit-Id': '17982343-1d59-48fa-b823-705dcea07343', 'Cache-Control': 'no-cache, private', 'Content-Type': 'application/json', 'X-Content-Type-Options': 'nosniff', 'X-Kubernetes-Pf-Flowschema-Uid': 'a7bfc6de-292c-48d3-bd63-8e625fdbc639', 'X-Kubernetes-Pf-Prioritylevel-Uid': '7e373b29-1156-4299-b472-203a7df999f2', 'Date': 'Mon, 24 Mar 2025 12:21:02 GMT', 'Content-Length': '287'})
HTTP response body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"pods is forbidden: User \"system:serviceaccount:default:dashboard-sa\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"","reason":"Forbidden","details":{"kind":"pods"},"code":403}

The application is failing due to insufficient permissions.

Step 3: Fix the Kustomize Configuration
Update the pod-reader role in /root/web-dashboard-kustomize/overlays/dev/patch-role.yaml as follows:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

After making the necessary updates, apply the changes using the Kustomize configuration with the following command:

kubectl kustomize /root/web-dashboard-kustomize/overlays/dev | kubectl apply -f -

####Step 4: Verify the Application Status

To confirm whether the application is running, execute the following command:

kubectl get pods

#########################################################################################
##########################################################################################
Solve this question on: ssh cluster1-controlplane


The deployment called web-dp-cka17-trb has 0 out of 1 pods up and running. Troubleshoot this issue and fix it. Make sure all required POD(s) are in running state and stable (not restarting).

The application runs on port 80 inside the container and is exposed on the node port 30090.
------------------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

List out the PODs
kubectl get pod

Let's look into the relevant events:

kubectl get event --field-selector involvedObject.name=<pod-name>

You should see some errors as below:

Warning   FailedScheduling   pod/web-dp-cka17-trb-9bdd6779-fm95t   0/3 nodes are available: 3 persistentvolumeclaim "web-pvc-cka17-trbl" not found. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

From the error we can see that its something related to the PVCs. So let' look into that.

kubectl get pv
kubectl get pvc

You will notice that web-pvc-cka17-trb is stuck in pending and also the capacity of web-pv-cka17-trb volume is 100Mi.
Now let's dig more into the PVC:

kubectl get pvc web-pvc-cka17-trb -o yaml

Notice the storage which is 150Mi which means its trying to claim 150Mi of storage from a 100Mi PV. So let's edit this PV.

kubectl edit pv web-pv-cka17-trb

Change storage: 100Mi to storage: 150Mi
Check again the pvc
kubectl get pvc

web-pvc-cka17-trb should be good now. let's see the PODs

kubectl get pod

POD should not be in pending state now but it must be crashing with Init:CrashLoopBackOff status, which means somehow the init container is crashing. So let's check the logs.

kubectl get event --field-selector involvedObject.name=<pod-name>

You should see someting like

Warning   Failed      pod/web-dp-cka17-trb-67c9bdcd85-4tvpr   Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/bin/bsh\\": stat /bin/bsh\: no such file or directory: unknown

Let's look into the deployment:

kubectl edit deploy web-dp-cka17-trb

Under initContainers: -> - command: change /bin/bsh\ to /bin/bash
let's see the PODs
kubectl get pod

Wait for some time to make sure it is stable, but you will notice that its restart so still something must be wrong.

So let's check the events again.

kubectl get event --field-selector involvedObject.name=<pod-name>

You should see someting like

Warning   Unhealthy   pod/web-dp-cka17-trb-647f69f8bd-67xmx   Liveness probe failed: Get "http://10.50.64.1:81/": dial tcp 10.50.64.1:81: connect: connection refused

Seems like its not able to connect to a service, let's look into the deployment to understand

kubectl edit deploy web-dp-cka17-trb

Notice that containerPort: 80 but under livenessProbe: the port: 81 so seems like livenessProbe is using wrong port. let's change port: 81 to port: 80

See the PODs now

kubectl get pod

It should be good now.
####################################################################
###################################################################
Solve this question on: ssh cluster3-controlplane


Create the web-app-route in the ck2145 namespace. This route should direct requests that contain the header X-Environment: canary to the web-service-canary on port 8080. All other traffic should continue to be routed to web-service also on port 8080.

Note: Gateway has already been created in the nginx-gateway namespace.
To test the gateway, execute the following command:

curl -H 'X-Environment: canary' http://localhost:30080
-----------------------------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane



To direct traffic based on the X-Environment: canary HTTP header to web-service-canary on port 8080, while routing the rest of the traffic to web-service on port 8080, please utilize the manifest provided below:

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-app-route
  namespace: ck2145
spec:
  parentRefs:
  - name: nginx-gateway 
    namespace: nginx-gateway
  rules:
  - matches:
    - headers:
      - name: X-Environment
        value: canary
    backendRefs:
    - name: web-service-canary
      port: 8080
  - backendRefs:
    - name: web-service
      port: 8080
	  
#############################################################################
###############################################################################
Solve this question on: ssh cluster3-controlplane


Create a Horizontal Pod Autoscaler (HPA) named backend-hpa in the cka0841 namespace for a deployment named backend-deployment, which scales based on CPU usage.

The HPA should be configured with:

A minimum of 3 replicas
A maximum of 15 replicas
Specify a resource-based metric to scale based on CPU utilization, maintaining the average utilization of the pods at 50% of the requested CPU.

Additionally, set the scale-down behavior to allow:

Reducing the number of pods by a maximum of 5 at a time
Or by 20% of the current replica count, whichever results in fewer pods being removed
This should occur within a time frame of 60 seconds.
-------------------------------------------------
SSH into the cluster3-controlplane host
ssh cluster3-controlplane



To create a Horizontal Pod Autoscaler (HPA) for the backend-deployment in the cka0841 namespace based on CPU utilization, and to enable scaling down by either 20% of the current replicas or a maximum reduction of 5 pods, utilize the following configuration:

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: cka0841
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      policies:
      - type: Pods
        value: 5
        periodSeconds: 60
      - type: Percent
        value: 20
        periodSeconds: 60
      selectPolicy: Min
	  
	  
############################################################################
############################################################################
Solve this question on: ssh cluster1-controlplane


Create a storage class with the name banana-sc-cka08-str as per the following specifications:


- Provisioner should be kubernetes.io/no-provisioner.

- Volume binding mode should be WaitForFirstConsumer.

- Volume expansion should be enabled.
----------------------------------------------------------------
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create a yaml template as below:
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-cka08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer

Apply the template:
kubectl apply -f <template-file-name>.yaml

###############################################################################
################################################################################
Solve this question on: ssh cluster4-controlplane


Utilize the official Calico definition file, available at Calico Installation Guide, to deploy the Calico CNI on the cluster.

Make sure to configure the CIDR to 172.17.0.0/16 and set the encapsulation method to VXLAN.

After the CNI installation, verify that pods can successfully communicate.
-------------------------------------------------------------------------------
SSH into the cluster4-controlplane host
ssh cluster4-controlplane

1. Install the Calico CNI
Install the operator on your cluster:
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml

Download the custom resources necessary to configure Calico to set the CIDR to 172.17.0.0/16 and set the encapsulation method to VXLAN:
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O

Below is the structure of the final configuration:

# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 172.17.0.0/16
      encapsulation: VXLAN
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}

Apply the manifest by running the following command:
kubectl create -f custom-resources.yaml

Verify Calico installation in your cluster.
watch kubectl get pods -n calico-system

2. Test the pod-to-pod communication.
Run an nginx image:
kubectl run web-app --image nginx

Retrieve the IP address of the pod:
kubectl get pod web-app -o jsonpath='{.status.podIP}'

Test the connection:
kubectl run test --rm -it -n kube-public --image=jrecord/nettools --restart=Never -- curl <IP>

