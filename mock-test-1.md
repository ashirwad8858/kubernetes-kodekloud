1. Solve this question on: ssh cluster1-controlplane
Create a storage class with the name banana-sc-cka08-str as per the properties given below:
- Provisioner should be kubernetes.io/no-provisioner.
- Volume binding mode should be WaitForFirstConsumer.
- Volume expansion should be enabled.

Solution:
ssh cluster1-controlplane
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-cka08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer

kubectl apply -f <template-file-name>.yaml
--------------------------------------------
2: Solve this question on: ssh cluster1-controlplane

The purple-app-cka27-trb pod is an nginx based app on the container port 80. This app is exposed within the cluster using a ClusterIP type service called purple-svc-cka27-trb.

There is another pod called purple-curl-cka27-trb which continuously monitors the status of the app running within purple-app-cka27-trb pod by accessing the purple-svc-cka27-trb service using curl.

Recently, we started seeing some errors in the logs of the purple-curl-cka27-trb pod.

Dig into the logs to identify the issue and resolve it.

Solution:
ssh cluster1-controlplane
kubectl logs purple-curl-cka27-trb
You will see some logs as below

Not able to connect to the nginx app on http://purple-svc-cka27-trb

Now to debug let's try to access this app from within the purple-app-cka27-trb pod

kubectl exec -it purple-app-cka27-trb -- bash
curl http://purple-svc-cka27-trb
exit

You will notice its stuck, so app is not reachable. Let's look into the service to see its configured correctly.

kubectl edit svc purple-svc-cka27-trb

Under ports: -> port: and targetPort: is set to 8080 but nginx default port is 80 so change 8080 to 80 and save the changes
Let's check the logs now

kubectl logs purple-curl-cka27-trb

You will see Thank you for using nginx. in the output now.
------------------------------------------------------------

Solve this question on: ssh cluster1-controlplane


It appears that the black-cka25-trb deployment in cluster1 isn't up to date. While listing the deployments, we are currently seeing 0 under the UP-TO-DATE section for this deployment. Troubleshoot, fix, and make sure that this deployment is up to date.

Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Check current status of the deployment

kubectl get deploy 

Let's check deployment status

kubectl get deploy black-cka25-trb -o yaml

Under status: you will see message: Deployment is paused so seems like deployment was paused, let check the rollout status

kubectl rollout status deployment black-cka25-trb

You will see this message

Waiting for deployment "black-cka25-trb" rollout to finish: 0 out of 1 new replicas have been updated...

So, let's resume

kubectl rollout resume deployment black-cka25-trb

Check again the status of the deployment

kubectl get deploy 

It should be good now.
--------------------------------------------
Solve this question on: ssh cluster3-controlplane
There is a deployment nginx-deployment-cka04-svcn in cluster3 which is exposed using service nginx-service-cka04-svcn.
Create an ingress resource nginx-ingress-cka04-svcn to load balance the incoming traffic with the following specifications:
pathType: Prefix and path: /

Backend Service Name: nginx-service-cka04-svcn

Backend Service Port: 80

ssl-redirect is set to false
-------------------------------------------------------------
Solve this question on: ssh cluster1-controlplane
Create a new deployment called ocean-tv-wl09 in the default namespace using the image kodekloud/webapp-color:v1.
Use the following specs for the deployment:

1. Replica count should be 3.
2. Set the Max Unavailable to 40% and Max Surge to 55%.
3. Create the deployment and ensure all the pods are ready.
4. After successful deployment, upgrade the deployment image to kodekloud/webapp-color:v2 and inspect the deployment rollout status.
5. Check the rolling history of the deployment, and on the cluster1-controlplane, save the current revision count number to the /opt/revision-count.txt file.
6. Finally, perform a rollback and revert the deployment image to the older version.
_-------------------------------------------------------
Solve this question on: ssh cluster1-controlplane
The db-deployment-cka05-trb deployment is having 0 out of 1 PODs ready.
Figure out the issues and fix them, but make sure to not remove any DB-related environment variables from the deployment/pod.

Solution:
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Check the DB POD logs:
kubectl logs <pod-name>

You might see something like as below which is not that helpful:

Error from server (BadRequest): container "db" in pod "db-deployment-cka05-trb-7457c469b7-zbvx6" is waiting to start: CreateContainerConfigError

So let's look into the kubernetes events for this pod:

kubectl get event --field-selector involvedObject.name=<pod-name>

You will see some errors as below:

Error: couldn't find key db in Secret default/db-cka05-trb

Now let's look into all secrets:

kubectl get secrets db-root-pass-cka05-trb -o yaml
kubectl get secrets db-user-pass-cka05-trb -o yaml
kubectl get secrets db-cka05-trb -o yaml

Now let's look into the deployment.

Edit the deployment
kubectl edit deployment db-deployment-cka05-trb -o yaml

You will notice that some of the keys are different what are reffered in the deployment.

Change some env keys: db to database , db-user to username and db-password to password
Change a secret reference: db-user-cka05-trb to db-user-pass-cka05-trb
Finally save the changes.

-------------------------------------------------
Solve this question on: ssh cluster1-controlplane
In the dev-wl07 namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.
To get back the working state, rollback the application to the previous version .
After rolling the deployment back, on the cluster1-controlplane node, save the image currently in use to the /root/rolling-back-record.txt file and increase the replica count to 5.
Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane
Check the status of the pod: -

kubectl get pods -n dev-wl07

One of the pods is in an error state. As a quick fix, we need to rollback to the previous revision as follows: -

kubectl rollout undo -n dev-wl07 deploy webapp-wl07

After successful rolling back, inspect the updated image: -

kubectl describe deploy -n dev-wl07 webapp-wl07 | grep -i image

On the cluster1-controlplane node, save the image name to the given path /root/rolling-back-record.txt: -

echo "kodekloud/webapp-color" > /root/rolling-back-record.txt

And increase the replica count to the 5 with help of kubectl scale command: -

kubectl scale deploy -n dev-wl07 webapp-wl07 --replicas=5

Verify it by running the command: kubectl get deploy -n dev-wl07

---------------------------------------------------------
Solve this question on: ssh cluster4-controlplane


We tried to schedule the grey-cka21-trb pod on cluster4-controlplane, which was supposed to be deployed by the kubernetes scheduler so far, but somehow it is stuck in a Pending state. Look into the issue and fix it. Make sure the pod is in the Running state.

SSH into the cluster4-controlplane host
ssh cluster4-controlplane

Follow below given steps
Let's check the POD status
kubectl get pod 

You will see that grey-cka21-trb pod is stuck in Pending state. So let's try to look into the logs and events

kubectl logs grey-cka21-trb 
kubectl get event --field-selector involvedObject.name=grey-cka21-trb

You might not find any relevant info in the logs/events. Let's check the status of the kube-scheduler pod

kubectl get pod  -n kube-system

You will notice that kube-scheduler-cluster4-controlplane pod us crashing, let's look into its logs

kubectl logs kube-scheduler-cluster4-controlplane  -n kube-system

You will see an error as below:

run.go:74] "command failed" err="failed to get delegated authentication kubeconfig: failed to get delegated authentication kubeconfig: stat /etc/kubernetes/scheduler.config: no such file or directory"

From the logs we can see that its looking for a file called /etc/kubernetes/scheduler.config which seems incorrect, let's look into the kube-scheduler manifest on cluster4.

First let's find out if /etc/kubernetes/scheduler.config

ls /etc/kubernetes/scheduler.config

You won't find it, instead the correct file is /etc/kubernetes/scheduler.conf so let's modify the manifest.

vi /etc/kubernetes/manifests/kube-scheduler.yaml 

Search for config in the file, you will find some typos, change every occurence of /etc/kubernetes/scheduler.config to /etc/kubernetes/scheduler.conf.

Let's see if kube-scheduler-cluster4-controlplane is running now

kubectl get pod -A

It should be good now and grey-cka21-trb should be good as well.

--------------------------------------------
Solve this question on: ssh cluster1-controlplane
Create an nginx pod called nginx-resolver-cka06-svcn using the image nginx, and expose it internally with a service called nginx-resolver-service-cka06-svcn.

Test that you are able to look up the service and pod names from within the cluster. Use the image busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc.cka06.svcn and /root/CKA/nginx.pod.cka06.svcn on cluster1-controlplane.

Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane
To create a pod nginx-resolver-cka06-svcn and expose it internally:

cluster1-controlplane ~ ➜  kubectl run nginx-resolver-cka06-svcn --image=nginx 
cluster1-controlplane ~ ➜  kubectl expose pod/nginx-resolver-cka06-svcn --name=nginx-resolver-service-cka06-svcn --port=80 --target-port=80 --type=ClusterIP 

To create a pod test-nslookup. Test that you are able to look up the service and pod names from within the cluster:

cluster1-controlplane ~ ➜   kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service-cka06-svcn
cluster1-controlplane ~ ➜   kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service-cka06-svcn > /root/CKA/nginx.svc.cka06.svcn

Get the IP of the nginx-resolver-cka06-svcn pod and replace the dots(.) with hyphon(-) which will be used below.
cluster1-controlplane ~ ➜   kubectl get pod nginx-resolver-cka06-svcn -o wide
cluster1-controlplane ~ ➜   IP=`kubectl get pod nginx-resolver-cka06-svcn -o wide --no-headers | awk '{print $6}' | tr '.' '-'`
cluster1-controlplane ~ ➜   kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup $IP.default.pod > /root/CKA/nginx.pod.cka06.svcn

------------------------------------------------------------------
Solve this question on: ssh cluster1-controlplane
Create a service account called deploy-cka20-arch. Further, create a cluster role called deploy-role-cka20-arch with permissions to get the deployments in cluster1.
Finally, create a cluster role binding called deploy-role-binding-cka20-arch to bind deploy-role-cka20-arch cluster role with the deploy-cka20-arch service account.
Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane

Create the service account, cluster role and role binding:

cluster1-controlplane ~ ➜  kubectl create serviceaccount deploy-cka20-arch
cluster1-controlplane ~ ➜  kubectl create clusterrole deploy-role-cka20-arch --resource=deployments --verb=get
cluster1-controlplane ~ ➜  kubectl create clusterrolebinding deploy-role-binding-cka20-arch --clusterrole=deploy-role-cka20-arch --serviceaccount=default:deploy-cka20-arch

You can verify it as below:

cluster1-controlplane ~ ➜  kubectl auth can-i get deployments --as=system:serviceaccount:default:deploy-cka20-arch
yes

-------------------------------------------------------------
Solve this question on: ssh cluster2-controlplane


Create an HTTPRoute named web-route in the nginx-gateway namespace that directs traffic from the web-gateway to a backend service named web-service on port 80 and ensures that the route is applied only to requests with the hostname cluster2-controlplane.

To test the configuration, run the following command:

curl http://cluster2-controlplane:30080

Solution
SSH into the cluster2-controlplane host
ssh cluster2-controlplane

To expose the web-service via the web-gateway, deploy an HTTPRoute resource with the following configuration:

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: nginx-gateway
spec:
  hostnames:
  - cluster2-controlplane
  parentRefs:
  - name: web-gateway
  rules:
  - backendRefs:
    - name: web-service
      port: 80
	  
---------------------------------------------------
Solve this question on: ssh cluster2-controlplane


On the cluster2-controlplane, a Helm chart repository is given under the /opt/ path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

The release name should be webapp-color-apd.
All the resources should be deployed on the frontend-apd namespace.
The service type should be node port.
Scale the deployment to 3.
Application version should be 1.20.0.
NOTE: - Remember to make necessary changes in the values.yaml and Chart.yaml files according to the specifications, and, to fix the issues, inspect the template files.

Solution
SSH into the cluster2-controlplane host
ssh cluster2-controlplane

In this task, we will use the helm commands. Here are the steps:

First, check the given namespace; if it doesn't exist, we must create it first; otherwise, it will give an error "namespaces not found" while installing the helm chart.
To check all the namespaces in the cluster2, we would have to run the following command: -

kubectl get ns

It will list all the namespaces. If the given namespace doesn't exist, then run the following command: -

kubectl create ns frontend-apd

Now, on the student-node node and go to the /opt/ directory. We have given the helm chart directory webapp-color-apd that contains templates, values files, and the chart file etc.

Update the values according to the given specifications as follows: -

a.) Update the value of the appVersion to 1.20.0 in the Chart.yaml file.

b.) Update the value of the replicaCount to 3 in the values.yaml file.

c.) Update the value of the type to NodePort in the values.yaml file.

These are the values we have to update.

Now, we will use the helm lint command to check the Helm chart because it can identify errors such as missing or misconfigured values, invalid YAML syntax, and deprecated APIs etc.

cd /opt/

helm lint /opt/webapp-color-apd/



If there is no misconfiguration, we will see the similar output: -

helm lint /opt/webapp-color-apd/
==> Linting /opt/webapp-color-apd/
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed

But in our case, there are some issues with the given templates.

Deployment apiVersion needs to be correctly written. It should be apiVersion: apps/v1.

In the service YAML, there is a typo in the template variable {{ .Values.service.name }} because of that, it's not able to reference the value of the name field defined in the values.yaml file for the Kubernetes service that is being created or updated.


Now run the following command to install the helm chart in the frontend-apd namespace: -

helm install webapp-color-apd -n frontend-apd /opt/webapp-color-apd

Use the helm ls command to list the release deployed using helm.

helm ls -n frontend-apd



-------------------------------------------
Solve this question on: ssh cluster1-controlplane

A persistent volume called papaya-pv-cka09-str is already created with a storage capacity of 150Mi. It's using the papaya-stc-cka09-str storage class with the path /opt/papaya-stc-cka09-str.


A persistent volume claim named papaya-pvc-cka09-str has also been created on this cluster. This PVC has requested 50Mi of storage from papaya-pv-cka09-str volume.


Resize the PVC to 80Mi and make sure the PVC is in the Bound state.

--------------------------------------------------
Solve this question on: ssh cluster4-controlplane


The pink-depl-cka14-trb Deployment was scaled to 2 replicas; however, the current replica count is still 1.


Troubleshoot and fix this issue. Make sure the CURRENT count is equal to the DESIRED count.


You can SSH into the cluster4 using ssh cluster4-controlplane command.
----------------------------------------------------
Solve this question on: ssh cluster1-controlplane


We have created a service account called green-sa-cka22-arch, a cluster role called green-role-cka22-arch, and a cluster role binding called green-role-binding-cka22-arch.


Update the permissions of this service account so that it can only get all the namespaces in cluster1.

Solution
SSH into the cluster1-controlplane host
ssh cluster1-controlplane



Edit the green-role-cka22-arch to update permissions:

cluster1-controlplane ~ ➜  kubectl edit clusterrole green-role-cka22-arch 




At the end add below code:

- apiGroups:
  - "*"
  resources:
  - namespaces
  verbs:
  - get




You can verify it as below:

cluster1-controlplane ~ ➜  kubectl auth can-i get namespaces --as=system:serviceaccount:default:green-sa-cka22-arch
yes

------------------------------------------
16: Solve this question on: ssh cluster1-controlplane
Create an HPA for a deployment named frontend-deployment in the default namespace. The HPA should scale the deployment based on CPU utilization, maintaining an average CPU usage of 70% across all pods. Set the minimum number of replicas to 2 and the maximum to 10
