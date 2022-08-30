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
kubectl create deployment <deploymentName> --image=<DockerHubRepositoryUser>/<imageName>
```

> To delete deloyment, use `kubectl delete <deploymentName>` command.
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


