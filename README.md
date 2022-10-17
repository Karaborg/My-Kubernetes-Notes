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
      port: <targettedPort>
      targetPort: <thePortAppUsing>
    #- protocol: 'TCP'                      # can be add more ports like this
    #  port: <targettedPort>
    #  targetPort: <thePortAppUsing>
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

## Updating & Deleting Resources
To make changes in our deployment, the only thing you need to do is, changing the `yaml` files as you want and then just use the `kubectl app -f=<yamlFileName/Path>` command.

You can also delete the resources based on a file with the command below:
```
kubectl delete -f=<yamlFileName/Path>
```

> The delete command above will not delete the yaml file itself. It will just delete the resources **based on** that file.

> You can also merge 2 yaml files to 1 yaml file. Just makse sure you seperated them with a `---` (3 dashes) as shown below:
```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: second-app
  ports:
    - protocol: 'TCP'
      port: 80
      targetPort: 8080
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: second-app-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template: 
    metadata:
      labels:
        app: second-app
        tier: backend
    spec: 
      containers:
        - name: second-node
          image: karaborg/kub-first-app:3
```

## Managing Data & Volumes
Just like in Docker, sometimes we will need to store data. However, managing data is not exactly the same as in Docker. In Kubernetes, data might be stored in the container, pods or even in the server. And that differs because you might restart the container or pods or even the server.

The biggest difference in volumes between Docker and Kubernetes is, Docker has anonymous volumes, named volumes and bind mounts. But, Kubernetes has tons of different volumes. You will not be using all of them. They are specified for specific use cases, but we will talk about 2 of them.

To get more information about Volumes, you can check out [Types of Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#volume-types) document.

### emptyDir
An emptyDir volume is first created when a Pod is assigned to a node and exists as long as that Pod is running on that node. As the name says, the emptyDir volume is initially empty. All containers in the Pod can read and write the same files in the emptyDir volume, though that volume can be mounted at the same or different paths in each container. When a Pod is removed from a node for any reason, the data in the emptyDir is deleted permanently.

> A container crashing does not remove a Pod from a node. The data in an emptyDir volume is safe across container crashes.

emptyDir configuration example (deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: karaborg/kub-data-demo:1
          volumeMounts:                           # clerify the volume to container
            - mountPath: /app/story               # to give the exect path to set as a volume
              name: story-volume                  # name of the volume
      volumes:                                    # add volume in the same level as container
        - name: story-volume                      # give a name for your volume
          emptyDir: {}                            # give the type of volume
```

> Since this type of volume closely attached to pods, it will not work as the same when we use multi replicas.

### hostPath 
This allows us to set a path on the host machine, so on the node, the real machine running this pod, and then the data from that path will be exposed to the different pods. So multiple pods can now share one in the same path on the host machine.

hostPath configuration example (deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: karaborg/kub-data-demo:1
          volumeMounts:                           # clerify the volume to container
            - mountPath: /app/story               # to give the exect path to set as a volume
              name: story-volume                  # name of the volume
      volumes:                                    # add volume in the same level as container
        - name: story-volume                      # give a name for your volume
          hostPath:                               # give the type of volume
            path: /data                           # set a path on host machine
            type: DirectoryOrCreate               # this will create the path if it's not created yet. Can be used 'Directory' if created
```

> Since this type of volume closely attached to nodes, it will not work as the same when we use multi nodes.

## Persistent Volumes
As we mentioned in the **emptyDir** and **hostPath** volumes, the volumes we defined are not persistent. emptyDir is attached to pods. That means we will not be able to use the same volume on different pods. And hostPath is attached to nodes. And that means we will not be able to use that volume on different nodes.

So persistent volumes comes here. These types of volumes are not attached to either pods or nodes. So we will not have to lose our data when we restart our pods or nodes. 

> emptyDir volume logic: Cluster > Node > Pod (indludes volume)

> hostPath volume logic: Cluster > Node (indludes volume) > Pod

> Persistent volume logic: Cluster (indludes volume) > Node > Pod

To get more information about Persistent Volumes, you can check out [Types of Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) document.

Now, we will create a yaml file named `host-pv.yaml`. And we will give the specifics of our persistent volume there as shown below:
```
apiVersion: v1                     
kind: PersistentVolume                          # must, to specify the "kind" we wil be using
metadata:                                       # to give a name
  name: host-pv
spec: 
  capacity:                                     # to control capacity
    storage: 1Gi                                # 1 gigabyte
  volumeMode: Filesystem                        # storage types, can be Filesystem or Block
  storageClassName: standard                    # to use Kubernetes stardad storage class
  accessModes:                                  # access type
    - ReadWriteOnce                             # can be mounted as a read-write volume by a single node
    #- ReadOnlyMany                             # read-only but it can be claimed by multiple nodes
    #- ReadWriteMany                            # read-write but it can be claimed by multiple nodes
  hostPath:                                     # type of volume
    path: /data                                 # path on host machine
    type: DirectoryOrCreate                     # to create the path if not exist
```

> To see Kubernetes storage class, use the command `kubectl get sc`

Once we are done with our persistent volume file, we also need to clarify the volume on our pods. To do so, we will create another yaml file named `host-pvc.yaml` as shown below:
```
apiVersion: v1
kind: PersistentVolumeClaim                     # must, to specify the "kind" we wil be using
metadata:
  name: host-pvc                                # to give a name
spec:
  volumeName: host-pv                           # to claim the volume
  accessModes:                                  # access type
    - ReadWriteOnce
  resources:                                    # request storage for this volume
    requests:
      storage: 1Gi                              # 1 gigabyte
```

After claiming the volume, we need to establish a connection between our pod and persistent volume. So, we need to go back to our `deployment.yaml` file. And now we need to connect our claim to our pod as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: karaborg/kub-data-demo:1
          volumeMounts:
            - mountPath: /app/story
              name: story-volume
      volumes:
        - name: story-volume
          persistentVolumeClaim:              # volume type
            claimName: host-pvc               # to specify which claim we want to use
```

Moreover, we neet to apply the changes with the commands below:
```
kubectl apply -f=host-pv.yaml
kubectl apply -f=host-pvc.yaml
kubectl apply -f=deployment.yaml
```

> To get a list of all persistent volumes, use the command `kubectl get pv`

> To get a list of all claims, use the command `kubectl get pvs`

## Environment Variables
You can always set environment variables for Kubernetes. To do so, you will need to modify your `deployment.yaml` file as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: karaborg/kub-data-demo:2
          env:                                   # 
            - name: STORY_FOLDER                 # variable name
              value: 'story'                     # variable value
          volumeMounts:
            - mountPath: /app/story
              name: story-volume
      volumes:
        - name: story-volume
          persistentVolumeClaim:
            claimName: host-pvc
```

But, to work more efficiently, we do not think that is the way you want to use environment variables. So, create a file for environments. In this case, we will create a file named `environment.yaml` as shown below:
```
apiVersion: v1
kind: ConfigMap                                 # type
metadata:
  name: data-store-env                          # name
data:
  folder: 'story'                               # key and it's value
```

After that we need to apply the yaml file with the command below:
```
kubectl apply -f=environment.yaml
```

> To get a list of config maps, `kubectl get configmap`

Once it is done, now we need to use the variable in our `deployment.yaml` file as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: karaborg/kub-data-demo:2
          env:
            - name: STORY_FOLDER
              valueFrom:                          # 
                configMapKeyRef:                  #
                  name: data-store-env            # the name we specified in our environment.yaml file
                  key: folder                     # the key we specified for our variable
          volumeMounts:
            - mountPath: /app/story
              name: story-volume
      volumes:
        - name: story-volume
          persistentVolumeClaim:
            claimName: host-pvc
```

And lastly, we apply the `deployment.yaml` file as shown below:
```
kubectl apply -f=deployment.yaml
```

## Kubernetes Networking
Let's say we have a multi-container application. One for **User API**, one for **Auth API** and another one for **Task API**. The Client can access **Users API** and **Task APIs** but not **Auth API**. In the meantime, We want **User API** and **Auth API** to talk to each other.

One of the ways of doing that is to put both containers inside the same Pod. And we call the connection as `Pod-Internal`.

> Cluster( Pod(Auth API <--Pod-Internal-- User API) Pod(Task API) )

To do so; we create a `users-deployment.yaml` file as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
        - name: users
          image: karaborg/kub-demo-users:latest
```

And also a `users-service.yaml` file:
```
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

Now we have a running Users API, we can now build our Auth API too. To do that, we will not create another `yaml` file. Instead, we will add our Auth API container inside of our `users-deployment.yaml` file as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
        - name: users
          image: karaborg/kub-demo-users:latest
          env:
            - name: AUTH_ADDRESS
              value: localhost
        - name: auth
          image: karaborg/kub-demo-auth:latest
```

With that done, when we apply our `yaml` file, Kubernetes will create both containers inside of just one pod. And since they are in the same pod. They can access each other with `localhost` URL.

But what if we want to connect from **another pod**?

> Cluster( Pod(Users API) --Cluster-Internal--> Pod(Auth API) <--Cluster-Internal-- Pod(Task API) )

Let's start with creating our `auth-deployment.yaml` file similar to **users-deployment.yaml** file as shown below:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: karaborg/kub-demo-auth:latest
```

Moreover, we will also need a `auth-service.yaml` file:
```
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

> We used `LoadBalancer` type of service so far. But since we don't want our Auth API to accessed by outside, and LoadBalancer allows that, we decide to use `ClusterIP`. Which doesn't allow access from outside.

Since now, we have our separated container for Auth API, we also need to remove our old Auth API container from `users-deployment.yaml` file:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
        - name: users
          image: karaborg/kub-demo-users:latest
          env:
            - name: AUTH_ADDRESS
              value: 10.105.86.144
```

> When we had our both containers in the same pod, we could use `localhost` to make a connection through URL links. But, since we decided to use `ClusterIP` and separate pods, we need to use the URL Auth API service uses.

> To get a list of services; use the command `kubectl get services`.

With that in mind, Kubernetes also automatically generates a value of service IP's. So we don't have to manually check the `ClusterIP` IP's. To get the automatically generated service IP, we can use `<SERVICE_NAME>_SERVICE_HOST` as an **environment variable**.

> In the case above, the Auth API service IP which named as `auth-service` can be used as `AUTH_SERVICE_SERVICE_HOST`.