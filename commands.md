# Main commands in Kubernetes

Learning the concepts of Kubernetes described here: https://kubernetes.io/docs/concepts/


* [Basic initial commands](#basic-initial-commands)
* [Deploy your first app on Kubernetes using kubectl](#deploy-your-first-app-on-kubernetes-using-kubectl)
* [Troubleshoot Kubernetes applications](#troubleshoot-kubernetes-applications)
   * [Show the app in the terminal](#show-the-app-in-the-terminal)
   * [View the container logs](#view-the-container-logs)
   * [Executing command on the container](#executing-command-on-the-container)
* [Exposing Your App](#exposing-your-app)
   * [Create a new service](#create-a-new-service)
   * [Using labels](#using-labels)



## Basic initial commands

```shell
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2017-12-15T21:07:38Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"", Minor:"", GitVersion:"v1.9.0", GitCommit:"925c127ec6b946659ad0fd596fa959be43f0cc05", GitTreeState:"clean", BuildDate:"2018-01-26T19:04:38Z", GoVersion:"go1.9.1", Compiler:"gc", Platform:"linux/amd64"}
$
$
$
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.180:8443
$
$
$
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    4m        v1.9.0
# only one node, and ready!
```

## Deploy your first app on Kubernetes using `kubectl`

Let’s run our first app on Kubernetes with the `kubectl run` command. The `run` command creates a new deployment. We need to provide the deployment name and app image location (include the full repository url for images hosted outside Docker hub). We want to run the app on a specific port so we add the `--port` parameter:

```shell
$ kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080

deployment "kubernetes-bootcamp" created
```

To list your deployments use the get deployments command:

```shell
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           12s
```


Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network. When we use `kubectl`, we're interacting through an API endpoint to communicate with our application.

We will cover other options on how to expose your application outside the kubernetes cluster in Module 4.

The `kubectl` command can create a proxy that will forward communications into the cluster-wide, private network. The proxy can be terminated by pressing control-C and won't show any output while its running.

We will open a second terminal window to run the proxy.

```shell
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

We now have a connection between our host (the online terminal) and the Kubernetes cluster. The proxy enables direct access to the API from these terminals.


You can see all those APIs hosted through the proxy endpoint, now available at through http://localhost:8001. For example, we can query the version directly through the API using the `curl` command:

`curl http://localhost:8001/version`

you get

```shell
$ curl http://localhost:8001/version
{
  "major": "",
  "minor": "",
  "gitVersion": "v1.9.0",
  "gitCommit": "925c127ec6b946659ad0fd596fa959be43f0cc05",
  "gitTreeState": "clean",
  "buildDate": "2018-01-26T19:04:38Z",
  "goVersion": "go1.9.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}$
```

The API server will automatically create an endpoint for each pod, based on the pod name, that is also accessible through the proxy.

First we need to get the Pod name, and we'll store in the environment variable POD_NAME:

```shell
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5dbf48f7d4-h8p6n
```

Now we can make an HTTP request to the application running in that pod:

```shell
$ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5dbf48f7d4-h8p6n | v=1
```

The url is the route to the API of the Pod.



## Troubleshoot Kubernetes applications

Troubleshoot Kubernetes applications using the kubectl get, describe, logs and exec commands.


Let’s verify that the application we deployed in the previous scenario is running. We’ll use the kubectl get command and look for existing Pods:

```shell
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5dbf48f7d4-r88kt   1/1       Running   0          1mx
```

If no pods are running, list the Pods again.

Next, to view what containers are inside that Pod and what images are used to build those containers we run the describe pods command:

```shell
$ kubectl describe pods
Name:           kubernetes-bootcamp-5dbf48f7d4-r88kt
Namespace:      default
Node:           host01/172.17.0.71
Start Time:     Tue, 10 Apr 2018 12:35:36 +0000
Labels:         pod-template-hash=1869049380
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.18.0.2
Controlled By:  ReplicaSet/kubernetes-bootcamp-5dbf48f7d4
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://c883764315bbb61b73a18264f0ef8ada0412c3c1922310750032fa44f93abd3f
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           8080/TCP
    State:          Running
      Started:      Tue, 10 Apr 2018 12:35:48 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qksnz (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-qksnz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qksnz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type     Reason                 Age              From               Message
  ----     ------                 ----             ----               -------
  Warning  FailedScheduling       2m (x2 over 2m)  default-scheduler  0/1 nodes are available: 1 NodeNotReady.
  Normal   Scheduled              2m               default-scheduler  Successfully assigned kubernetes-bootcamp-5dbf48f7d4-r88kt to host01
  Normal   SuccessfulMountVolume  2m               kubelet, host01    MountVolume.SetUp succeeded for volume "default-token-qksnz"
  Normal   Pulled                 1m               kubelet, host01    Container image "gcr.io/google-samples/kubernetes-bootcamp:v1" already present on machine
  Normal   Created                1m               kubelet, host01    Created container
  Normal   Started                1m               kubelet, host01    Started container
$
```

We see here details about the Pod’s container: IP address, the ports used and a list of events related to the lifecycle of the Pod.

The output of the describe command is extensive and covers some concepts that we didn’t explain yet, but don’t worry, they will become familiar by the end of this bootcamp.


### Show the app in the terminal

Recall that Pods are running in an isolated, private network - so we need to proxy access to them so we can debug and interact with them. To do this, we'll use the kubectl proxy command to run a proxy in a second terminal window. Click on the command below to automatically open a new terminal and run the proxy:

```shell
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

Now again, we'll get the Pod name and query that pod directly through the proxy. To get the Pod name and store it in the POD_NAME environment variable:

```shell
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-5dbf48f7d4-r88kt
```

To see the output of our application, run a curl request.

```shell
$ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5dbf48f7d4-r88kt | v=1
```

The url is the route to the API of the Pod.

### View the container logs

Anything that the application would normally send to STDOUT becomes logs for the container within the Pod. We can retrieve these logs using the kubectl logs command:

```shell
$ kubectl logs $POD_NAME
Kubernetes Bootcamp App Started At: 2018-04-10T12:35:48.710Z | Running On:  kubernetes-bootcamp-5dbf48f7d4-r88kt

Running On: kubernetes-bootcamp-5dbf48f7d4-r88kt | Total Requests: 1 | App Uptime: 247.554 seconds | Log Time: 2018-04-10T12:39:56.264Z
```

Note: We don’t need to specify the container name, because we only have one container inside the pod.


### Executing command on the container

We can execute commands directly on the container once the Pod is up and running. For this, we use the `exec` command and use the name of the Pod as a parameter. Let’s list the environment variables:

```shell
$ kubectl exec $POD_NAME env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-5dbf48f7d4-r88kt
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
NPM_CONFIG_LOGLEVEL=info
NODE_VERSION=6.3.1
HOME=/root
$
```

Again, worth mentioning that the name of the container itself can be omitted since we only have a single container in the Pod.

Next let’s start a bash session in the Pod’s container:

```shell
$ kubectl exec -ti $POD_NAME bash
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/#
```

We have now an open console on the container where we run our NodeJS application. The source code of the app is in the server.js file:

```shell
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/# cat server.js
var http = require('http');
var requests=0;
var podname= process.env.HOSTNAME;
var startTime;
var host;
var handleRequest = function(request, response) {
  response.setHeader('Content-Type', 'text/plain');
  response.writeHead(200);
  response.write("Hello Kubernetes bootcamp! | Running on: ");
  response.write(host);
  response.end(" | v=1\n");
  console.log("Running On:" ,host, "| Total Requests:", ++requests,"| App Uptime:", (new Date() - startTime)/1000 , "seconds", "| Log Time:",new Date());
}
var www = http.createServer(handleRequest);
www.listen(8080,function () {
    startTime = new Date();;
    host = process.env.HOSTNAME;
    console.log ("Kubernetes Bootcamp App Started At:",startTime, "| Running On: " ,host, "\n" );
});
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/#
```

You can check that the application is up by running a curl command:

```shell
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/# curl localhost:8080
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5dbf48f7d4-r88kt | v=1
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/#
```

Note: here we used localhost because we executed the command inside the NodeJS container

To close your container connection type `exit`.

```shell
root@kubernetes-bootcamp-5dbf48f7d4-r88kt:/# exit
exit
$
```


## Exposing Your App

Docs on "Using a Service to Expose Your App": https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/

How to expose Kubernetes applications outside the cluster using the kubectl expose command. You will also learn how to view and apply labels to objects with the kubectl label command.


### Create a new service
Let’s verify that our application is running. We’ll use the kubectl get command and look for existing Pods:

```shell
$ kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
kubernetes-bootcamp-5dbf48f7d4-428gs   1/1       Running   0          1m
```

Next let’s list the current Services from our cluster:

```shell
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m
```

We have a Service called kubernetes that is created by default when minikube starts the cluster. To create a new service and expose it to external traffic we’ll use the expose command with NodePort as parameter (minikube does not support the LoadBalancer option yet)

```shell
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
service "kubernetes-bootcamp" exposed
```

Let’s run again the get services command:

```shell
$ kubectl get services
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          4m
kubernetes-bootcamp   NodePort    10.106.74.87   <none>        8080:31956/TCP   1m
```

We have now a running Service called kubernetes-bootcamp. Here we see that the Service received a unique cluster-IP, an internal port and an external-IP (the IP of the Node).

To find out what port was opened externally (by the NodePort option) we’ll run the `describe service` command:

```shell
$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.106.74.87
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31956/TCP
Endpoints:                172.18.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$
```

Create an environment variable called NODE_PORT that has the value of the Node port assigned:

```shell
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ echo NODE_PORT=$NODE_PORT
NODE_PORT=31956
```

Now we can test that the app is exposed outside of the cluster using `curl`, the IP of the Node and the externally exposed port:

```shell
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-5dbf48f7d4-428gs | v=1
```

And we get a response from the server. The Service is exposed.

### Using labels

The Deployment created automatically a label for our Pod. With `describe deployment` command you can see the name of the label:

```shell
kubectl describe deployment
```

Let’s use this label to query our list of Pods. We’ll use the kubectl get pods command with -l as a parameter, followed by the label values:

```shell
kubectl get pods -l run=kubernetes-bootcamp
```

You can do the same to list the existing services:

```shell
kubectl get services -l run=kubernetes-bootcamp
```

Get the name of the Pod and store it in the POD_NAME environment variable:

```shell
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
```

To apply a new label we use the label command followed by the object type, object name and the new label:

```shell
kubectl label pod $POD_NAME app=v1
```

This will apply a new label to our Pod (we pinned the application version to the Pod), and we can check it with the describe pod command:

```shell
kubectl describe pods $POD_NAME
```

We see here that the label is attached now to our Pod. And we can query now the list of pods using the new label:

```shell
kubectl get pods -l app=v1
```

And we see the Pod.

