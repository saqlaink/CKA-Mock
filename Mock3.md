1. One of our Junior DevOps engineers have deployed a pod nginx-wl06 on the cluster3-controlplane node. However, while specifying the resource limits, instead of using Mebibyte as the unit, Gebibyte was used.

As a result, the node doesn't have sufficient resources to deploy this pod and it is stuck in a pending state

Fix the units and re-deploy the pod (Delete and recreate the pod if needed).

ANS:
Run the following command to check the pending pods on all the namespaces: -

kubectl get pods -A

After that, inspect the pod Events as follows: -

kubectl get pods -A | grep -i pending

kubectl describe po nginx-wl06

Make use of the kubectl edit command to update the values from Gi to Mi:-

kubectl edit po nginx-wl06

It will save the temporary file under the /tmp/ directory. Use the kubectl replace command as follows: -

kubectl replace -f /tmp/kubectl-edit-xxxx.yaml --force

It will delete the existing pod and will re-create it again with new changes.

2. Find the pod that consumes the most memory and store the result to the file /opt/high_memory_pod in the following format cluster_name,namespace,pod_name.

The pod could be in any namespace in any of the clusters that are currently configured on the student-node.

NOTE: It's recommended to wait for a few minutes to allow deployed objects to become fully operational and start consuming resources.

3. Part I:

Create a ClusterIP service .i.e. service-3421-svcn in the spectra-1267 ns which should expose the pods namely pod-23 and pod-21 with port set to 8080 and targetport to 80.

Part II:

Store the pod names and their ip addresses from the spectra-1267 ns at /root/pod_ips_cka05_svcn where the output is sorted by their IP's.

Please ensure the format as shown below:

POD_NAME IP_ADDR
pod-1 ip-1
pod-3 ip-2
pod-2 ip-3
...

ANS:
The easiest way to route traffic to a specific pod is by the use of labels and selectors . List the pods along with their labels:

student-node ~ ➜ kubectl get pods --show-labels -n spectra-1267
NAME READY STATUS RESTARTS AGE LABELS
pod-12 1/1 Running 0 5m21s env=dev,mode=standard,type=external
pod-34 1/1 Running 0 5m20s env=dev,mode=standard,type=internal
pod-43 1/1 Running 0 5m20s env=prod,mode=exam,type=internal
pod-23 1/1 Running 0 5m21s env=dev,mode=exam,type=external
pod-32 1/1 Running 0 5m20s env=prod,mode=standard,type=internal
pod-21 1/1 Running 0 5m20s env=prod,mode=exam,type=external

Looks like there are a lot of pods created to confuse us. But we are only concerned with the labels of pod-23 and pod-21.

As we can see both the required pods have labels mode=exam,type=external in common. Let's confirm that using kubectl too:

student-node ~ ➜ kubectl get pod -l mode=exam,type=external -n spectra-1267  
NAME READY STATUS RESTARTS AGE
pod-23 1/1 Running 0 9m18s
pod-21 1/1 Running 0 9m17s

Nice!! Now as we have figured out the labels, we can proceed further with the creation of the service:

student-node ~ ➜ kubectl create service clusterip service-3421-svcn -n spectra-1267 --tcp=8080:80 --dry-run=client -o yaml > service-3421-svcn.yaml

Now modify the service definition with selectors as required before applying to k8s cluster:

student-node ~ ➜ cat service-3421-svcn.yaml
apiVersion: v1
kind: Service
metadata:
creationTimestamp: null
labels:
app: service-3421-svcn
name: service-3421-svcn
namespace: spectra-1267
spec:
ports:

- name: 8080-80
  port: 8080
  protocol: TCP
  targetPort: 80
  selector:
  app: service-3421-svcn # delete
  mode: exam # add
  type: external # add
  type: ClusterIP
  status:
  loadBalancer: {}

Finally let's apply the service definition:

student-node ~ ➜ kubectl apply -f service-3421-svcn.yaml
service/service-3421 created

student-node ~ ➜ k get ep service-3421-svcn -n spectra-1267
NAME ENDPOINTS AGE
service-3421 10.42.0.15:80,10.42.0.17:80 52s

To store all the pod name along with their IP's , we could use imperative command as shown below:

student-node ~ ➜ kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP

POD_NAME IP_ADDR
pod-12 10.42.0.18
pod-23 10.42.0.19
pod-34 10.42.0.20
pod-21 10.42.0.21
...

# store the output to /root/pod_ips

student-node ~ ➜ kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn

4. A pod called logger-complete-cka04-arch has been created in the default namespace. Inspect this pod and save ALL the logs to the file /root/logger-complete-cka04-arch on the student-node.

ANS:
kubectl logs logger-complete-cka04-arch --context cluster3 > /root/logger-complete-cka04-arch

5. Create a deployment called app-wl01 using the nginx image and scale the application pods to 2.

6. Find the node across all clusters that consumes the most memory and store the result to the file /opt/high_memory_node in the following format cluster_name,node_name.

The node could be in any clusters that are currently configured on the student-node.

7. The green-deployment-cka15-trb deployment is having some issues since the corresponding POD is crashing and restarting multiple times continuously.

Investigate the issue and fix it, make sure the POD is in running state and its stable (i.e NO RESTARTS!).

ANS:
List the pods to check its status
kubectl get pod

its must have crashed already so lets look into the logs.

kubectl logs -f green-deployment-cka15-trb-xxxx

You will see some logs like these

2022-09-18 17:13:25 98 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2022-09-18 17:13:25 98 [Note] InnoDB: Memory barrier is not used
2022-09-18 17:13:25 98 [Note] InnoDB: Compressed tables use zlib 1.2.11
2022-09-18 17:13:25 98 [Note] InnoDB: Using Linux native AIO
2022-09-18 17:13:25 98 [Note] InnoDB: Using CPU crc32 instructions
2022-09-18 17:13:25 98 [Note] InnoDB: Initializing buffer pool, size = 128.0M
Killed

This might be due to the resources issue, especially the memory, so let's try to recreate the POD to see if it helps.

kubectl delete pod green-deployment-cka15-trb-xxxx

Now watch closely the POD status

kubectl get pod

Pretty soon you will see the POD status has been changed to OOMKilled which confirms its the memory issue. So let's look into the resources that are assigned to this deployment.

kubectl get deploy
kubectl edit deploy green-deployment-cka15-trb

Under resources: -> limits: change memory from 256Mi to 512Mi and save the changes.
Now watch closely the POD status again

kubectl get pod

It should be stable now.

8. A manifest file is available at the /root/app-wl03/ on the student-node node. There are some issues with the file; hence couldn't deploy a pod on the cluster3-controlplane node.

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

9. Run a pod called looper-cka16-arch using the busybox image that runs the while loop while true; do echo hello; sleep 10;done. This pod should be created in the default namespace.

ANS:
Create the pod definition:

student-node ~ ➜ vi looper-cka16-arch.yaml

##Content should be:

```
apiVersion: v1
kind: Pod
metadata:
  name: looper-cka16-arch
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10;done"]

```

Create the pod:
student-node ~ ➜ kubectl apply -f looper-cka16-arch.yaml --context cluster3

10. A pod definition file is created at /root/peach-pod-cka05-str.yaml on the student-node. Update this manifest file to create a persistent volume claim called peach-pvc-cka05-str to claim a 100Mi of storage from peach-pv-cka05-str PV (this is already created). Use the access mode ReadWriteOnce.

Further add peach-pvc-cka05-str PVC to peach-pod-cka05-str POD and mount the volume at /var/www/html location. Ensure that the pod is running and the PV is bound.

ANS:
Update /root/peach-pod-cka05-str.yaml template file to create a PVC to utilise the same in POD template.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: peach-pvc-cka05-str
spec:
  volumeName: peach-pv-cka05-str
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: peach-pod-cka05-str
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
      - mountPath: "/var/www/html"
        name: nginx-volume
  volumes:
    - name: nginx-volume
      persistentVolumeClaim:
        claimName: peach-pvc-cka05-str
```

Apply the template:
kubectl apply -f /root/peach-pod-cka05-str.yaml

11. Create a deployment named hr-web-app-cka08-svcn using the image kodekloud/webapp-color with 2 replicas.

Expose the hr-web-app-cka08-svcn as service hr-web-app-service-cka08-svcn application on port 30082 on the nodes of the cluster.

The web application listens on port 8080.

ANS:
On student-node, use the command: kubectl create deployment hr-web-app-cka08-svcn --image=kodekloud/webapp-color --replicas=2

Now we can run the command: kubectl expose deployment hr-web-app-cka08-svcn --type=NodePort --port=8080 --name=hr-web-app-service-cka08-svcn --dry-run=client -o yaml > hr-web-app-service-cka08-svcn.yaml to generate a service definition file.

Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.

12. A service account called deploy-cka19-trb is created in cluster1 along with a cluster role called deploy-cka19-trb-role. This role should have the permissions to get all the deployments under the default namespace. However, at the moment, it is not able to.

Find out what is wrong and correct it so that the deploy-cka19-trb service account is able to get deployments under default namespace.

ANS:
Let's see if deploy-cka19-trb service account is able to get the deployments.
kubectl auth can-i get deployments --as=system:serviceaccount:default:deploy-cka19-trb

We can see its not since we are getting no in the output.

Let's look into the cluster role:
kubectl get clusterrole deploy-cka19-trb-role -o yaml

The rules would be fine but we can see that there is no cluster role binding and service account associated with this. So let's create a cluster role binding.

kubectl create clusterrolebinding deploy-cka19-trb-role-binding --clusterrole=deploy-cka19-trb-role --serviceaccount=default:deploy-cka19-trb

Let's see if deploy-cka19-trb service account is able to get the deployments now.

kubectl auth can-i get deployments --as=system:serviceaccount:default:deploy-cka19-trb

13. demo-pod-cka29-trb pod is stuck in aPending state, look into issue to fix the same, Make sure pod is in Running state and stable.

ANS:
Look into the POD events
kubectl get event --field-selector involvedObject.name=demo-pod-cka29-trb

You will see some Warnings like:

Warning FailedScheduling pod/demo-pod-cka29-trb 0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

This seems to be something related to PersistentVolumeClaims, Let's check that:

kubectl get pvc

You will notice that demo-pvc-cka29-trb is stuck in Pending state. Let's dig into it

kubectl get event --field-selector involvedObject.name=demo-pvc-cka29-trb

You will notice this error:

Warning VolumeMismatch persistentvolumeclaim/demo-pvc-cka29-trb Cannot bind to requested volume "demo-pv-cka29-trb": incompatible accessMode

Which means the PVC is using incompatible accessMode, let's check the it out

kubectl get pvc demo-pvc-cka29-trb -o yaml
kubectl get pv demo-pv-cka29-trb -o yaml

Let's re-create the PVC with correct access mode i.e ReadWriteMany

kubectl get pvc demo-pvc-cka29-trb -o yaml > /tmp/pvc.yaml
vi /tmp/pvc.yaml

Under spec: change accessModes: from ReadWriteOnce to ReadWriteMany
Delete the old PVC and create new

kubectl delete pvc demo-pvc-cka29-trb
kubectl apply -f /tmp/pvc.yaml

Check the POD now

kubectl get pod demo-pod-cka29-trb

It should be good now.

14. We want to deploy a python based application on the cluster using a template located at /root/olive-app-cka10-str.yaml on student-node. However, before you proceed we need to make some modifications to the YAML file as per details given below:

The YAML should also contain a persistent volume claim with name olive-pvc-cka10-str to claim a 100Mi of storage from olive-pv-cka10-str PV.

Update the deployment to add a sidecar container, which can use busybox image (you might need to add a sleep command for this container to keep it running.)

Share the python-data volume with this container and mount the same at path /usr/src. Make sure this container only has read permissions on this volume.

Finally, create a pod using this YAML and make sure the POD is in Running state.
Note: Additionally, you can expose a NodePort service for the application. The service should be named olive-svc-cka10-str and expose port 5000 with a nodePort value of 32006.
However, inclusion/exclusion of this service won't affect the validation for this task.

ANS:
Update olive-app-cka10-str.yaml template so that it looks like as below:

```
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

Apply the template:
kubectl apply -f olive-app-cka10-str.yaml

15. A pod called elastic-app-cka02-arch is running in the default namespace. The YAML file for this pod is available at /root/elastic-app-cka02-arch.yaml on the student-node. The single application container in this pod writes logs to the file /var/log/elastic-app.log.

One of our logging mechanisms needs to read these logs to send them to an upstream logging server but we don't want to increase the read overhead for our main application container so recreate this POD with an additional sidecar container that will run along with the application container and print to the STDOUT by running the command tail -f /var/log/elastic-app.log. You can use busybox image for this sidecar container.

ANS:
Recreate the pod with a new container called sidecar. Update the /root/elastic-app-cka02-arch.yaml YAML file as shown below:

```
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
```

Next, recreate the pod:

student-node ~ ➜ kubectl replace -f /root/elastic-app-cka02-arch.yaml --force --context cluster3
pod "elastic-app-cka02-arch" deleted
pod/elastic-app-cka02-arch replaced

16. There is some issue on the student-node preventing it from accessing the cluster4 Kubernetes Cluster.

Troubleshoot and fix this issue. Make sure that you are able to run the kubectl commands (For example: kubectl get node --context=cluster4) from the student-node.

The kubeconfig for all the clusters is stored in the default kubeconfig file: /root/.kube/config on the student-node.

17. We have an external webserver running on student-node which is exposed at port 9999. We have created a service called external-webserver-cka03-svcn that can connect to our local webserver from within the kubernetes cluster3 but at the moment it is not working as expected.

Fix the issue so that other pods within cluster3 can use external-webserver-cka03-svcn service to access the webserver.

ANS:
Let's check if the webserver is working or not:

student-node ~ ➜ curl student-node:9999
...

<h1>Welcome to nginx!</h1>
...

Now we will check if service is correctly defined:

student-node ~ ➜ kubectl describe svc external-webserver-cka03-svcn
Name: external-webserver-cka03-svcn
Namespace: default
.
.
Endpoints: <none> # there are no endpoints for the service
...

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

student-node ~ ➜ export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')

```
student-node ~ ➜ kubectl --context cluster3 apply -f - <<EOF
apiVersion: v1
kind: Endpoints
metadata:
  # the name here should match the name of the Service
  name: external-webserver-cka03-svcn
subsets:
  - addresses:
      - ip: $IP_ADDR
    ports:
      - port: 9999
EOF
```

Finally check if the curl test works now:

student-node ~ ➜ kubectl --context cluster3 run --rm -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-cka03-svcn
...

<title>Welcome to nginx!</title>
...

18. Create a generic secret called db-user-pass-cka17-arch in the default namespace on cluster1 using the contents of the file /opt/db-user-pass on the student-node

ANS:
student-node ~ ➜ kubectl create secret generic db-user-pass-cka17-arch --from-file=/opt/db-user-pass

19. One of the nginx based pod called cyan-pod-cka28-trb is running under cyan-ns-cka28-trb namespace and it is exposed within the cluster using cyan-svc-cka28-trb service.

This is a restricted pod so a network policy called cyan-np-cka28-trb has been created in the same namespace to apply some restrictions on this pod.

Two other pods called cyan-white-cka28-trb and cyan-black-cka28-trb are also running in the default namespace.

The nginx based app running on the cyan-pod-cka28-trb pod is exposed internally on the default nginx port (80).

Expectation: This app should only be accessible from the cyan-white-cka28-trb pod.

Problem: This app is not accessible from anywhere.

Troubleshoot this issue and fix the connectivity as per the requirement listed above.

Note: You can exec into cyan-white-cka28-trb and cyan-black-cka28-trb pods and test connectivity using the curl utility.

You may update the network policy, but make sure it is not deleted from the cyan-ns-cka28-trb namespace.

ANS:
Let's look into the network policy

kubectl edit networkpolicy cyan-np-cka28-trb -n cyan-ns-cka28-trb

Under spec: -> egress: you will notice there is not cidr: block has been added, since there is no restrcitions on egress traffic so we can update it as below. Further you will notice that the port used in the policy is 8080 but the app is running on default port which is 80 so let's update this as well (under egress and ingress):

```
Change port: 8080 to port: 80
- ports:
  - port: 80
    protocol: TCP
  to:
  - ipBlock:
      cidr: 0.0.0.0/0
```

Now, lastly notice that there is no POD selector has been used in ingress section but this app is supposed to be accessible from cyan-white-cka28-trb pod under default namespace. So let's edit it to look like as below:

```
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: default
   podSelector:
      matchLabels:
        app: cyan-white-cka28-trb
```

Now, let's try to access the app from cyan-white-pod-cka28-trb

kubectl exec -it cyan-white-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local

Also make sure its not accessible from the other pod(s)

kubectl exec -it cyan-black-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local

It should not work from this pod. So its looking good now.

20. We tried to schedule grey-cka21-trb pod on cluster4 which was supposed to be deployed by the kubernetes scheduler so far but somehow its stuck in Pending state. Look into the issue and fix the same, make sure the pod is in Running state.

You can SSH into the cluster4 using ssh cluster4-controlplane command.

ANS:
Follow below given steps
Let's check the POD status
kubectl get pod --context=cluster4

You will see that grey-cka21-trb pod is stuck in Pending state. So let's try to look into the logs and events

kubectl logs grey-cka21-trb --context=cluster4
kubectl get event --context=cluster4 --field-selector involvedObject.name=grey-cka21-trb

You might not find any relevant info in the logs/events. Let's check the status of the kube-scheduler pod

kubectl get pod --context=cluster4 -n kube-system

You will notice that kube-scheduler-cluster4-controlplane pod us crashing, let's look into its logs

kubectl logs kube-scheduler-cluster4-controlplane --context=cluster4 -n kube-system

You will see an error as below:

run.go:74] "command failed" err="failed to get delegated authentication kubeconfig: failed to get delegated authentication kubeconfig: stat /etc/kubernetes/scheduler.config: no such file or directory"

From the logs we can see that its looking for a file called /etc/kubernetes/scheduler.config which seems incorrect, let's look into the kube-scheduler manifest on cluster4.

ssh cluster4-controlplane

First let's find out if /etc/kubernetes/scheduler.config

ls /etc/kubernetes/scheduler.config

You won't find it, instead the correct file is /etc/kubernetes/scheduler.conf so let's modify the manifest.

vi /etc/kubernetes/manifests/kube-scheduler.yaml

Search for config in the file, you will find some typos, change every occurence of /etc/kubernetes/scheduler.config to /etc/kubernetes/scheduler.conf.

Let's see if kube-scheduler-cluster4-controlplane is running now

kubectl get pod -A

It should be good now and grey-cka21-trb should be good as well.
