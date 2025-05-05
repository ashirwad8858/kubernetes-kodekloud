Solve this question on: ssh cluster1-controlplane


Create a storage class called orange-stc-cka07-str as per the properties given below:


Provisioner should be kubernetes.io/no-provisioner.
Volume binding mode should be WaitForFirstConsumer.

Next, create a persistent volume called orange-pv-cka07-str as per the properties given below:

Capacity should be 150Mi.
Access mode should be ReadWriteOnce.
Reclaim policy should be Retain.
It should use storage class orange-stc-cka07-str.
Local path should be /opt/orange-data-cka07-str.
Also add node affinity to create this value on cluster1-controlplane.

Finally, create a persistent volume claim called orange-pvc-cka07-str as per the properties given below:

Access mode should be ReadWriteOnce.
It should use storage class orange-stc-cka07-str.
Storage request should be 128Mi.
The volume should be orange-pv-cka07-str.

Solution--
Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create a yaml file as below:
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: orange-stc-cka07-str
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer

---
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
  local:
    path: /opt/orange-data-cka07-str
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cluster1-controlplane

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: orange-pvc-cka07-str
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: orange-stc-cka07-str
  volumeName: orange-pv-cka07-str
  resources:
    requests:
      storage: 128Mi

Apply the template:
kubectl apply -f <template-file-name>.yaml
#####################################################
Find the node across all cluster1, cluster3, and cluster4 that consumes the most CPU and store the result to the file /opt/high_cpu_node in the following format cluster_name,node_name.

The node could be in any clusters that are currently configured on the student-node.

NOTE: It's recommended to wait for a few minutes to allow deployed objects to become fully operational and start consuming resources.

#####################################################
Solve this question on: ssh cluster4-controlplane


Create a PriorityClass named high-priority with a value of 1000000. A deployment named hp-webapp is in the namespace high-priority. Modify the deployment to use the priority class you created.

Solution--

SSH into the cluster4-controlplane host
ssh cluster4-controlplane

Create a PriorityClass named high-priority with a value of 1000000 by applying the manifest provided below:
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class is for high-priority applications."

Edit the deployment hp-webapp in the high-priority namespace to utilize the high-priority PriorityClass by using the command below:
kubectl edit deployment hp-webapp -n high-priority

In the editor, navigate to the spec.template.spec section and modify the following field:

priorityClassName: high-priority

Verify that the hp-webapp deployment is using the correct PriorityClass by running:
kubectl get deployment hp-webapp -n high-priority -o yaml | grep priorityClassName

##########################################################
Solve this question on: ssh cluster1-controlplane


One application, webpage-server-01, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the /root/new-version directory on the cluster1-controlplane. Validate the chart before installing it on the Kubernetes cluster.


Use the helm command to validate and install the chart. After successfully installing the newer version, uninstall the older version.

Solution--
SSH into the cluster1-controlplane host
ssh cluster1-controlplane



In this task, we will use the helm commands. Here are the steps: -



Use the helm ls command to list the release deployed on the default namespace using helm.
helm ls -n default



First, validate the helm chart by using the helm lint command: -
cd /root/

helm lint ./new-version



Now, install the new version of the application by using the helm install command as follows: -
helm install --generate-name ./new-version



We haven't got any release name in the task, so we can generate the random name from the --generate-name option.


Finally, uninstall the old version of the application by using the helm uninstall command: -

helm uninstall webpage-server-01 -n default

##################################################
Solve this question on: ssh cluster3-controlplane


Your cluster has a failed deployment named backend-api with multiple pods. Troubleshoot the deployment so that all pods are in a running state. Do not make adjustments to the resource limits defined on the deployment pods.

Solution--

SSH into the cluster3-controlplane host
ssh cluster3-controlplane

1. Check Deployment Status
Verify the current status of the deployment:

kubectl get deploy

Expected output (shows that only 2 out of 3 pods are running):

NAME          READY   UP-TO-DATE   AVAILABLE   AGE
backend-api   2/3     2            2           3m48s

The third pod is not getting scheduled.

2. Find the Root Cause
Describe the ReplicaSet to check why the third pod is not being created:

kubectl describe replicasets backend-api-7977bfdbd5

The issue is due to ResourceQuota limitations.
The namespace has a memory limit of 300Mi, and the deployment is exceeding it.
requests.memory=128Mi, used: requests.memory=256Mi, limited: requests.memory=300Mi
Warning  FailedCreate  Error creating: pods "backend-api-7977bfdbd5-hpcjw" is forbidden: exceeded quota: cpu-mem-quota, 
requested: requests.memory=128Mi, used: requests.memory=256Mi, limited: requests.memory=300Mi

3, Modify the Deployment to Fit Within the ResourceQuota
Edit the deployment and reduce memory requests:

kubectl edit deployment backend-api -n default

Modify the resources section:

resources:
  requests:
    cpu: "50m"   # Reduced from 100m to 50m
    memory: "90Mi"   # Reduced from 128Mi to 90Mi (Fits within quota)
  limits:
    cpu: "150m"   
    memory: "150Mi"

4. Verify the Pods (Still Only 2 Running)
Check if the new pod is scheduled:

kubectl get pods -n default

If the third pod is still missing, the old ReplicaSet might be preventing it.

5. Delete the Old ReplicaSet to Allow New Pods to Start
Find the old ReplicaSet:

kubectl get rs -n default

Delete the outdated ReplicaSet:

kubectl delete rs backend-api-7977bfdbd5 -n default

Wait for the new pods to come up.

6. Confirm All Pods Are Running
kubectl get pods -n default

##############################################################
Solve this question on: ssh cluster1-controlplane


Deploy a Vertical Pod Autoscaler (VPA) named analytics-vpa for a deployment named analytics-deployment in the cka24456 namespace. The VPA should automatically adjust the CPU and memory requests of the pods to optimize resource utilization. Ensure that the VPA operates in Auto mode, allowing it to evict and recreate pods with updated resource requests as needed.


#############################################################
Solve this question on: ssh cluster4-controlplane


A pod called pink-pod-cka16-trb is created in the default namespace in cluster4. This app runs on port tcp/5000, and it is to be exposed to end-users using an ingress resource called pink-ing-cka16-trb such that it becomes accessible using the command curl http://kodekloud-pink.app on the cluster4-controlplane host. There is an ingress.yaml file under the root folder in cluster4-controlplane. Create an ingress resource by following the command and continue with the task.

kubectl create -f ingress.yaml



However, even after creating the ingress resource, it is not working. Troubleshoot and fix this issue, making any necessary changes to the objects.

################################################################
Solve this question on: ssh cluster1-controlplane


A pod named beta-pod-cka01-arch has been created in the beta-cka01-arch namespace. Inspect the logs and save all logs starting with the string ERROR in file /root/beta-pod-cka01-arch_errors on the cluster1-controlplane.

###################################################################
Solve this question on: ssh cluster1-controlplane


John is setting up a two-tier application stack that is supposed to be accessible using the service curlme-cka01-svcn. To test that the service is accessible, he is using a pod called curlpod-cka01-svcn. However, at the moment, he is unable to get any response from the application.

Troubleshoot and fix this issue so the application stack is accessible.

While you may delete and recreate the service curlme-cka01-svcn, please do not alter it in anyway.

Solution--

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

######################################################################
Solve this question on: ssh cluster3-controlplane


We have deployed a 2-tier web application on the cluster3 nodes in the canara-wl05 namespace. However, at the moment, the web app pod cannot establish a connection with the MySQL pod successfully.


You can check the status of the application from the terminal by running the curl command with the following syntax:

curl http://cluster3-controlplane:NODE-PORT
To make the application work, create a new secret called db-secret-wl05 with the following key values: -

1. DB_Host=mysql-svc-wl05
2. DB_User=root
3. DB_Password=password123



Next, configure the web application pod to load the new environment variables from the newly created secret.


Note: Check the web application again using the curl command, and the status of the application should be success
Solutuion:
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp-pod-wl05
  name: webapp-pod-wl05
  namespace: canara-wl05
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    name: webapp-pod-wl05
    envFrom:
    - secretRef:
        name: db-secret-wl05

#######################################################################

Solve this question on: ssh cluster4-controlplane


Troubleshoot and resolve the issue with the deployment named nginx-frontend in the cka4974 namespace, which is currently failing to run. Note that the application is intended to serve traffic on port 81.

Solution--
SSH into the cluster4-controlplane host
ssh cluster4-controlplane

Let's check the POD status
kubectl get pod -n cka4974
NAME                              READY   STATUS             RESTARTS      AGE
nginx-frontend-64f67d769f-7wfzc   0/1     CrashLoopBackOff   2 (18s ago)   34s

You will see that nginx-frontend pod is crashing or restarting. So let's try to describe the pod.

kubectl logs -f kube-controller-manager-cluster4-controlplane  -n kube-system

You will see some logs as below:

  Warning  Failed     1s (x5 over 86s)  kubelet            Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/var/lib/kubelet/pods/f4d8bbc7-5ac6-472e-a5c8-5b5cacb9ac03/volumes/kubernetes.io~configmap/nginx-conf-vol" to rootfs at "/etc/nginx/conf.d/default.conf": mount /var/lib/kubelet/pods/f4d8bbc7-5ac6-472e-a5c8-5b5cacb9ac03/volumes/kubernetes.io~configmap/nginx-conf-vol:/etc/nginx/conf.d/default.conf (via /proc/self/fd/6), flags: 0x5001: not a directory: unknown
  Warning  BackOff    1s (x8 over 85s)  kubelet            Back-off restarting failed container nginx in pod nginx-frontend-64f67d769f-7wfzc_cka4974(f4d8bbc7-5ac6-472e-a5c8-5b5cacb9ac03)

The volume mount tries to mount a ConfigMap directory directly over a file path without using subPath. This causes NGINX to crash with an invalid mount error.

Correct the deployment
kubectl edit -n cka4974 deployments.apps nginx-frontend

In the volumeMounts section, update it to the following:

volumeMounts:
        - mountPath: /etc/nginx/conf.d/default.conf
          name: nginx-conf-vol
          subPath: default.conf    

Verify
Check the status of the running pod:

kubectl get pods -n cka4974

To test the application, execute:

kubectl exec -it -n cka4974 deployments/nginx-frontend -- curl -I http://localhost:81
HTTP/1.1 200 OK
Server: nginx/1.27.4
Date: Tue, 01 Apr 2025 05:44:45 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 05 Feb 2025 11:06:32 GMT
Connection: keep-alive
ETag: "67a34638-267"
Accept-Ranges: bytes

Details

#######################################################################

Solve this question on: ssh cluster1-controlplane


A deployment called nodeapp-dp-cka08-trb is created in the default namespace on cluster1. This app is using an ingress resource named nodeapp-ing-cka08-trb.


From cluster1-controlplane host, we should be able to access this app using the command curl http://kodekloud-ingress.app. However, it is not working at the moment. Troubleshoot and fix the issue.


Solution--

SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Try to access the app using curl http://kodekloud-ingress.app command. You will see 404 Not Found error.

Look into the ingress to make sure its configued properly.

kubectl get ingress
kubectl edit ingress nodeapp-ing-cka08-trb

Under rules: -> host: change example.com to kodekloud-ingress.app
Under backend: -> service: -> name: Change example-service to nodeapp-svc-cka08-trb
Change port: -> number: from 80 to 3000
You should be able to access the app using curl http://kodekloud-ingress.app command now.

##############################################################
Solve this question on: ssh cluster3-controlplane


Modify the existing web-gateway on cka5673 namespace to handle HTTPS traffic on port 443 for kodekloud.com, using a TLS certificate stored in a secret named kodekloud-tls.

Solution--

SSH into the cluster3-controlplane host
ssh cluster3-controlplane

Check the configuration of the web-gateway.

cluster3-controlplane ~ ➜  kubectl get gateway web-gateway -n cka5673 -o yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
  resourceVersion: "21919"
  uid: a1d0e35d-5126-4000-88ec-f440941eed75
spec:
  gatewayClassName: kodekloud
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: https
    port: 80
    protocol: HTTP

The current configuration of the web-gateway is incorrect as it is listening on port 80 using the HTTP protocol. To update the web-gateway to listen on port 443 with the TLS certificate, use the following manifest:

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
spec:
  gatewayClassName: kodekloud
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: kodekloud.com
      tls:
        certificateRefs:
          - name: kodekloud-tls
		  
#############################################################################
Solve this question on: ssh cluster3-controlplane


Extend the web-route on cka7395 to direct traffic with the path prefix /api to a service named api-service on port 8080, while all other traffic continues to route to web-service.
Solution--
SSH into the cluster3-controlplane host
ssh cluster3-controlplane

Next, retrieve the configuration of the web-route, which is currently routing all traffic to web-service, using the command below:

kubectl get httproute -n cka7395 -o yaml

You should see output similar to the following:

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: cka7395
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80

To configure the HTTP route to direct traffic to api-service on the /api path, use the manifest below:

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: cka7395
spec:
  parentRefs:
  - name: nginx-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-service
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-service
      port: 80
	  
###############################################################

Solve this question on: ssh cluster1-controlplane


There is a persistent volume named apple-pv-cka04-str. Create a persistent volume claim named apple-pvc-cka04-str and request a 40Mi of storage from apple-pv-cka04-str PV.
The access mode should be ReadWriteOnce and storage class should be manual.

Solution--
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create a yaml template as below:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: apple-pvc-cka04-str
spec:
  volumeName: apple-pv-cka04-str
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi

Apply the template:
kubectl apply -f <template-file-name>.yaml

####################################################################

Solve this question on: ssh cluster2-controlplane


As a Kubernetes administrator, you are unable to run any of the kubectl commands on the cluster. Troubleshoot the problem and get the cluster to a functioning state.

SSH into the Control Plane
Run the following command to access cluster2-controlplane:

ssh cluster2-controlplane

1. Check Cluster Status
Run kubectl get nodes to check the cluster status:

cluster2-controlplane ~ ➜  kubectl get nodes
The connection to the server cluster2-controlplane:6443 was refused - did you specify the right host or port?

This indicates that the API server is down.

2. Verify Kubernetes Containers Are Running
Check if all the Kubernetes-related containers are running:

cluster2-controlplane ~ ➜ crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
fbbf1db37defb       ead0a4a53df89       5 hours ago         Running             coredns                   0                   4be392af8ae56       coredns-69f9c977-lsjwc
b7a2fc42b8b83       ead0a4a53df89       5 hours ago         Running             coredns                   0                   8fdd89e10d6d7       coredns-69f9c977-bfsfw
63f27f8d81c29       a0eed15eed449       5 hours ago         Running             etcd                      0                   3e093d91c4361       etcd-cluster2-controlplane
f07cb1acaa26f       0824682bcdc8e       5 hours ago         Exited              kube-controller-manager   0                   fba0d299aab89       kube-controller-manager-cluster2-controlplane
2f36c95ff0b7e       7ace497ddb8e8       5 hours ago         Exited              kube-scheduler            0                   11e422ba2d28b       kube-scheduler-cluster2-controlplane

The kube-apiserver container is missing.

3. Check kubelet Logs
Since kube-apiserver is missing, check if kubelet is running:

cluster2-controlplane ~ ➜ systemctl status kubelet

Unit kubelet.service could not be found.

This confirms that kubelet is missing or not installed.

4. Reinstall kubelet
Since kubelet is missing, we need to install it manually.

Find the Correct Kubelet Version
cluster2-controlplane ~ ➜ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.0", GitCommit:"70d3cc986aa8221cd1dfb1121852688902d3bf53", GitTreeState:"clean", BuildDate:"2024-12-11T18:04:20Z", GoVersion:"go1.23.3", Compiler:"gc", Platform:"linux/amd64"}

The kubelet version must match kubeadm.

Install kubelet (Matching Version)
sudo apt install -y kubelet=1.32.0-1.1

Start the kubelet Service**
sudo systemctl start kubelet

5. Verify That the Cluster is Restored
Wait a few moments for kubelet to bring back kube-apiserver, then check the cluster status:

cluster2-controlplane ~ ➜ kubectl get nodes

NAME                    STATUS   ROLES           AGE   VERSION
cluster2-controlplane   Ready    control-plane   27m   v1.32.0
cluster2-node01         Ready    <none>          26m   v1.32.0

Details


