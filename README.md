 Kubernetes For Everyone
-----------------------

## Log Into Google Cloud

Visit [Google Cloud Shell](https://console.cloud.google.com/home/dashboard?cloudshell=true) and login with the details provided to you.

In the cloud shell that appears on the bottom of your screen enter the following:

```bash
gcloud container clusters get-credentials my-first-cluster-1 --zone europe-central2-a --project kubernetes-345009
```

In this workshop you will create your own deployment. It contains several pods that will run an application. The application will look like this:

&nbsp;

![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_1_star.png)

&nbsp;

&nbsp;

In this workshop we make use of the Google Cloud Shell. This shell provides all the tools that are necessary to manage Kubernetes. In the shell we will use the utility kubectl. 

On the cheat sheet you can find the useful commands that you can use during the workshop.

## Create a namespace
Now that we are into the cluster we need to create a namespace, but first let's see what namespaces are available.

```bash
kubectl get namespace
```
```bash
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
kube-public   Active    1d
```

You probably see something similar to above. Those are the default namespaces that are always there. By default if you don't specify a namespace, all your deployments and changes will be done in the default namespace. 
&nbsp;

Now lets create your own namespace. Use the command below and change the name in the example to your own name or favorite alias.

```bash
kubectl create namespace <insert-name>
```
**Note** Keep a note of your namespace as we will use it throughout the exercise, if you forget what you set, you can always use: 

```bash
kubectl get namespaces
```

Most commands are easy to use in Kubernetes, switching between namespaces is a different story. Copy and execute the command below to switch to your own namespace. Don't forget to switch the namespace name to fit the one you used earlier.

The actions which follow will now only execute within this namespace. Deployments in other namespaces will not be visible. Now lets validate the namespace change with the second command.

```bash
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name>
# Validate it 
kubectl config view | grep namespace:
```
 
&nbsp;
## Create a deployment
Now that we are in the right namespace we will deploy our replicaset. A replicaset is part of the manifest that tells kubernetes how many copies of your pods are desired.

```bash
kubectl apply -f https://raw.githubusercontent.com/Parsifal-M/KubernetesForEveryone/master/Training/k8s_deployment.yaml
````
We have just deployed the following deployment script to our namespace. It's a deployment with 3 pods. 
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubern8sdemo
  labels:
    app: kubern8sdemo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubern8sdemo
  template:
    metadata:
      labels:
        app: kubern8sdemo
    spec:
      containers:
      - name: kubern8sdemo
        image: mvdmeij/k8sdemo:v1
        ports:
        - containerPort: 80
```

To make sure the deployment went correctly we are going to check the status:

```bash
kubectl get deployments
```
If you want to delete the deployment you can use the following command: 

```bash
kubectl delete deployment <deployment_name> 
```

If you didn't delete the deployment, the status should look similar to below. It shows the amounts of pods created during this deployment. As you can see you have a desired amount of 3 pods and the current amount of pods is also 3. 

```bash
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubern8sdemo   3         3         3            3           43s
```
&nbsp;

## Pods
Let's now take a closer look at the pods. 
```bash
kubectl get pods 
```

```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-55vjr   1/1       Running   0          1m
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          1m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          1m
```
As you can see there are three pods created each running with their own name. What would happen if we would delete one pod? Use the command below and change the name of the pod to yours. 

```bash
kubectl delete pod kubern8sdemo-68fcf74b6-55vjr
```

Okay we have now deleted the pod. Let's take a look at the status of the pods.

```bash
kubectl get pods 
```

```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          2m
kubern8sdemo-68fcf74b6-s4j2h   1/1       Running   0          24s
```
In this example you can see there are still 3 pods running, how is that possible? We have deleted a pod, but we have stated that the desired amount of the pods should be 3. Kubernetes has automatically created a new pod. Above you can see a new pod has been created 24s ago while the other 2 were created 2m ago. The kubernetes master node will always make sure to keep as many available as is desired.

&nbsp;

So how do we reduce the amounts of pods? We will have to scale the replicaset. We are going to scale down the deployment to 2 pods. This will be a permanent change to the deployment, unless we later adjust the desired amount of pods again.

```bash
kubectl scale deployments kubern8sdemo --replicas=2 
```
Now let's check the amount of pods again.

```bash
kubectl get pods -n <your-namespace>
```
```bash
NAME                           READY     STATUS    RESTARTS   AGE
kubern8sdemo-68fcf74b6-h8ncl   1/1       Running   0          7m
kubern8sdemo-68fcf74b6-qddsp   1/1       Running   0          7m
```
As you can see the amount of pods has been reduced. Let's check the deployment to see the desired amount of pods.
```bash
kubectl get deployments 
```
```bash
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubern8sdemo   2         2         2            2           10m
```
The desired amount is now 2. 

&nbsp;

## Create a Service

Create a service
We have our application ready, let's make a start to expose it to the outside world. In order to do that we need to create a service. This service will be used to link to the deployment with the use of the selectors and labels. Create the service:

Let's apply the service:

```bash
kubectl create -f https://raw.githubusercontent.com/Parsifal-M/KubernetesForEveryone/master/Training/k8s_service.yaml 
```

To check the status of the service, use the command below.

```bash
kubectl get service -n <your-namespace>
NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubern8sservice   LoadBalancer   10.108.57.90   <Pending>        80:32227/TCP   18m
```

Something similar should appear. The service has created and can be reachable within the kubernetes space by using the 'kubern8sservice' and will soon be available via the EXTERNAL-IP which will populate within 1-2 minutes!

## Update Pods

Our developers have worked hard to create a new version of our application. Let's update the deployment! Remember the deployment?

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubern8sdemo
  labels:
    app: kubern8sdemo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubern8sdemo
  template:
    metadata:
      labels:
        app: kubern8sdemo
    spec:
      containers:
      - name: kubern8sdemo
        image: mvdmeij/k8sdemo:v1
        ports:
        - containerPort: 80
```

As you can see our old deployment uses a docker image with tag v1. The developers recently finished version 5. With a single command we can update the version of our deployed pods to that version. If you want to see how quick the application is updated, make sure to keep an eye on your browser. Before proceeding to the next step, wait till the application has been updated in your browser.

```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v5 
```
![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_5_stars.png)

&nbsp;

It seems that version 5 was a bit too much for our users. Lets roll back a few versions and see what kubernetes does under the hood. If you want to watch live changes to the pods include -w as shown below. Keep an eye on the terminal while executing the version change.

Before you do this, click the **+ symbol**  (on the left hand side) on the Google Cloud Shell to open a new tab, then enter the below:

```bash
kubectl get pods -w 
```

Keep **both** tabs open, swap back to your **first** (original) tab and enter the below, then swap back to the **new tab**.

```bash
kubectl set image deployment/kubern8sdemo kubern8sdemo=mvdmeij/k8sdemo:v3 
```

While watching the live changes, you should see something similar to this:

```bash
NAME                           READY     STATUS              RESTARTS   AGE
kubern8sdemo-7b4db456b-j59sd   1/1       Running             0          2m
kubern8sdemo-7b4db456b-npk75   1/1       Running             0          2m
kubern8sdemo-84ff7fd64-w4ksc   0/1       ContainerCreating   0          2s
kubern8sdemo-84ff7fd64-w4ksc   1/1       Running             0          3s
kubern8sdemo-7b4db456b-j59sd   1/1       Terminating         0          2m
kubern8sdemo-84ff7fd64-m4rzh   0/1       Pending             0          0s
kubern8sdemo-84ff7fd64-m4rzh   0/1       Pending             0          0s
kubern8sdemo-84ff7fd64-m4rzh   0/1       ContainerCreating   0          0s
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-j59sd   0/1       Terminating         0          2m
kubern8sdemo-84ff7fd64-m4rzh   1/1       Running             0          4s
kubern8sdemo-7b4db456b-npk75   1/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
kubern8sdemo-7b4db456b-npk75   0/1       Terminating         0          2m
```

Kubernetes is creating a new container before terminating the old one until our desired amount is reached. Use control/cmd c to stop it the pod watch.
&nbsp;

Below you can see how your application should now look in the browser:

![Application](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/App_3_stars.png)

&nbsp;

## Clean up!
Clean up your deployment and your service before you go to the next task.

&nbsp;

## Do It Yourself

![Super Mario](https://github.com/Wesbest/KubernetesForEveryone/blob/master/Pictures/SuperMario.png)

Try to get Super Mario running in your browser.

**Tips**

Image source: https://hub.docker.com/r/pengbai/docker-supermario/ 

Don't reinvent the wheel! Use our templates: https://github.com/smii/KubernetesForEveryone/tree/master/Templates

Use your favorite editor but the real masters may also use vim ofcourse!

&nbsp;


Requirements: &nbsp;

Create a deployment with **2 pods**. &nbsp;

Create a services with type **LoadBalancer** &nbsp;

Note: Use port **8080** in your **deployment** and **service** yaml(s).

&nbsp;

## The End
Well that's about it. Raise your hand if you have questions!
