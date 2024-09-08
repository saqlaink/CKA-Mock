1. A template to create a Kubernetes pod is stored at /root/red-probe-cka12-trb.yaml on the student-node. However, using this template as-is is resulting in an error.

Fix the issue with this template and use it to create the pod. Once created, watch the pod for a minute or two to make sure its stable i.e, it's not crashing or restarting.

Make sure you do not update the args: section of the template.

ANS:
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

21s Warning Unhealthy pod/red-probe-cka12-trb Liveness probe failed: cat: can't open '/healthcheck': No such file or directory

So seems like Liveness probe is failing, lets look into it:

vi red-probe-cka12-trb.yaml

Notice the command - sleep 3 ; touch /healthcheck; sleep 30;sleep 30000 it starts with a delay of 3 seconds, but the liveness probe initialDelaySeconds is set to 1 and failureThreshold is also 1. Which means the POD will fail just after first attempt of liveness check which will happen just after 1 second of pod start. So to make it stable we must increase the initialDelaySeconds to at least 5

vi red-probe-cka12-trb.yaml

Change initialDelaySeconds from 1 to 5 and save apply the changes.
Delete old pod:

kubectl delete pod red-probe-cka12-trb

Apply changes:

kubectl apply -f red-probe-cka12-trb.yaml

2. Create a persistent volume called data-pv-cka02-str with the below properties:

- Its capacity should be 128Mi.
- The volume type should be hostpath and path should be /opt/data-pv-cka02-str.

Next, create a persistent volume claim called data-pvc-cka02-str as per below properties:

- Request 50Mi of storage from data-pv-cka02-str PV.

ANS

```
Create a yaml template as below:
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv-cka02-str
spec:
  capacity:
    storage: 128Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/data-pv-cka02-str
  storageClassName: manual
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc-cka02-str
spec:
  storageClassName: manual
  volumeName: data-pv-cka02-str
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

3. Decode the existing secret called beta-sec-cka14-arch created in the beta-ns-cka14-arch namespace and store the decoded content inside the file /opt/beta-sec-cka14-arch on the student-node.

ANS:
Look for the data in beta-sec-cka14-arch secret:

student-node ~ ➜ kubectl get secret beta-sec-cka14-arch -o json --context cluster3 -n beta-ns-cka14-arch

You will find only one data entry in it called secret . Let's decode it and save output in /opt/beta-sec-cka14-arch file:

student-node ~ ➜ kubectl get secret beta-sec-cka14-arch --context cluster3 -n beta-ns-cka14-arch -o json | jq .data.secret | tr -d '"' | base64 -d > /opt/beta-sec-cka14-arch

4. On cluster4 we are having some weird issue where we are intermittently getting below error while running kubectl commands.

The connection to the server cluster4-controlplane:6443 was refused - did you specify the right host or port?

Whenever you get this error, you can wait for 10-15 seconds to make kubectl command work again, but it will come again after few second

We also noticed that kube-controller-manager-cluster4-controlplane pod is restarting continuously. Look into the issue and troubleshoot the same.

You can SSH into the cluster4 using ssh cluster4-controlplane command.

5. There is a deployment called nginx-dp-cka04-trb which has been used to deploy a static website. The access to this website can be tested by running: curl http://kodekloud-exam.app:30002. However, it is not working at the moment.

Troubleshoot and fix it.

ANS:
First list the available pods:
kubectl get pod

Look into the nginx-dp-xxxx POD logs
kubectl logs -f <pod-name>

You may not see any logs so look into the kubernetes events for <pod-name> POD

Look into the POD events
kubectl get event --field-selector involvedObject.name=<pod-name>

You will see an error something like:

70s Warning FailedMount pod/nginx-dp-cka04-trb-767b767dc-6c5wk Unable to attach or mount volumes: unmounted volumes=[nginx-config-volume-cka04-trb], unattached volumes=[index-volume-cka04-trb kube-api-access-4fbrb nginx-config-volume-cka04-trb]: timed out waiting for the condition

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

Under configMap: change nginx-configuration-cka04-trb to nginx-config-cka04-trb. Once done wait for the POD to come up.

Try to access the website now:

curl http://kodekloud-exam.app:30002

6. The blue-dp-cka09-trb deployment is having 0 out of 1 pods running. Fix the issue to make sure that pod is up and running.

ANS:
List the pods
kubectl get pod

Most probably you see Init:Error or Init:CrashLoopBackOff for the corresponding pod.

Look into the logs
kubectl logs blue-dp-cka09-trb-xxxx -c init-container

You will see an error something like

sh: can't open 'echo 'Welcome!'': No such file or directory

Edit the deployment
kubectl edit deploy blue-dp-cka09-trb

Under initContainers: -> - command: add -c to the next line of - sh, so final command should look like this
initContainers:

- command:
  - sh
  - -c
  - echo 'Welcome!'

If you will check pod then it must be failing again but with different error this time, let's find that out

kubectl get event --field-selector involvedObject.name=blue-dp-cka09-trb-xxxxx

You will see an error something like

Warning Failed pod/blue-dp-cka09-trb-69dd844f76-rv9z8 Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config" to rootfs at "/etc/nginx/nginx.conf": mount /var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config:/etc/nginx/nginx.conf (via /proc/self/fd/6), flags: 0x5001: not a directory: unknown

Edit the deployment again
kubectl edit deploy blue-dp-cka09-trb

```
Under volumeMounts: -> - mountPath: /etc/nginx/nginx.conf -> name: nginx-config add subPath: nginx.conf and save the changes.
```

Finally the pod should be in running state.

7. There is a sample script located at /root/service-cka25-arch.sh on the student-node.
   Update this script to add a command to filter/display the targetPort only for service service-cka25-arch using jsonpath. The service has been created under the default namespace on cluster1.

ANS:
Update service-cka25-arch.sh script:

student-node ~ ➜ vi service-cka25-arch.sh

Add below command in it:

kubectl --context cluster1 get service service-cka25-arch -o jsonpath='{.spec.ports[0].targetPort}'

8. There is a requirement to share a volume between two containers that are running within the same pod. Use the following instructions to create the pod and related objects:

- Create a pod named grape-pod-cka06-str.

- The main container should use the nginx image and mount a volume called grape-vol-cka06-str at path /var/log/nginx.

- The sidecar container can use busybox image, you might need to add a sleep command to this container to keep it running. Next, mount the same volume called grape-vol-cka06-str at the path /usr/src.

- The volume should be of type emptyDir.

ANS:

```
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
```

9. There is an existing persistent volume called orange-pv-cka13-trb. A persistent volume claim called orange-pvc-cka13-trb is created to claim storage from orange-pv-cka13-trb.

However, this PVC is stuck in a Pending state. As of now, there is no data in the volume.

Troubleshoot and fix this issue, making sure that orange-pvc-cka13-trb PVC is in Bound state.

ANS:
List the PVC to check its status
kubectl get pvc

So we can see orange-pvc-cka13-trb PVC is in Pending state and its requesting a storage of 150Mi. Let's look into the events

kubectl get events --sort-by='.metadata.creationTimestamp' -A

You will see some errors as below:

Warning VolumeMismatch persistentvolumeclaim/orange-pvc-cka13-trb Cannot bind to requested volume "orange-pv-cka13-trb": requested PV is too small

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

10. We deployed an app using a deployment called web-dp-cka06-trb. it's using the httpd:latest image. There is a corresponding service called web-service-cka06-trb that exposes this app on the node port 30005. However, the app is not accessible!

Troubleshoot and fix this issue. Make sure you are able to access the app using curl http://kodekloud-exam.app:30005 command.

ANS:
List the deployments to see if all PODs under web-dp-cka06-trb deployment are up and running.
kubectl get deploy

You will notice that 0 out of 1 PODs are up, so let's look into the POD now.

kubectl get pod

You will notice that web-dp-cka06-trb-xxx pod is in Pending state, so let's checkout the relevant events.

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxx

You should see some error/warning like this:

Warning FailedScheduling pod/web-dp-cka06-trb-76b697c6df-h78x4 0/1 nodes are available: 1 persistentvolumeclaim "web-cka06-trb" not found. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

Let's look into the PVCs

kubectl get pvc

You should see web-pvc-cka06-trb in the output but as per logs the POD was looking for web-cka06-trb PVC. Let's update the deployment to fix this.

kubectl edit deploy web-dp-cka06-trb

Under volumes: -> name: web-str-cka06-trb -> persistentVolumeClaim: -> claimName change web-cka06-trb to web-pvc-cka06-trb and save the changes.

Look into the POD again to make sure its running now

kubectl get pod

You will find that its still failing, most probably with ErrImagePull or ImagePullBackOff error. Now lets update the deployment again to make sure its using the correct image.

kubectl edit deploy web-dp-cka06-trb

Under spec: -> containers: -> change image from httpd:letest to httpd:latest and save the changes.
Look into the POD again to make sure its running now

kubectl get pod

You will notice that POD is still crashing, let's look into the POD logs.

kubectl logs web-dp-cka06-trb-xxxx

If there are no useful logs then look into the events

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxxx --sort-by='.lastTimestamp'

You should see some errors/warnings as below

Warning FailedPostStartHook pod/web-dp-cka06-trb-67dccb7487-2bjgf Exec lifecycle hook ([/bin -c echo 'Test Page' > /usr/local/apache2/htdocs/index.html]) for Container "web-container" in Pod "web-dp-cka06-trb-67dccb7487-2bjgf_default(4dd6565e-7f1a-4407-b3d9-ca595e6d4e95)" failed - error: rpc error: code = Unknown desc = failed to exec in container: failed to start exec "c980799567c8176db5931daa2fd56de09e84977ecd527a1d1f723a862604bd7c": OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin": permission denied: unknown, message: ""

Let's look into the lifecycle hook of the pod

kubectl edit deploy web-dp-cka06-trb

Under containers: -> lifecycle: -> postStart: -> exec: -> command: change /bin to /bin/sh
Look into the POD again to make sure its running now

kubectl get pod

Finally pod should be in running state. Let's try to access the webapp now.

curl http://kodekloud-exam.app:30005

You will see error curl: (7) Failed to connect to kodekloud-exam.app port 30005: Connection refused
Let's look into the service

kubectl edit svc web-service-cka06-trb

Let's verify if the selector labels and ports are correct as needed. You will note that service is using selector: -> app: web-cka06-trb
Now, let's verify the app labels:

kubectl get deploy web-dp-cka06-trb -o yaml

Under labels you will see labels: -> deploy: web-app-cka06-trb
So we can see that service is using wrong selector label, let's edit the service to fix the same

kubectl edit svc web-service-cka06-trb

Let's try to access the webapp now.

curl http://kodekloud-exam.app:30005

Boom! app should be accessible now.

11. A YAML template for a Kubernetes deployment is stored at /root/app-cka07-trb.yaml. However, creating a deployment using this file is failing. Investigate the cause of the errors and fix the issue.
    Make sure that the pod is in running state once deployed.

Note: Do not to make any changes in the template file.

ANS:
Try to apply the template:
kubectl apply -f /root/app-cka07-trb.yaml

You will see an error something like:

Error from server (NotFound): error when creating "/root/app-cka07-trb.yaml": namespaces "app-cka07-trb" not found
Error from server (NotFound): error when creating "/root/app-cka07-trb.yaml": namespaces "app-cka07-trb" not found

From the error you can see that its looking for app-cka07-trb namespace so let's find out if this namespace exists:

kubectl get ns

You will not see this namespace in the list so let's create it.

Create the namespace
kubectl create ns app-cka07-trb

Apply the template
kubectl a apply -f /root/app-cka07-trb.yaml

Verify the POD is up
kubectl get pod -n app-cka07-trb

12. A pod called check-time-cka03-trb is continuously crashing. Figure out what is causing this and fix it.

Make sure that the check-time-cka03-trb POD is in running state.

This pod prints the current date and time at a pre-defined frequency and saves it to a file. Ensure that it continues this operation once you have fixed it.

ANS:
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

13. An etcd backup is already stored at the path /opt/cluster1_backup_to_restore.db on the cluster1-controlplane node. Use /root/default.etcd as the --data-dir and restore it on the cluster1-controlplane node itself.

You can ssh to the controlplane node by running ssh root@cluster1-controlplane from the student-node.

ANS:
SSH into cluster1-controlplane node:

student-node ~ ➜ ssh root@cluster1-controlplane

Install etcd utility (if not installed already) and restore the backup:

```
cluster1-controlplane ~ ➜ cd /tmp
cluster1-controlplane ~ ➜ export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep tag_name | cut -d '"' -f 4)
cluster1-controlplane ~ ➜ wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
cluster1-controlplane ~ ➜ tar xvf etcd-${RELEASE}-linux-amd64.tar.gz ; cd etcd-${RELEASE}-linux-amd64
cluster1-controlplane ~ ➜ mv etcd etcdctl /usr/local/bin/
cluster1-controlplane ~ ➜ etcdctl snapshot restore --data-dir /root/default.etcd /opt/cluster1_backup_to_restore.db
```

14. We have deployed a 2-tier web application on the cluster3 nodes in the canara-wl05 namespace. However, at the moment, the web app pod cannot establish a connection with the MySQL pod successfully.

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

```
curl http://10.17.63.11:31020
<!doctype html>
<title>Hello from Flask</title>
...

    <img src="/static/img/failed.png">
    <h3> Failed connecting to the MySQL database. </h3>


    <h2> Environment Variables: DB_Host=Not Set; DB_Database=Not Set; DB_User=Not Set; DB_Password=Not Set; 2003: Can&#39;t connect to MySQL server on &#39;localhost:3306&#39; (111 Connection refused) </h2>
```

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

```
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #39b54b;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">


    <img src="/static/img/success.jpg">
    <h3> Successfully connected to the MySQL database.</h3>
```

15. Create a deployment called app-wl01 using the nginx image and scale the application pods to 2.

16. One of our applications runs on the cluster3-controlplane node. Due to the possibility of traffic increase, we want to scale the application pods to loadbalance the traffic and provide a smoother user experience.

cluster3-controlplane node has enough resources to deploy more application pods. Scale the deployment called essports-wl02 to 5.

ANS:
Set the correct context:

kubectl config use-context cluster3

Now, get the details of the nodes: -

kubectl get nodes -owide

then SSH to the given node by the following command: -

ssh cluster3-controlplane

And run the kubectl scale command as follows: -

kubectl scale deploy essports-wl02 --replicas=5

OR

You can run the kubectl scale command from the student node as well: -

kubectl scale deploy essports-wl02 --replicas=5

Verify the scaled-up pods by kubectl get command: -

kubectl get pods

The number of pods should be 1 to 5.

17. A manifest file is available at the /root/app-wl03/ on the student-node node. There are some issues with the file; hence couldn't deploy a pod on the cluster3-controlplane node.

After fixing the issues, deploy the pod, and it should be in a running state.

NOTE: - Ensure that the existing limits are unchanged.

ANS:
Use the cd command to move to the given directory: -

cd /root/app-wl03/

While creating the resource, you will see the error output as follows: -

kubectl create -f app-wl03.yaml
The Pod "app-wl03" is invalid: spec.containers[0].resources.requests: Invalid value: "1Gi": must be less than or equal to memory limit

In the spec.containers.resources.requests.memory value is not configured as compare to the memory limit.

As a fix, open the manifest file with the text editor such as vim or nano and set the value to 100Mi or less than 100Mi.

It should be look like as follows: -

```
resources:
     requests:
       memory: 100Mi
     limits:
       memory: 100Mi
```

Final, create the resource from the kubectl create command: -

kubectl create -f app-wl03.yaml
pod/app-wl03 created

18. John is setting up a two tier application stack that is supposed to be accessible using the service curlme-cka01-svcn. To test that the service is accessible, he is using a pod called curlpod-cka01-svcn. However, at the moment, he is unable to get any response from the application.

Troubleshoot and fix this issue so the application stack is accessible.

While you may delete and recreate the service curlme-cka01-svcn, please do not alter it in anyway.

ANS:
Test if the service curlme-cka01-svcn is accessible from pod curlpod-cka01-svcn or not.

kubectl exec curlpod-cka01-svcn -- curl curlme-cka01-svcn

.....
% Total % Received % Xferd Average Speed Time Time Time Current
Dload Upload Total Spent Left Speed
0 0 0 0 0 0 0 0 --:--:-- 0:00:10 --:--:-- 0

We did not get any response. Check if the service is properly configured or not.

kubectl describe svc curlme-cka01-svcn ''

```
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
```

The service has no endpoints configured. As we can delete the resource, let's delete the service and create the service again.

To delete the service, use the command kubectl delete svc curlme-cka01-svcn.
You can create the service using imperative way or declarative way.

Using imperative command:
kubectl expose pod curlme-cka01-svcn --port=80

Using declarative manifest:

```
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
```

You can test the connection from curlpod-cka-1-svcn using following.

kubectl exec curlpod-cka01-svcn -- curl curlme-cka01-svcn

19. Create a pod with name tester-cka02-svcn in dev-cka02-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3. Make sure to use command sleep 3600 with restart policy set to Always .

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

20. Part I:

Create a ClusterIP service .i.e. service-3421-svcn in the spectra-1267 ns which should expose the pods namely pod-23 and pod-21 with port set to 8080 and targetport to 80.

Part II:

Store the pod names and their ip addresses from the spectra-1267 ns at /root/pod_ips_cka05_svcn where the output is sorted by their IP's.

Please ensure the format as shown below:
