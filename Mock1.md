1. Create a pod with name tester-cka02-svcn in dev-cka02-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3. Make sure to use command sleep 3600 with restart policy set to Always .
   Once the tester-cka02-svcn pod is running, store the output of the command nslookup kubernetes.default from tester pod into the file /root/dns_output on student-node.

```markdown
kubectl create ns dev-cka02-svcn
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
  command: - sleep - "3600"
  restartPolicy: Always
  EOF

student-node ~ ➜ kubectl exec -n dev-cka02-svcn -i -t tester-cka02-svcn -- nslookup kubernetes.default
;; connection timed out; no servers could be reached

command terminated with exit code 1

Looks like something is broken at the moment, if we observe the kube-system namespace, we will see no coredns pods are not running which is creating the problem, let's scale them for the nslookup command to work:

kubectl scale deployment -n kube-system coredns --replicas=2

kubectl exec -n dev-cka02-svcn -i -t tester-cka02-svcn -- nslookup kubernetes.default >> /root/dns_output
```

2. Run a pod called alpine-sleeper-cka15-arch using the alpine image in the default namespace that will sleep for 7200 seconds.

3. We have created a service account called green-sa-cka22-arch, a cluster role called green-role-cka22-arch and a cluster role binding called green-role-binding-cka22-arch.

Update the permissions of this service account so that it can only get all the namespaces in cluster1.

4. There is a Cronjob called orange-cron-cka10-trb which is supposed to run every two minutes (i.e 13:02, 13:04, 13:06…14:02, 14:04…and so on). This cron targets the application running inside the orange-app-cka10-trb pod to make sure the app is accessible. The application has been exposed internally as a ClusterIP service.

However, this cron is not running as per the expected schedule and is not running as intended.

Make the appropriate changes so that the cronjob runs as per the required schedule and it passes the accessibility checks every-time.

5.The pink-depl-cka14-trb Deployment was scaled to 2 replicas however, the current replicas is still 1.

Troubleshoot and fix this issue. Make sure the CURRENT count is equal to the DESIRED count.

You can SSH into the cluster4 using ssh cluster4-controlplane command.

6. A persistent volume called papaya-pv-cka09-str is already created with a storage capacity of 150Mi. It's using the papaya-stc-cka09-str storage class with the path /opt/papaya-stc-cka09-str.

Also, a persistent volume claim named papaya-pvc-cka09-str has also been created on this cluster. This PVC has requested 50Mi of storage from papaya-pv-cka09-str volume.

Resize the PVC to 80Mi and make sure the PVC is in Bound state.

7. A pod called logger-cka03-arch has been created in the default namespace. Inspect this pod and save ALL INFO and ERROR's to the file /root/logger-cka03-arch-all on the student-node.

8. Create a ReplicaSet with name checker-cka10-svcn in ns-12345-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3.

Make sure to specify the below specs as well:

command sleep 3600
replicas set to 2
container name: dns-image

Once the checker pods are up and running, store the output of the command nslookup kubernetes.default from any one of the checker pod into the file /root/dns-output-12345-cka10-svcn on student-node.

9. Create a service account called deploy-cka20-arch. Further create a cluster role called deploy-role-cka20-arch with permissions to get the deployments in cluster1.

Finally create a cluster role binding called deploy-role-binding-cka20-arch to bind deploy-role-cka20-arch cluster role with deploy-cka20-arch service account.

10. Create a nginx pod called nginx-resolver-cka06-svcn using image nginx, expose it internally with a service called nginx-resolver-service-cka06-svcn.

Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc.cka06.svcn and /root/CKA/nginx.pod.cka06.svcn

11. We tried to schedule grey-cka21-trb pod on cluster4 which was supposed to be deployed by the kubernetes scheduler so far but somehow its stuck in Pending state. Look into the issue and fix the same, make sure the pod is in Running state.

You can SSH into the cluster4 using ssh cluster4-controlplane command.

12. The yello-cka20-trb pod is stuck in a Pending state. Fix this issue and get it to a running state. Recreate the pod if necessary.

Do not remove any of the existing taints that are set on the cluster nodes.

13. In the dev-wl07 namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.

To get back the working state, rollback the application to the previous version .

After rolling the deployment back, on the controlplane node, save the image currently in use to the /root/rolling-back-record.txt file and increase the replica count to the 5.

You can SSH into the cluster1 using ssh cluster1-controlplane command.

```markdown
Check the status of the pod: -
kubectl get pods -n dev-wl07

One of the pods is in an error state. As a quick fix, we need to rollback to the previous revision as follows: -

kubectl rollout undo -n dev-wl07 deploy webapp-wl07

After successful rolling back, inspect the updated image: -

kubectl describe deploy -n dev-wl07 webapp-wl07 | grep -i image

On the Controlplane node, save the image name to the given path /root/rolling-back-record.txt: -

ssh cluster1-controlplane

echo "kodekloud/webapp-color" > /root/rolling-back-record.txt

And increase the replica count to the 5 with help of kubectl scale command: -

kubectl scale deploy -n dev-wl07 webapp-wl07 --replicas=5

Verify it by running the command: kubectl get deploy -n dev-wl07
```

14. The db-deployment-cka05-trb deployment is having 0 out of 1 PODs ready.
    Figure out the issues and fix the same but make sure that you do not remove any DB related environment variables from the deployment/pod.

```markdown
Find out the name of the DB POD:
kubectl get pod

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
```

15. Create a new deployment called ocean-tv-wl09 in the default namespace using the image kodekloud/webapp-color:v1.
    Use the following specs for the deployment:

    Replica count should be 3.

    Set the Max Unavailable to 40% and Max Surge to 55%.

    Create the deployment and ensure all the pods are ready.

    After successful deployment, upgrade the deployment image to kodekloud/webapp-color:v2 and inspect the deployment rollout status.

    Check the rolling history of the deployment and on the student-node, save the current revision count number to the /opt/revision-count.txt file.

    Finally, perform a rollback and revert back the deployment image to the older version.

16. There is a script located at /root/pod-cka26-arch.sh on the student-node. Update this script to add a command to filter/display the label with value component of the pod called kube-apiserver-cluster1-controlplane (on cluster1) using jsonpath.

```markdown
Update pod-cka26-arch.sh script:

student-node ~ ➜ vi pod-cka26-arch.sh

Add below command in it:

kubectl --context cluster1 get pod -n kube-system kube-apiserver-cluster1-controlplane -o jsonpath='{.metadata.labels.component}'
```

17. There is a deployment nginx-deployment-cka04-svcn in cluster3 which is exposed using service nginx-service-cka04-svcn.

Create an ingress resource nginx-ingress-cka04-svcn to load balance the incoming traffic with the following specifications:

pathType: Prefix and path: /

Backend Service Name: nginx-service-cka04-svcn

Backend Service Port: 80

ssl-redirect is set to false

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
  paths: - path: /
  pathType: Prefix
  backend:
  service:
  name: nginx-service-cka04-svcn
  port:
  number: 80
  EOF

Check if the ingress resource was successfully created:

student-node ~ ➜ kubectl get ingress
NAME CLASS HOSTS ADDRESS PORTS AGE
nginx-ingress-cka04-svcn <none> \* 172.25.0.10 80 13s

As the ingress controller is exposed on cluster3-controlplane using traefik service, we need to ssh to cluster3-controlplane first to check if the ingress resource works properly:

student-node ~ ➜ ssh cluster3-controlplane

cluster3-controlplane:~# curl -I 172.25.0.11
HTTP/1.1 200 OK
...

18. It appears that the black-cka25-trb deployment in cluster1 isn't up to date. While listing the deployments, we are currently seeing 0 under the UP-TO-DATE section for this deployment. Troubleshoot, fix and make sure that this deployment is up to date.

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

19. The purple-app-cka27-trb pod is an nginx based app on the container port 80. This app is exposed within the cluster using a ClusterIP type service called purple-svc-cka27-trb.

There is another pod called purple-curl-cka27-trb which continuously monitors the status of the app running within purple-app-cka27-trb pod by accessing the purple-svc-cka27-trb service using curl.

Recently we started seeing some errors in the logs of the purple-curl-cka27-trb pod.

Dig into the logs to identify the issue and make sure it is resolved.

Note: You will not be able to access this app directly from the student-node but you can exec into the purple-app-cka27-trb pod to check.

Solution
Check the purple-curl-cka27-trb pod logs

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

20. Create a storage class with the name banana-sc-cka08-str as per the properties given below:

- Provisioner should be kubernetes.io/no-provisioner,

- Volume binding mode should be WaitForFirstConsumer.

- Volume expansion should be enabled.
