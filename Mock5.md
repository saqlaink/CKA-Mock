1. A pod called nginx-cka01-trb is running in the default namespace. There is a container called nginx-container running inside this pod that uses the image nginx:latest. There is another sidecar container called logs-container that runs in this pod.

For some reason, this pod is continuously crashing. Identify the issue and fix it. Make sure that the pod is in a running state and you are able to access the website using the curl http://kodekloud-exam.app:30001 command on the controlplane node of cluster1.

ANS:
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

```
cat: can't open '/var/log/httpd/access.log': No such file or directory
cat: can't open '/var/log/httpd/error.log': No such file or directory
```

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

curl http://kodekloud-exam.app:30001

You will see error

curl: (7) Failed to connect to kodekloud-exam.app port 30001: Connection refused

So you are not able to access the website, et's look into the service configuration.

Edit the service
kubectl edit svc nginx-service-cka01-trb -o yaml

Change app label under selector from httpd-app-cka01-trb to nginx-app-cka01-trb
You should be able to access the website now.
curl http://kodekloud-exam.app:30001

2. We have deployed a simple web application called frontend-wl04 on cluster1. This version of the application has some issues from a security point of view and needs to be updated to version 2.

Update the image and wait for the application to fully deploy.

You can verify the running application using the curl command on the terminal:

```
student-node ~ ➜  curl http://cluster1-node01:30080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from frontend-wl04-84fc69bd96-p7rbl!</h1>

  <h2>
    Application Version: v1
  </h2>

</div>
student-node ~ ➜
```

Version 2 Image details as follows:

1. Current version of the image is `v1`, we need to update with the image to kodekloud/webapp-color:v2.
2. Use the imperative command to update the image.

3. The cat-cka22-trb pod is stuck in Pending state. Look into the issue to fix the same. Make sure that the pod is in running state and its stable (i.e not restarting or crashing).

Note: Do not make any changes to the pod (No changes to pod config but you may destory and re-create).

ANS:
Let's check the POD status
kubectl get pod

You will see that cat-cka22-trb pod is stuck in Pending state. So let's try to look into the events

kubectl --context cluster2 get event --field-selector involvedObject.name=cat-cka22-trb

You will see some logs as below

```
Warning   FailedScheduling   pod/cat-cka22-trb   0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/master: }, 2 node(s) didn't match Pod's node affinity/selector. preemption: 0/2 nodes are available: 3 Preemption is not helpful for scheduling.
```

So seems like this POD is using the node affinity, let's look into the POD to understand the node affinity its using.

kubectl --context cluster2 get pod cat-cka22-trb -o yaml

Under affinity: you will see its looking for key: node and values: cluster2-node02 so let's verify if node01 has these labels applied.

kubectl --context cluster2 get node cluster2-node01 -o yaml

Look under labels: and you will not find any such label, so let's add this label to this node.

kubectl label node cluster1-node01 node=cluster2-node01

Check again the node details

kubectl get node cluster2-node01 -o yaml

The new label should be there, let's see if POD is scheduled now on this node

kubectl --context cluster2 get pod

Its is but it must be crashing or restarting, so let's look into the pod logs

kubectl --context cluster2 logs -f cat-cka22-trb

You will see logs as below:

The HOST variable seems incorrect, it must be set to kodekloud

Let's look into the POD env variables to see if there is any HOST env variable

kubectl --context cluster2 get pod -o yaml

Under env: you will see this

```
env:
- name: HOST
  valueFrom:
    secretKeyRef:
      key: hostname
      name: cat-cka22-trb
```

So we can see that HOST variable is defined and its value is being retrieved from a secret called "cat-cka22-trb". Let's look into this secret.

kubectl --context cluster2 get secret
kubectl --context cluster2 get secret cat-cka22-trb -o yaml

You will find a key/value pair under data:, let's try to decode it to see its value:

echo "<the decoded value you see for hostname" | base64 -d

ok so the value is set to kodekloude which is incorrect as it should be set to kodekloud. So let's update the secret:

echo "kodekloud" | base64
kubectl edit secret cat-cka22-trb

Change requests storage hostname: a29kZWtsb3Vkdg== to hostname: a29kZWtsb3VkCg== (values may vary)
POD should be good now.

4. Create a service account called pink-sa-cka24-arch. Further create a cluster role called pink-role-cka24-arch with full permissions on all resources in the core api group under default namespace in cluster1.

Finally create a cluster role binding called pink-role-binding-cka24-arch to bind pink-role-cka24-arch cluster role with pink-sa-cka24-arch service account.

5. On cluster3, there is a web application pod running inside the default namespace. This pod which is part of a deployment called webapp-color-wl10 and makes use of an environment variable that can change constantly. Add this environment variable to a configmap and configure the pod in the deployment to make use of this config map.

Use the following specs-

1. Create a new configMap called webapp-wl10-config-map with the key and value as - APP_COLOR=red.

2. Update the deployment to make use of the newly created configMap name.

3. Delete and recreate the deployment if necessary.

ANS:
Set the correct context: -

kubectl config use-context cluster3

Inspect the given pod in the default namespace: -

kubectl get pods -n default

Let's create a new configMap in the default namespace as follows:-

kubectl create configmap webapp-wl10-config-map --from-literal=APP_COLOR=red

And now configure the newly created configmap to the web application pod's template: -

```
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
```

NOTE: - It will terminate the old pods and will recreate the new pods with configMap.

You can inspect the newly created pod to check the configMap: -

kubectl describe po webapp-color-wl10-d79c6f76c-4gjmz

Note: - In your lab, pod name could be different.

6. ETCD Key Value store which is running as a pod in cluster1 . Take the backup of it and store it on the cluster1-controlplane node at the path /opt/cluster1_backup.db.

You can ssh to the controlplane node by running ssh root@cluster1-controlplane from the student-node.

NOTE: - If the etcd utility tool is unavailable on the controlplane, install it first.

ANS:
SSH into cluster1-controlplane node:

student-node ~ ➜ ssh root@cluster1-controlplane

Install etcd utility (if not installed already) and take etcd backup:

```
cluster1-controlplane ~ ➜ cd /tmp
cluster1-controlplane ~ ➜ export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep tag_name | cut -d '"' -f 4)
cluster1-controlplane ~ ➜ wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
cluster1-controlplane ~ ➜ tar xvf etcd-${RELEASE}-linux-amd64.tar.gz ; cd etcd-${RELEASE}-linux-amd64
cluster1-controlplane ~ ➜ mv etcd etcdctl  /usr/local/bin/
cluster1-controlplane ~ ➜ ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/cluster1_backup.db
```

7. Create a generic secret called secure-sec-cka12-arch in the secure-sys-cka12-arch namespace on the cluster3. Use the key/value of color=darkblue to create the secret.

ANS:
Create a namespace called secure-sys-cka12-arch

```
student-node ~ ➜ kubectl create ns secure-sys-cka12-arch --context cluster3
```

Create a generic secret called secure-sec-cka12-arch

```
student-node ~ ➜ kubectl create secret generic secure-sec-cka12-arch --from-literal=color=darkblue -n secure-sys-cka12-arch --context cluster3
```

8. We recently deployed a DaemonSet called logs-cka26-trb under kube-system namespace in cluster2 for collecting logs from all the cluster nodes including the controlplane node. However, at this moment, the DaemonSet is not creating any pod on the controlplane node.

Troubleshoot the issue and fix it to make sure the pods are getting created on all nodes including the controlplane node.

ANS:
Check the status of DaemonSet

kubectl --context2 cluster2 get ds logs-cka26-trb -n kube-system

You will find that DESIRED CURRENT READY etc have value 2 which means there are two pods that have been created. You can check the same by listing the PODs

kubectl --context2 cluster2 get pod -n kube-system

You can check on which nodes these are created on

kubectl --context2 cluster2 get pod <pod-name> -n kube-system -o wide

Under NODE you will find the node name, so we can see that its not scheduled on the controlplane node which is because it must be missing the reqiured tolerations. Let's edit the DaemonSet to fix the tolerations

kubectl --context2 cluster2 edit ds logs-cka26-trb -n kube-system

Under tolerations: add below given tolerations as well

```
- key: node-role.kubernetes.io/control-plane
  operator: Exists
  effect: NoSchedule
```

Wait for some time PODs should schedule on all nodes now including the controlplane node.

9. There is a deployment nginx-deployment-cka04-svcn in cluster3 which is exposed using service nginx-service-cka04-svcn.

Create an ingress resource nginx-ingress-cka04-svcn to load balance the incoming traffic with the following specifications:

pathType: Prefix and path: /

Backend Service Name: nginx-service-cka04-svcn

Backend Service Port: 80

ssl-redirect is set to false

ANS:
First change the context to "cluster3":

student-node ~ ➜ kubectl config use-context cluster3
Switched to context "cluster3".

Now apply the ingress resource with the given requirements:

```
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
```

Check if the ingress resource was successfully created:

student-node ~ ➜ kubectl get ingress
NAME CLASS HOSTS ADDRESS PORTS AGE
nginx-ingress-cka04-svcn <none> \* 172.25.0.10 80 13s

As the ingress controller is exposed on cluster3-controlplane using traefik service, we need to ssh to cluster3-controlplane first to check if the ingress resource works properly:

student-node ~ ➜ ssh cluster3-controlplane

```
cluster3-controlplane:~# curl -I 172.25.0.11
```

HTTP/1.1 200 OK
...

10. A storage class called coconut-stc-cka01-str was created earlier.

Use this storage class to create a persistent volume called coconut-pv-cka01-str as per below requirements:

- Capacity should be 100Mi.

- The volume type should be hostpath and the path should be /opt/coconut-stc-cka01-str.

- Use coconut-stc-cka01-str storage class.

- This volume must be created on cluster1-node01 (the /opt/coconut-stc-cka01-str directory already exists on this node).

- It must have a label with key: storage-tier with value: gold.

Also create a persistent volume claim with the name coconut-pvc-cka01-str as per below specs:

- Request 50Mi of storage from coconut-pv-cka01-str PV, it must use matchLabels to use the PV.

- Use coconut-stc-cka01-str storage class.

- The access mode must be ReadWriteMany.

ANS:
First set the context to cluster1

kubectl config use-context cluster1

Create a yaml template as below:

```
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
```

11. The deployment called web-dp-cka17-trb has 0 out of 1 pods up and running. Troubleshoot this issue and fix it. Make sure all required POD(s) are in running state and stable (not restarting).

The application runs on port 80 inside the container and is exposed on the node port 30090.

ANS:
List out the PODs
kubectl get pod

Let's look into the relevant events:

kubectl get event --field-selector involvedObject.name=<pod-name>

You should see some errors as below:

Warning FailedScheduling pod/web-dp-cka17-trb-9bdd6779-fm95t 0/3 nodes are available: 3 persistentvolumeclaim "web-pvc-cka17-trbl" not found. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

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

```
Warning   Failed      pod/web-dp-cka17-trb-67c9bdcd85-4tvpr   Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/bin/bsh\\": stat /bin/bsh\: no such file or directory: unknown
```

Let's look into the deployment:

kubectl edit deploy web-dp-cka17-trb

Under initContainers: -> - command: change /bin/bsh\ to /bin/bash
let's see the PODs
kubectl get pod

Wait for some time to make sure it is stable, but you will notice that its restart so still something must be wrong.

So let's check the events again.

kubectl get event --field-selector involvedObject.name=<pod-name>

You should see someting like

Warning Unhealthy pod/web-dp-cka17-trb-647f69f8bd-67xmx Liveness probe failed: Get "http://10.50.64.1:81/": dial tcp 10.50.64.1:81: connect: connection refused

Seems like its not able to connect to a service, let's look into the deployment to understand

kubectl edit deploy web-dp-cka17-trb

Notice that containerPort: 80 but under livenessProbe: the port: 81 so seems like livenessProbe is using wrong port. let's change port: 81 to port: 80

See the PODs now

kubectl get pod

It should be good now.

12. Create a loadbalancer service with name wear-service-cka09-svcn to expose the deployment webapp-wear-cka09-svcn application in app-space namespace.

ANS:
On student node run the command:

```
student-node ~ ➜  kubectl expose -n app-space deployment webapp-wear-cka09-svcn --type=LoadBalancer --name=wear-service-cka09-svcn --port=8080
service/wear-service-cka09-svcn exposed
```

student-node ~ ➜ k get svc -n app-space
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
wear-service-cka09-svcn LoadBalancer 10.43.68.233 172.25.0.14 8080:32109/TCP 14s

13. Find the pod that consumes the most CPU and store the result to the file /opt/high_cpu_pod in the following format cluster_name,namespace,pod_name.

The pod could be in any namespace in any of the clusters that are currently configured on the student-node.

NOTE: It's recommended to wait for a few minutes to allow deployed objects to become fully operational and start consuming resources.

ANS:
Check out the metrics for all pods across all clusters:

```
student-node ~ ➜  kubectl top pods -A --context cluster1 --no-headers | sort -nr -k3 | head -1
kube-system   kube-apiserver-cluster1-controlplane            30m   258Mi

student-node ~ ➜  kubectl top pods -A --context cluster2 --no-headers | sort -nr -k3 | head -1
kube-system   metrics-server-7cd5fcb6b7-fhdrl           5m    18Mi

student-node ~ ➜  kubectl top pods -A --context cluster3 --no-headers | sort -nr -k3 | head -1
kube-system   metrics-server-7cd5fcb6b7-zvfrg           5m    18Mi

student-node ~ ➜  kubectl top pods -A --context cluster4 --no-headers | sort -nr -k3 | head -1
kube-system   metrics-server-7cd5fcb6b7-zvfrg           5m    18Mi

student-node ~ ➜
```

Using this, find the pod that uses most CPU. In this case, it is kube-apiserver-cluster1-controlplane on cluster1.

Save the result in the correct format to the file:

student-node ~ ➜ echo cluster1,kube-system,kube-apiserver-cluster1-controlplane > /opt/high_cpu_pod

14. One of our applications runs on the cluster3-controlplane node. Due to the possibility of traffic increase, we want to scale the application pods to loadbalance the traffic and provide a smoother user experience.

cluster3-controlplane node has enough resources to deploy more application pods. Scale the deployment called essports-wl02 to 5.

15. Create a pod with name tester-cka02-svcn in dev-cka02-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3. Make sure to use command sleep 3600 with restart policy set to Always .

Once the tester-cka02-svcn pod is running, store the output of the command nslookup kubernetes.default from tester pod into the file /root/dns_output on student-node.

ANS:
Change to the cluster1 context before attempting the task:

kubectl config use-context cluster1

Since the "dev-cka02-svcn" namespace doesn't exist, let's create it first:

kubectl create ns dev-cka02-svcn

Create the pod as per the requirements:

```
kubectl apply -f - << EOF
apiVersion: v1
kind: Pod
metadata:
  name: tester-cka02-svcn
  namespace: dev-cka02-svcn
spec:
  containers:
  - name: tester-cka02-svcn
    image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "3600"
  restartPolicy: Always
EOF
```

Now let's test if the nslookup command is working :

student-node ~ ➜ kubectl exec -n dev-cka02-svcn -i -t tester-cka02-svcn -- nslookup kubernetes.default
;; connection timed out; no servers could be reached

command terminated with exit code 1

Looks like something is broken at the moment, if we observe the kube-system namespace, we will see no coredns pods are not running which is creating the problem, let's scale them for the nslookup command to work:

kubectl scale deployment -n kube-system coredns --replicas=2

Now let store the correct output into the /root/dns_output on student-node :

kubectl exec -n dev-cka02-svcn -i -t tester-cka02-svcn -- nslookup kubernetes.default >> /root/dns_output

We should have something similar to below output:

student-node ~ ➜ cat /root/dns_output
Server: 10.96.0.10
Address: 10.96.0.10#53

Name: kubernetes.default.svc.cluster.local
Address: 10.96.0.1

16. Create a storage class with the name banana-sc-cka08-str as per the properties given below:

- Provisioner should be kubernetes.io/no-provisioner,

- Volume binding mode should be WaitForFirstConsumer.

- Volume expansion should be enabled.

ANS:

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-cka08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

17. A new service account called thor-cka24-trb has been created in cluster1. Using this service account, we are trying to list and get the pods and secrets deployed in default namespace. However, this service account is not able to perform these operations.

Look into the issue and apply the appropriate fix(es) so that the service account thor-cka24-trb can perform these operations.

ANS:
Check if this service account is associated with any role binding

kubectl get rolebinding -o yaml | grep -B 5 -A 5 thor-cka24-trb

You will see role-cka24-trb is associated with this SA. So let's edit it to see the permissions

kubectl edit role role-cka24-trb

Update it so that resourcesand verbs section look as below:

```
resources:
- pods
- secrets
verbs:
- list
- get
```
