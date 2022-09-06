# Welcome
This is me taking notes on GitHub while learning `Kubernetes`. I will probably be going to need this document when I forget how docker works. If you find something useful, be my guest. I will try to update this whenever I study.

## Intro
Before we start, we need to clarify a couple of things. To work with Kubernetes, we first need a `Cluster`, which we will run our Kubernetes. This **Cluster** will have a `Master Node`, and one or more `Worker Node/s`. This is where we will deploy our application and run it. To interact with the Kubernetes cluster, we will need `kubectl` tool.

To give a metaphor: imagine the `Cluster` as an **army**. Army has a **general****, which we call `Master Node`. The general commands to soldiers, which are `Worker Nodes`. And general is following the commands of the president, which is `kubectl`.

## Setup
Normally, if we were using a cloud cluster, we only had to install `kubectl` to our local but, since we are learning and want to use Kubernetes in our local, we will going to step a cluster on our local with a tool called `Minikube`. Which will create a virtual machine and start a cluster with a single node.

> `Minikube` does not replace `kubectl`. You need to install **kubectl**.

### Kubectl
You can install `kubectl` installation by following [Kubernetes Documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-on-windows-using-chocolatey-or-scoop) here.
> Would suggest you to go along with `Chocolatey`.

### Minikube
You can install `minikube` installation by following [Minikube Documentation](https://minikube.sigs.k8s.io/docs/start/) here.
> Would suggest you to go along with `Chocolatey`

> To start minikube, you can use `VirtualBox` driver, but I will use `Docker`. For more info; check out [drivers](https://minikube.sigs.k8s.io/docs/drivers/) page.

> If you want to delete a failed minikube cluster; you can delete it with `minikube delete` command.

And with that command, minikube will start your `Cluster` with `Master Node` and `Worker Node`.

After the setup, you can check your minikube status with `minikube status` command. And also check your dashboard with `minikube dashboard` command which will open the dashboard on your default browser automatically.

## A First Deployment (Imperative Approach)
To start, we will choose a basic web application. Nothing extraordinary. The only thing to mention is that; it is a Node application, it uses port **80** and it still has a **dockerfile**. Kubernetes still need images. The difference is Kubernetes will take care of deployment for us.

Dockerfile Example:
```
FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 8080

CMD [ "node", "app.js" ]
```

As the first step, we will build our image with the dockerfile:
```
docker build -t <imageName> .
```

After building the image, it is time for the deployment. To do so; we first need to upload our image to DockerHub because as Cluster will act like it is a whole different machine, it cannot pull our local image. So, after uploading the image, we will use the command below:
```
kubectl create deployment <deploymentName> --image=<DockerHubRepositoryName>/<imageName>
```

> To delete service, use `kubectl delete service <serviceName>`

> To delete deloyment, use `kubectl delete deployment <deploymentName>` command.

> To check deployments, use `kubectl get deployments` command.

> To check pods, use `kubectl get pods` command.

After the first deployment, I would suggest you to check the **deployments** to check if the application is running. If not, check the **pods** to see the issue. If everything is looking good, you can now check the Kubernetes dashboard with `minikube dashboard` command and see details about your cluster's status, deployments and pods.

While you are on the minikube dashboard, you can find an IP address under your pods. But, that IP is for local access itself. So we cannot use that IP to open our web application just yet. That IP will also change time-to-time so it is also not reliable.

To expose the port, we will use the command `kubectl expose ...` as shown below:
```
kubectl expose deployment <deploymentName> --type=LoadBalancer --port=<exposedPort>
```

> The `type` is `ClusterIP` as the **default** which means it is only reachable from inside of the cluster but, there are other types. Such as `NodePort`, which should expose the IP address of the worker nodes, which would make the application accessable from the outside. But even better than that, `LoadBalancer`, which will not only create a unique address but also evenly shares the traffic.

> To check `Services`, use `kubectl get services` command. And if you notice, there will be a pending `EXTERNL-IP`. That will be always pending for our local clusters but if we had a cloud provider, Kubernetes will give an external IP for us to connect too. And since we are not, we need to map a port with a `minikube service <deplomentName>` command, which will open the applcation on our browser.

> If you are working on a cloud provider, you do not have to do anything.

## Restarting Containers
Assuming our web application is running with Kubernetes. We built our image, pushed the image to DockerHub and created our deployment. But what if our pods crash? Well, it gets restarted. You can check how many times your pods restarted with the `kubectl get pods` command. And also you can see the restart count from the **Kubernetes Dashboard**

> If your container has an build error, doesn't build right, Kuberneters will not restart if it's doing constantly.

## Scaling
We can also scale up and down the pods we have. We could scale up to 3 times, so that would open 2 more pods with the running pod and with the help of `Load Balancer`, the traffic would be shared between those 3 pods. And even if one or two of the pods would crash, Load Balancer would direct the traffic to the running pod.

To scale deployments:
```
kubectl scale deployment/<deploymentName> --replicas=<desiredNumberOfPods>
```

After scaling, you can check the deployments and pods.

## Updating Deployments
To update, you first need to update your image on DockerHub. But there is an important note here. Kubernetes will not update the deployment **if it has the same image tag**, so it is `very important to define tags` now! So, after pushing the newly built image **with a tag**, you can update the deployment with the command below:
```
kubectl set image deployment/<deploymentName> <containerName>=<DockerHubRepositoryName>/<imageName>:<imageTag>
```

> If you do not know the `container name`, you can find the name under pods in **Kubernetes Dashboard**.

> You may not see the difference imidietly, give it some time to re-deploy the deployment.

## Deployment Rollbacks
Let's say you had a typo while updating the deployment. You wanted to give a specific tag but, there was a typo. Don't worry, Kubernetes will not shut down the running pods until the new pods are up and running, but the updating process will be stuck in a loop for a while. It will try to pull the non-existing image and fail at the deployment state.

The first thing you want to do is, check the pods with a `kubectl get pods` command. Check the status if it is **ImagePullBackOff**.

And if it is, you can now rollback the `latest` update with the command below:
```
kubectl rollout undo deployment/<deploymentName>
```

If you also want to rollback to an older deployment, first, see the `deployment history` with the command below:
```
kubectl rollout history deployment/<deploymentName>
```

And also see detailed `information about a specific older deployment`, use the command below:
```
kubectl rollout history deployment/<deploymentName> --revision=<revisionNumber>
```

> You can find the `revision number` with the **rollout history list** command.

Then, you can `rollback to an older deployment` with the command below:
```
kubectl rollout undo deployment/<deploymentName> --to-revision=<revisionNumber>
```

## The Imperative vs The Declarative Approach
We have now learned how to start our application with Kubernetes (with an imperative approach). But with that approach, we were still using a bunch of individual commands to trigger certain Kubernetes actions. 

Now we will look at the declarative approach, in which we will create a `yaml` file and define our changes and desired states. And Kubernetes will use this file.

> Imperative approach is more like `docker run ...`, and declarative is more like `docker-compose`.

Therefore; we will start with creating our `yaml` file. 

> The name can be anything, just make sure, it is a yaml file.

Example:
```
apiVersion: apps/v1                         # must, current verison can be found by simply searching "kubernetes deployment yaml"
kind: Deployment                            # must, to specify the "kind" we wil be using
metadata:                                   # name of the deployment, not some actual data
  name: <deploymentName>
spec:                                       # specification details 
  replicas: 1                               # not a must, default is "1"
  selector:                                 # to make sure the "template" is controlled by the "Deployment". Should mention both app and tier
    matchLabels:
        app: <appLabel>
        tier: <tierLabel>
  template:                                 # to define "pods"
    metadata:
      labels:
        app: <appLabel>                     # to label it with a "name"
        tier: <tierLabel>                   # to label it with a "tier". Sust like right above, both works, separately or together
    spec:                                   # to define a specific pod specifications
      containers:
        - name: <containerName>
          image: <DockerHubRepositoryName>/<imageName>:<imageTag>
        #- name: ...
        #  image: ...
```

> To get more information about the tags, you can use [API Reference for Kubernetes](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#deployment-v1-apps) documentation. Just make sure you are looking for **the right version**.

After creating our yaml file, we can now start our deployment with the command below:
```
kubectl apply -f=<yamlFileName/Path>
```

> More yaml file can be added with another `-f` tag.

Now our application is running. But we cannot reach it yet. We need to expose ports. And since we are doing the declarative approach, we can also create a `yaml` file for the `Service`.

> The name of the yaml file is again, does not matter.

Example:
```
apiVersion: v1                              # must, it is the version
kind: Service                               # must, to specify the "kind" we wil be using
metadata:                                   # name of the service, not some actual data
  name: <serviceName>
spec:                                       # specification details
  selector:                                 # to select target resorces to controlled by this service
    app: <appLabel>                         # "app" or/and "tier" does not matter. Both works, separately or together
  ports:
    - protocol: 'TCP'                       # default is "TCP"
      port: <thePortAppUsing>
      targetPort: <targettedPort>
    #- protocol: 'TCP'                      # can be add more ports like this
    #  port: <thePortAppUsing>
    #  targetPort: <targettedPort>
  type: LoadBalancer                        # can be LoadBalancer / ClusterIP / NodePort
```

After that, we will use the `apply` command just like we used for the deployment yaml file:
```
kubectl apply -f=<yamlFileName/Path>
```

And after that, we can see the URL for our application with the command below:
```
minikube service <serviceName>
```