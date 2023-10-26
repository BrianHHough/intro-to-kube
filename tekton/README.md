# Tekton - CI/CD for K8s

## Note

You can also follow along with the info through the Uplimit Course [here](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/module/project-3-instructions#corise_clm9w276p000j3b7lvx4bov7v).

## Background

StartupCo is thinking of adding CI/CD in place to help with the automatic build, push, and deployment of its we applications to Kubernetes. 

As a result, they looked to Tekon .
🛠️. Tekton is a powerful Kubernetes-native framework for building continuous integration and continuous delivery (CI/CD) pipelines. Creating a simple project to automate a basic CI/CD pipeline using Tekton in a Minikube environment is a great way to learn and practice its usage. 

The goal for this project is to automate the process of building, pushing an Docker image to a container reigstry and deploying said image using Tekton on a local Minikube cluster 🐳.



## Setup

Start up your `minikube` environment and install necessary dependencies by running:

```bash
cd tekton
minikube start
bash setup.sh
```

Monitor the installation by calling and make sure that `tekton-pipelines-controller` and `tekton-pipelines-webhook` and `READY`:

```bash
kubectl get pods --namespace tekton-pipelines --watch

NAME                                           READY   STATUS              RESTARTS   AGE
tekton-pipelines-controller-6d989cc968-j57cs   0/1     Pending             0          3s
tekton-pipelines-webhook-69744499d9-t58s5      0/1     ContainerCreating   0          3s
tekton-pipelines-controller-6d989cc968-j57cs   0/1     ContainerCreating   0          3s
tekton-pipelines-controller-6d989cc968-j57cs   0/1     Running             0          5s
tekton-pipelines-webhook-69744499d9-t58s5      0/1     Running             0          6s
tekton-pipelines-controller-6d989cc968-j57cs   1/1     Running             0          10s
tekton-pipelines-webhook-69744499d9-t58s5      1/1     Running             0          20s
```

It took awhile to print, but these were what logged in total:
```bash
NAME                                           READY   STATUS              RESTARTS   AGE
tekton-events-controller-5994bb6b74-fm95v      0/1     ContainerCreating   0          5m55s
tekton-pipelines-controller-6b984488dc-mwl49   0/1     ContainerCreating   0          5m55s
tekton-pipelines-webhook-d55cc5fb7-5dmf5       0/1     ContainerCreating   0          5m47s
tekton-pipelines-webhook-d55cc5fb7-5dmf5       0/1     Running             0          5m50s
tekton-events-controller-5994bb6b74-fm95v      0/1     Running             0          6m2s
tekton-pipelines-controller-6b984488dc-mwl49   0/1     Running             0          6m5s
tekton-pipelines-webhook-d55cc5fb7-5dmf5       1/1     Running             0          5m57s
tekton-events-controller-5994bb6b74-fm95v      1/1     Running             0          6m16s
tekton-pipelines-controller-6b984488dc-mwl49   1/1     Running             0          6m16s
tekton-pipelines-controller-6b984488dc-mwl49   1/1     Running             0          8m42s
tekton-pipelines-controller-6b984488dc-mwl49   0/1     Running             1 (3s ago)   8m46s
tekton-pipelines-controller-6b984488dc-mwl49   1/1     Running             1 (13s ago)   8m56s
...
```

## Timeouts for Minikube
I had to run `minikube stop` and it ran for awhile without stating how it was working or stoping:
```
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
```

Also, my `minikube start` got stuck on this:
```
😄  minikube v1.31.2 on Ubuntu 20.04 (docker/amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
``` 

To fix this, I'll check if minikube is running:
```bash
docker ps -a | grep minikube
```

This logged out:
```bash
d59e95bfe88f   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   3 weeks ago   Exited (130) 3 minutes ago             minikube
```

To make sure we stop minikube, I ran this:
```bash
docker stop minikube
```

This printed out: `minikube`

I was then able to run `minikube start` without any problem and saw this:
```bash
@BrianHHough ➜ /workspaces/intro-to-kube/tekton (calico-brianhhough) $ minikube start
😄  minikube v1.31.2 on Ubuntu 20.04 (docker/amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
🔗  Configuring Calico (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## 🐞 Debugging: `Unable to connect to the server: net/http: TLS handshake timeout`

I started experiencing more issues with `minikube` which I suspect was due to multiple containers running, being updated, etc. across different projects. 

The error wazs noticed after running `minikube start` and then trying to run `bash setup.sh` where I saw this:
```bash
Unable to connect to the server: net/http: TLS handshake timeout
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 28.5M  100 28.5M    0     0  26.4M      0  0:00:01  0:00:01 --:--:-- 37.0M
(Reading database ... 70203 files and directories currently installed.)
Preparing to unpack .../tektoncd-cli-0.32.0_Linux-64bit.deb ...
Unpacking cli (0.32.0) over (0.32.0) ...
Setting up cli (0.32.0) ...
Unable to connect to the server: net/http: TLS handshake timeout
Unable to connect to the server: net/http: TLS handshake timeout
```

I thought to check the status of minikube with `minikube status`, which showed this:
```bash
minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Stopped
kubeconfig: Configured
```

Then I tried to run this: `kubectl get pods --namespace tekton-pipelines --watch` to get the pods and saw this timeout error:

```bash
E1026 06:29:02.899783   15844 memcache.go:265] couldn't get current server API group list: Get "https://192.168.49.2:8443/api?timeout=32s": net/http: TLS handshake timeout
E1026 06:29:12.900838   15844 memcache.go:265] couldn't get current server API group list: Get "https://192.168.49.2:8443/api?timeout=32s": net/http: TLS handshake timeout
E1026 06:29:22.905306   15844 memcache.go:265] couldn't get current server API group list: Get "https://192.168.49.2:8443/api?timeout=32s": net/http: TLS handshake timeout
E1026 06:29:32.907879   15844 memcache.go:265] couldn't get current server API group list: Get "https://192.168.49.2:8443/api?timeout=32s": net/http: TLS handshake timeout
E1026 06:29:42.909736   15844 memcache.go:265] couldn't get current server API group list: Get "https://192.168.49.2:8443/api?timeout=32s": net/http: TLS handshake timeout
```

I then tried to stop minikube with `minikube stop` and then I got a ton of errors about the `docker daemon`:
```bash
  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
E1026 06:31:22.062297   16597 kic.go:469] unable to kill containers : killing containers docker: docker rm -f 8556c1f01bc6 6fe598a2cf31 4f022baf061e f95f2a9234e1 3da4339d5f1d 309bebb333db a01924ff4060 6df37a7fb3da 048bfb24c8f7 772e8ec32d24 1727ace42e1b c1740642b088 f1fec8583f1e f06f9617236b f4a46f9fb943 55fdc9d80951 3e2fa551f727 ab8ba9ce89dc 8487b44b78f8 63ca6a9e4f54 107fed89903e 37a4ae742ecb 867d33c521f0 46151ca97663 3d96e4e5deae 855d4661ffc4 fdc96fb008d9 903153f77832 8b30e0667037 d62da1e7abed 30e5623deb15 4b8a2120c0a8 28c1db347c78 ac1f9ac57e24 24eee09a97c0 ef1cbd587a01 e7c1c1c206bb 53506162e270 bbf877517421: exit status 1
stdout:

stderr:
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running
...
🛑  1 node stopped.
```

This clearly wasn't where it was at, so I figured, why not just delete minikube and start completely from scratch.

I ran `minikube delete` which printed the below:
```bash
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /home/codespace/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

Then I started `minikube start` once more and saw this clean output:
```bash
😄  minikube v1.31.2 on Ubuntu 20.04 (docker/amd64)
✨  Automatically selected the docker driver. Other choices: ssh, none
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Now, when I checked `minikube status`, I saw this:
```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

I ran the `bash setup.sh` command, which logged all the creation events, and then I ran `kubectl get pods --namespace tekton-pipelines --watch` and I see the output as expected:
```bash
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-events-controller-5994bb6b74-92zxx      1/1     Running   0          2m52s
tekton-pipelines-controller-6b984488dc-tcr4r   1/1     Running   0          2m52s
tekton-pipelines-webhook-d55cc5fb7-976mv       1/1     Running   0          2m51s
```

Finally we are back in business!! 🥳


## Testing

We have configured [GitHub Actions](https://github.com/features/actions) with minikube! GitHub Actions works as a CI/CD tool but for our intents and purposes, we will use it to create an autograder for your `yml` file changes. To test, create a branch in our `intro-to-kube` repo prefixed with
`tekton-*` and push changes up to that branch. [See the output of your results here for reference](https://github.com/abanuelo/intro-to-kube/actions/workflows/tekton.yml).

```bash
# create your branch
git checkout -b tekton-<github-username>
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin tekton-<github-username>
```

## Main Project

We will write the `build.yml`, `deploy.yml`, and `build-push-deploy-pipeline.yml` that will work to orchestrate the automatic release of updates to our web application at StartupCo. 

0. To get started we will be building up a [Task](https://tekton.dev/docs/getting-started/tasks/) to build and run test suites for our web application. A Task defines a series of Steps that run sequentially to perform logic that the Task requires. Every Task runs as a pod on the Kubernetes cluster, with each step running in its own container. Please complete the `tasks/build.yml` file by adding the following Steps to your Task:

    a. Clone the github repo from the params into a directory called: `/workspace/source` inside the Pod

    b. Make `/workspace/source` your workDir and run `npm install`. 

    c. Make `/workspace/source` your workDir and run `npm test`

    Once you are ready to test run `kubectl apply -f ./tasks/build.yml` and you can see its creation status by running `tkn task list`. You can also push up to a branch prefixed with `tekton-` to run automated tests against your changes.

I defined a Tekton task named `build` and applied it to my Kubernetes cluster with the below:
```bash
# Run the command to create the build task
kubectl apply -f ./tasks/build.yml

# Output:
task.tekton.dev/build created
```

```bash
# Run the command to check the status of the build task:
tkn task list

# Output:
NAME    DESCRIPTION   AGE
build                 5 seconds ago
```



1. Run the `./tasks/push.yml` file as it just contains boiler plate code that will be needed later: `kubectl apply -f ./tasks/push.yml`.

The output of running this was:
```bash
task.tekton.dev/push created
```


1. Next up we will be writing the deploy Task. A couple of things to note, since minikube does not out-of-the-box support pulling pushing/pulling images from a container registry, the `push.yml` current mocks out the push of an image to Docker's container registry. In the [challenge](#challenge) section of this project, you can revisit our mocked code to get minikube to push up images of your web application to a container registry. 

    For now, the `tasks/deploy.yml` file assumes that we have pushed up `nginx:latest` image that we then update for the `k8s/nginx-deployment.yml`.  Please complete the `tasks/deploy.yml` file by adding the following Steps to your Task:


    a. Clone the repo specified by `REPO_URL` in `/workspace/source/`

    b. Run  `kubectl apply -f /workspace/source/tekton/k8s/nginx-deployment.yml`

    c. Run: `kubectl set image deployment/nginx-deployment nginx=<IMAGE-NAME> -n=<NAMESPACE>` where `IMAGE_NAME` and `NAMESPACE` are passed as parameters to our Task.

    Once you are ready to test run `kubectl apply -f ./tasks/deploy.yml` and you can see its creation status by running `tkn task list`.

The output of the `kubectl apply -f ./tasks/deploy.yml` command was:
```bash
task.tekton.dev/deploy created
```

Running `tkn task list` showed me the list of tasks:
```bash
NAME     DESCRIPTION   AGE
build                  16 minutes ago
deploy                 40 seconds ago
push                   7 minutes ago
```


2. Let's add all the tasks together in a [Pipeline](https://tekton.dev/docs/getting-started/pipelines/). A Pipeline defines an ordered series of Tasks arranged in a specific execution order as part of the CI/CD workflow. Work to complete the `build-push-pipeline.yml` file by sequencing the Tasks as the file name suggests: build &rarr; push &rarr; deploy. Run `kubectl apply -f build-push-deploy-pipeline.yml` when finished.

The output of the `kubectl apply -f build-push-deploy-pipeline.yml` command was:
```bash
pipeline.tekton.dev/build-push-deploy created
```

3. Last but not least, let's run our `build-push-deploy-pipelinerun.yml` which is represented as a [PipelineRun](https://tekton.dev/docs/getting-started/pipelines/). A PipelineRun, represented in the API as an object of kind PipelineRun, sets the value for the parameters and executes a Pipeline. Run `kubectl apply -f build-push-deploy-pipelinerun.yml`. And afterwards we can see the live logs of our Tasks running in order by running:

```bash
tkn pipelinerun logs build-push-deploy -f -n default

# Or you can also describe the necessary pipelinerun
tkn pipelinerun describe build-push-deploy
```

### 🐞 Debugging: `invalid input params for task deploy`

After running the above two commands, I got this:
```bash
# kubectl apply -f build-push-deploy-pipelinerun.yml
pipelinerun.tekton.dev/build-push-deploy created

# tkn pipelinerun logs build-push-deploy -f -n default
invalid input params for task deploy: missing values for these params which have no default values: [REPO_URL]
```

This was because I originally had the `REPO_URL` in the `build-task` of my `build-push-deploy-pipeline.yml` file, but not in the `deploy-task`.




## Challenge

0. As we hinted in step 2 for the [Main Project](#main-project), we can actually push a real image to our registry and use it as part of the `deploy.yml` task. In order to do this, some pre-requistes include:

- Having access to a GitHub repo for a project that contains a Dockerfile ([example repo here](https://github.com/sahat/hackathon-starter)). If you personally have one, feel free to use yours.

    Now to get this working lets:
    
    a. Update `build.yml` and `build-push-deploy-pipeline.yml` to update the `REPO_URL`.

    b. Follow [this minikube guide on container registry add-ons](https://minikube.sigs.k8s.io/docs/handbook/registry/) to connect to your GitHub Container Registry account 

    c. In the `push.yml`, uncomment the boiler code near the end of the file.

    d. Update the necessary parameters in the `push` Task under the `build-push-deploy-pipeline.yml`.

    e. For the `deploy.yml`, change Steps to (1) point to a new Deployment file for your Dockerized app. Feel free to use the file in `k8s/app-deployment.yml` to guide that process. 