1. Find the node across all clusters that consumes the most CPU and store the result to the file /opt/high_cpu_node in the following format cluster_name,node_name.

The node could be in any clusters that are currently configured on the student-node.

NOTE: It's recommended to wait for a few minutes to allow deployed objects to become fully operational and start consuming resources.

ANS:
Check out the metrics for all node across all clusters:

student-node ~ ➜ kubectl top node --context cluster1 --no-headers | sort -nr -k2 | head -1
cluster1-controlplane 127m 1% 703Mi 1%

student-node ~ ➜ kubectl top node --context cluster2 --no-headers | sort -nr -k2 | head -1
cluster2-controlplane 126m 1% 675Mi 1%

student-node ~ ➜ kubectl top node --context cluster3 --no-headers | sort -nr -k2 | head -1
cluster3-controlplane 577m 7% 1081Mi 1%

student-node ~ ➜ kubectl top node --context cluster4 --no-headers | sort -nr -k2 | head -1
cluster4-controlplane 130m 1% 679Mi 1%

Using this, find the node that uses most cpu. In this case, it is cluster3-controlplane on cluster3.

Save the result in the correct format to the file:

student-node ~ ➜ echo cluster3,cluster3-controlplane > /opt/high_cpu_node

2. Before proceeding, ensure you are operating within the correct Kubernetes context by switching to cluster1 with the following command:

kubectl config use-context cluster1

We have set up a service account named blue-sa-cka21-arch, along with a cluster role blue-role-cka21-arch and a corresponding cluster role binding blue-role-binding-cka21-arch.

You need to adjust the permissions for the service account so it no longer has cluster-wide access. Update the service account to ensure it can only has get access for pods within the default namespace of cluster1.

ANS:
Ensure you are working within the correct Kubernetes context:

kubectl config use-context cluster1

Since you're restricting the service account from cluster-wide access to namespace-specific access, you might need to delete the existing ClusterRoleBinding:
kubectl delete clusterrolebinding blue-role-binding-cka21-arch

Create a Role that specifically allows get operations on pods within the default namespace:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: blue-role-cka21-arch
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get"]
```

Save this to a file named role.yaml and apply it with:

kubectl apply -f role.yaml

Bind the service account to the new role using a RoleBinding. This grants the service account the permissions defined in the role, but only within the default namespace:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: blue-role-binding-cka21-arch
  namespace: default
subjects:
- kind: ServiceAccount
  name: blue-sa-cka21-arch
  namespace: default
roleRef:
  kind: Role
  name: blue-role-cka21-arch
  apiGroup: rbac.authorization.k8s.io
```

Save this to a file named rolebinding.yaml and apply it with:

kubectl apply -f rolebinding.yaml

To ensure that the permissions are correctly set, you can use the kubectl auth can-i command to check if the service account can perform the desired action within the default namespace:
kubectl auth can-i get pods --as=system:serviceaccount:default:blue-sa-cka21-arch --namespace default

This command should return yes, confirming that the service account can now get pods in the default namespace.

3. A pod called color-app-cka13-arch has been created in the default namespace. This pod logs can be accessed using kubectl logs -f color-app-cka13-arch command from the student-node. It is currently displaying Color is pink output. Update the pod definition file to make use of the environment variable with the value - green and recreate this pod.

4. The deployment called trace-wl08 inside the test-wl08 namespace on cluster1 has undergone several, routine, rolling updates and rollbacks.

Inspect the revision 2 of this deployment and store the image name that was used in this revision in the /opt/trace-wl08-revision-book.txt file on the student-node.

ANS:
List the deployments in test-wl08 namespace as follows: -

kubectl get deploy -n test-wl08

Run the following command to check the revisions of the deployment called trace-wl08:-

kubectl rollout history deployment -n test-wl08 trace-wl08

Inspect the revision 2 by using the --revision option: -

kubectl rollout history deployment -n test-wl08 trace-wl08 --revision=2

Under the Containers section, you will see the image name.

On the student-node, save that image name in the given file /opt/trace-wl08-revision-book.txt:

echo "busybox:1.35" > /opt/trace-wl08-revision-book.txt

5. We have created a service account called red-sa-cka23-arch, a cluster role called red-role-cka23-arch and a cluster role binding called red-role-binding-cka23-arch.

Identify the permissions of this service account and write down the answer in file /opt/red-sa-cka23-arch in format resource:pods|verbs:get,list on student-node

6. Create a storage class called orange-stc-cka07-str as per the properties given below:

- Provisioner should be kubernetes.io/no-provisioner.

- Volume binding mode should be WaitForFirstConsumer.

Next, create a persistent volume called orange-pv-cka07-str as per the properties given below:

- Capacity should be 150Mi.

- Access mode should be ReadWriteOnce.

- Reclaim policy should be Retain.

- It should use storage class orange-stc-cka07-str.

- Local path should be /opt/orange-data-cka07-str.

- Also add node affinity to create this value on cluster1-controlplane.

Finally, create a persistent volume claim called orange-pvc-cka07-str as per the properties given below:

- Access mode should be ReadWriteOnce.

- It should use storage class orange-stc-cka07-str.

- Storage request should be 128Mi.

- The volume should be orange-pv-cka07-str.

ANS:

```
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
```

7. There is a pod called pink-pod-cka16-trb created in the default namespace in cluster4. This app runs on port tcp/5000 and it is to be exposed to end-users using an ingress resource called pink-ing-cka16-trb in such a way that it is supposed to be accessible using the command: curl http://kodekloud-pink.app on cluster4-controlplane host. There is an ingress.yaml file under root folder in cluster4-controlplane create an ingress resource by following the command and continue with task.

kubectl create -f ingress.yaml

However, even after creating the ingress resource, it is not working. Troubleshoot and fix this issue, making any necessary to the objects.

Note: You should be able to ssh into the cluster4-controlplane using ssh cluster4-controlplane command.

ANS:
SSH into the cluster4-controlplane host
ssh cluster4-controlplane

create ingress with the given yaml file.ingress.yaml

kubectl create -f ingress.yaml

Now try to access the app.

curl kodekloud-pink.app

You must be getting 503 Service Temporarily Unavailable error.
Let's look into the service:

kubectl edit svc pink-svc-cka16-trb

Under ports: change protocol: UDP to protocol: TCP
Try to access the app again

curl kodekloud-pink.app

You must be getting curl: (6) Could not resolve host: example.com error, from the error we can see that its not able to resolve example.com host which indicated that it can be some issue related to the DNS. As we know CoreDNS is a DNS server that can serve as the Kubernetes cluster DNS, so it can be something related to CoreDNS.

Let's check if we have CoreDNS deployment running:

kubectl get deploy -n kube-system

You will see that for coredns all relicas are down, you will see 0/0 ready pods. So let's scale up this deployment.

kubectl scale --replicas=2 deployment coredns -n kube-system

Once CoreDNS is up let's try to access to app again.

curl kodekloud-pink.app

It should work now.

8. A pod named beta-pod-cka01-arch has been created in the beta-cka01-arch namespace. Inspect the logs and save all logs starting with the string ERROR in file /root/beta-pod-cka01-arch_errors on the student-node.

9. John is setting up a two tier application stack that is supposed to be accessible using the service curlme-cka01-svcn. To test that the service is accessible, he is using a pod called curlpod-cka01-svcn. However, at the moment, he is unable to get any response from the application.

Troubleshoot and fix this issue so the application stack is accessible.

While you may delete and recreate the service curlme-cka01-svcn, please do not alter it in anyway.

10. We have deployed a 2-tier web application on the cluster3 nodes in the canara-wl05 namespace. However, at the moment, the web app pod cannot establish a connection with the MySQL pod successfully.

You can check the status of the application from the terminal by running the curl command with the following syntax:

curl http://cluster3-controlplane:NODE-PORT

To make the application work, create a new secret called db-secret-wl05 with the following key values: -

1. DB_Host=mysql-svc-wl05
2. DB_User=root
3. DB_Password=password123

Next, configure the web application pod to load the new environment variables from the newly created secret.

Note: Check the web application again using the curl command, and the status of the application should be success.

You can SSH into the cluster3 using ssh cluster3-controlplane command.

ANS:
Set the correct context: -

kubectl config use-context cluster3

List the nodes: -

kubectl get nodes -o wide

Run the curl command to know the status of the application as follows: -

ssh cluster3-controlplane

curl http://10.17.63.11:31020

<!doctype html>
<title>Hello from Flask</title>
...

    <img src="/static/img/failed.png">
    <h3> Failed connecting to the MySQL database. </h3>


    <h2> Environment Variables: DB_Host=Not Set; DB_Database=Not Set; DB_User=Not Set; DB_Password=Not Set; 2003: Can&#39;t connect to MySQL server on &#39;localhost:3306&#39; (111 Connection refused) </h2>

As you can see, the status of the application pod is failed.

NOTE: - In your lab, IP addresses could be different.

Let's create a new secret called db-secret-wl05 as follows: -

kubectl create secret generic db-secret-wl05 -n canara-wl05 --from-literal=DB_Host=mysql-svc-wl05 --from-literal=DB_User=root --from-literal=DB_Password=password123

After that, configure the newly created secret to the web application pod as follows: -

```
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
```

then use the kubectl replace command: -

kubectl replace -f <FILE-NAME> --force

In the end, make use of the curl command to check the status of the application pod. The status of the application should be success.

curl http://10.17.63.11:31020

<!doctype html>
<title>Hello from Flask</title>
<body style="background: #39b54b;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

    <img src="/static/img/success.jpg">
    <h3> Successfully connected to the MySQL database.</h3>

11. On cluster4 we are having some weird issue where we are intermittently getting below error while running kubectl commands.

The connection to the server cluster4-controlplane:6443 was refused - did you specify the right host or port?

Whenever you get this error, you can wait for 10-15 seconds to make kubectl command work again, but it will come again after few second

We also noticed that kube-controller-manager-cluster4-controlplane pod is restarting continuously. Look into the issue and troubleshoot the same.

Note: After updating system pods might take a little time to come in running state so wait for few min after making changes.

You can SSH into the cluster4 using ssh cluster4-controlplane command.
SSH Password for host cluster4-controlplane is C0ntr0lplan3Pa$$wd

ANS:
Let's check the POD status
kubectl get pod --context=cluster4 -n kube-system

You will see that kube-controller-manager-cluster4-controlplane pod is crashing or restarting. So let's try to watch the logs.

kubectl logs -f kube-controller-manager-cluster4-controlplane --context=cluster4 -n kube-system

You will see some logs as below:

leaderelection.go:330] error retrieving resource lock kube-system/kube-controller-manager: Get "https://10.10.129.21:6443/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=5s": dial tcp 10.10.129.21:6443: connect: connection refused

You will notice that somehow the connection to the kube api is breaking, let's check if kube api pod is healthy.

kubectl get pod --context=cluster4 -n kube-system

Now you might notice that kube-apiserver-cluster4-controlplane pod is also restarting, so we should dig into its logs or relevant events.

kubectl logs -f kube-apiserver-cluster4-controlplane -n kube-system

kubectl get event --field-selector involvedObject.name=kube-apiserver-cluster4-controlplane -n kube-system

In events you will see this error

Warning Unhealthy pod/kube-apiserver-cluster4-controlplane Liveness probe failed: Get "https://10.10.132.25:6444/livez": dial tcp 10.10.132.25:6444: connect: connection refused

From this we can see that the Liveness probe is failing for the kube-apiserver-cluster4-controlplane pod, and we can see its trying to connect to port 6444 port but the default api port is 6443. So let's look into the kube api server manifest.

ssh cluster4-controlplane
vi /etc/kubernetes/manifests/kube-apiserver.yaml

Under livenessProbe: you will see the port: value is 6444, change it to 6443 and save. Now wait for few seconds let the kube api pod come up.

kubectl get pod -n kube-system

Watch the PODs status for some time and make sure these are not restarting now.

12. There is a deployment called nodeapp-dp-cka08-trb created in the default namespace on cluster1. This app is using an ingress resource named nodeapp-ing-cka08-trb.

From cluster1-controlplane host we should be able to access this app using the command: curl http://kodekloud-ingress.app. However, it is not working at the moment. Troubleshoot and fix the issue.

Note: You should be able to ssh into the cluster1-controlplane using ssh cluster1-controlplane command.

ANS:
SSh into cluster1-controlplane
ssh cluster1-controlplane

Try to access the app using curl http://kodekloud-ingress.app command. You will see 404 Not Found error.

Look into the ingress to make sure its configued properly.

kubectl get ingress
kubectl edit ingress nodeapp-ing-cka08-trb

Under rules: -> host: change example.com to kodekloud-ingress.app
Under backend: -> service: -> name: Change example-service to nodeapp-svc-cka08-trb
Change port: -> number: from 80 to 3000
You should be able to access the app using curl http://kodekloud-ingress.app command now.

13. Create a loadbalancer service with name wear-service-cka09-svcn to expose the deployment webapp-wear-cka09-svcn application in app-space namespace.

14. Create a persistent volume called red-pv-cka03-str of type: hostPath type, use the path /opt/red-pv-cka03-str and capacity: 100Mi.

ANS:
Set context to cluster1.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: red-pv-cka03-str
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/red-pv-cka03-str
```

Apply the template:
kubectl apply -f <template-file-name>.yaml

15. Deploy a messaging-cka07-svcn pod using the redis:alpine image with the labels set to tier=msg.

Now create a service messaging-service-cka07-svcn to expose the messaging-cka07-svcn application within the cluster on port 6379.

TIP: Use imperative commands.

ANS:
On student-node, use the command kubectl run messaging-cka07-svcn --image=redis:alpine -l tier=msg

Now run the command: kubectl expose pod messaging-cka07-svcn --port=6379 --name messaging-service-cka07-svcn

16. The controlplane node called cluster4-controlplane in the cluster4 cluster is planned for a regular maintenance. In preparation for this maintenance work, we need to take backups of this cluster. However, something is broken at the moment!

Troubleshoot the issues and take a snapshot of the ETCD database using the etcdctl utility at the location /opt/etcd-boot-cka18-trb.db.

Note: Make sure etcd listens at its default port. Also you can SSH to the cluster4-controlplane host using the ssh cluster4-controlplane command from the student-node.

ANS:
SSH into cluster4-controlplane host.
ssh cluster4-controlplane

Let's take etcd backup

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-boot-cka18-trb.db

It might stuck for forever, let's see why that would happen. Try to list the PODs first

kubectl get pod -A

There might an error like

The connection to the server cluster4-controlplane:6443 was refused - did you specify the right host or port?

There seems to be some issue with the cluster so let's look into the logs

journalctl -u kubelet -f

You will see a lot of connect: connection refused erros but that must be because the different cluster components are not able to connect to the api server so try to filter out these logs to look more closly

journalctl -u kubelet -f | grep -v 'connect: connection refused'

You should see some erros as below

cluster4-controlplane kubelet[2240]: E0923 04:38:15.630925 2240 file.go:187] "Could not process manifest file" err="invalid pod: [spec.containers[0].volumeMounts[1].name: Not found: \"etcd-cert\"]" path="/etc/kubernetes/manifests/etcd.yaml"

So seems like there is some incorrect volume which etcd is trying to mount, let's look into the etcd manifest.

vi /etc/kubernetes/manifests/etcd.yaml

Search for etcd-cert, you will notice that the volume name is etcd-certs but the volume mount is trying to mount etcd-cert volume which is incorrect. Fix the volume mount name and save the changes. Let's restart kubelet service after that.

systemctl restart kubelet

Wait for few minutes to see if its good now.

kubectl get pod -A

You should be able to list the PODs now, let's try to take etcd backup now:

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-boot-cka18-trb.db

It should work now.

17. There is a persistent volume named apple-pv-cka04-str. Create a persistent volume claim named apple-pvc-cka04-str and request a 40Mi of storage from apple-pv-cka04-str PV.
    The access mode should be ReadWriteOnce and storage class should be manual.

ANS:
Set context to cluster1:

```
Create a yaml template as below:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: apple-pvc-cka04-str
spec:
  volumeName: apple-pv-cka04-str
  storageClassName: manual
  accessModes: - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi
```

Apply the template:
kubectl apply -f <template-file-name>.yaml
