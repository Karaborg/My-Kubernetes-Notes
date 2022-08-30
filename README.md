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
