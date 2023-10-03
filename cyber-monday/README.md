# [Cyber Monday](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/project-1-instructions)

## Disclaimer

If this is your first time working with kubectl to discover cluster resources, we strongly recommended diving into the [Scavenger project](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/scavenger-hunt) to learn about all about [minikube](https://minikube.sigs.k8s.io/docs/), k8s, and kubectl.

## Note

You can also follow along with the info through the Uplimit Course [here](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/project-1-instructions).

## Background

Hey there, Rockstar Engineer at StartupCo! 🎉 Guess what's just around the corner? The craziest shopping day of the year, Cyber Monday! 💻💥 We need to prepare our web app for this epic event, and it's going to be a wild ride! 🎢

We're counting on you to configure a highly available web app that can handle the massive traffic onslaught. 🚀 Don't worry we aren't exposing the app to the public yet, we just need all the pieces in place for when we do!

We will:

- Deploy a single nginx Pod with containerPort 80 and label `just-a-pod`
- Deploy multiple [nginx containers](https://hub.docker.com/_/nginx) 🐳 using a Deployment with 3 ReplicaSets. The nginx containers should also have a containerPort of 80 and a label `just-a-pod`.
- Use `kubectl cp` and `kubectl exec` commands to support an automated script to take backups of a log file within our pods and push it into our local machine.

Get ready to configure this epic setup, and let's make this Cyber Monday the stuff of legends! 🎆 Best of luck, and let's rock this!

## Setup

Start your minikube and let the games begin!

```
 minikube start
 
 cd cyber-monday
```

## Testing

We have configured [GitHub Actions](https://github.com/features/actions) with minikube! GitHub Actions works as a CI/CD tool but for our intents and purposes, we will use it to create an autograder for your `yml` file changes. To test, create a branch in our `intro-to-kube` repo prefixed with
`cyber-*` and push changes up to that branch. [See the output of your results here for reference](https://github.com/abanuelo/intro-to-kube/actions/workflows/cyber-monday.yml).

```
# create your branch
git checkout -b cyber-brianhhough
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin cyber-<github-username>
```

## Main Project

0. Let's start simple. Create a single nginx pod with
   > a. containerPort 80 
   
   > b. label `just-a-pod` from the nginx latest image
   
   Complete, `nginx-pod.yml`. Run the command below to test or see [Testing](#testing).

   The updated file looks like this:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
      name: nginx-pod
      # TODO: 
      # - Add your nginx-pod spec with label `just-a-pod` and containerPort 80
      labels:
         just-a-pod: "true"
   spec:
      containers:
         - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
   ```

   To deploy: `kubectl apply -f nginx-pod.yml`

   To test if this pod is running: `kubectl get pods nginx-pod`

   This logged:
   ```bash
   NAME        READY   STATUS    RESTARTS   AGE
   nginx-pod   1/1     Running   0          2m13s
   ```
   

1. Now we are going to take that same pod but wrap it in a deployment so that 3 Replicas of that Pod exist at any given time. Please refer to [K8s Deployment documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) as needed. Complete the YAML file to:

   > a. Add 3 replicas

   > b. Add a selector that matches labels `just-a-pod`

   > c. Add a template for the nginx container used in `nginx-pod.yml`

      Once you feel confident in your code run the snippet below or see [Testing](#testing).

   Run: `kubectl apply -f nginx-deploy.yml`

   ^ I tried to run this but got the following error:
   ```bash
   Warning: resource deployments/nginx-deployment is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
   The Deployment "nginx-deployment" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"nginx", "just-a-pod":"true"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
   ```

   What I needed to do was delete the existing Deployment pod with: `kubectl delete deployment nginx-deployment` and then re-aplly the `nginx-deploy.yml` file with this: `kubectl apply -f nginx-deploy.yml`

2. As users start pinging these pods (which won't happen yet fortunately for us 😊), we want to collect metadata around what items they are purchasing for Cyber Monday. Since our logging and metrics aren't set in place yet, we are going to try to collect this metadata by supporting an automated script to take backups of a logfile within our pods and push it into our local machine. To do so we will:

      a. Copy our logging script `log.sh` into our pod, executing it, and observing it write metadata to `/tmp/foo.log` to simulate users pinging the pods and data getting logged.

      b. Run a kubectl command to copy the `/tmp/foo.log` file to our local machine's working directory `/workspaces/intro-to-kube/cyber-monday`

   To do so, we will be using the [`kubectl cp`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cp) command to copy files to and from our pods and a [`kubectl exec`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec) command to execute the `log.sh` script.

   Step #1: Run while in `cyber-monday`: `kubectl cp log.sh nginx-pod:/tmp/log.sh`
   
   Step #2: Provide execute permissions to execute the `log.sh` script in the Nginx Pod:  
   ```bash
   kubectl exec nginx-pod -- chmod +x /tmp/log.sh
   kubectl exec nginx-pod -- /tmp/log.sh
   ```

   Step #3: Copy generated `/tmp/foo.log` from Nginx Pod to codespace directory `/workspaces/intro-to-cube/cyber-monday` with this command:
   ```bash
   kubectl cp nginx-pod:/tmp/foo.log /workspaces/intro-to-kube/cyber-monday/foo.log
   ```

3. Let's start on our single Pod created from the `nginx-pod.yml`. Run the appropriate `kubectl cp` to copy over the `log.sh` to the Pod's `/tmp/` directory. Write this command in the `backup.txt`.

4. Now let's run the `log.sh` script inside the Pod using the appropriate `kubectl exec` command. Before this let's just give the `/tmp/log.sh` proper permissions and touch the `/tmp/foo.log` file by running:

   ```
   kubectl exec nginx-pod -- /bin/sh -c "chmod 777 /tmp/log.sh && touch /tmp/foo.log"
   ```

   Write your command in `backup.txt`. To verify if this worked, you can use a `kubectl exec` command to cat the contents of that file using: `tail -n 10 /tmp/foo.log`.

   The command to test this is: `kubectl exec nginx-pod -- tail -n 10 /tmp/foo.log`

   After running this, I saw this printed, meaning that it worked! 
   ```json
   {"item": "POS system", "user": 112, "date": "2023-10-03T07:45:26Z"}
   {"item": "mouse", "user": 104, "date": "2023-10-03T07:45:26Z"}
   {"item": "laptop", "user": 693, "date": "2023-10-03T07:45:26Z"}
   {"item": "smart gloves", "user": 941, "date": "2023-10-03T07:45:26Z"}
   {"item": "wireless charger", "user": 349, "date": "2023-10-03T07:45:26Z"}
   {"item": "handheld scanner", "user": 555, "date": "2023-10-03T07:45:26Z"}
   {"item": "handheld scanner", "user": 467, "date": "2023-10-03T07:45:26Z"}
   {"item": "3D printer", "user": 779, "date": "2023-10-03T07:45:26Z"}
   {"item": "signal generator", "user": 393, "date": "2023-10-03T07:45:26Z"}
   {"item": "power strip", "user": 244, "date": "2023-10-03T07:45:26Z"}
   ```

5. Last but not least, let's write a `kubectl cp` command to copy the `/tmp/foo.log` file from inside the Pod onto our machine's current working directory. You can verify it works when the file `foo.log` appears in your current working directory. Write this command inside the `backup.txt` file.

## Challenge
0. We have added the automated scripting to our Pod created by `nginx-pod.yml`. However, we have not added that same logic to the Pods encapsulated by our Deployment. For this challenge, repeat the same steps for all pods in the deployment, but ensure that when copying the log files to our local machine, they are prefixed by the Pod's unique identifier.

### Step #1

To list all the pods created by the deployment, we wil run this: 
```bash
kubectl get pods -l just-a-pod=true
```
This printed out the following table, showing the 3 deployment pods:
```bash
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5f54d4d754-rzv4m   1/1     Running   0          27m
nginx-deployment-5f54d4d754-wbntq   1/1     Running   0          27m
nginx-deployment-5f54d4d754-xzwj6   1/1     Running   0          27m
nginx-pod                           1/1     Running   0          48m
```

### Step #2

Copy `log.sh` script into each of the deployment pods:
```bash
kubectl cp log.sh nginx-deployment-5f54d4d754-rzv4m:/tmp/log.sh
kubectl cp log.sh nginx-deployment-5f54d4d754-wbntq:/tmp/log.sh
kubectl cp log.sh nginx-deployment-5f54d4d754-xzwj6:/tmp/log.sh
```

### Step #3

Give the script each the permissions to execute:
```bash
kubectl exec nginx-deployment-5f54d4d754-rzv4m -- /bin/sh -c "chmod +x /tmp/log.sh && /tmp/log.sh"
kubectl exec nginx-deployment-5f54d4d754-wbntq -- /bin/sh -c "chmod +x /tmp/log.sh && /tmp/log.sh"
kubectl exec nginx-deployment-5f54d4d754-xzwj6 -- /bin/sh -c "chmod +x /tmp/log.sh && /tmp/log.sh"
```

### Step #4

Copy each deployment pod's log file to codespace and include a prefix of pod's unique identifier:
```bash
kubectl cp nginx-deployment-5f54d4d754-rzv4m:/tmp/foo.log ./nginx-deployment-5f54d4d754-rzv4m-foo.log
kubectl cp nginx-deployment-5f54d4d754-wbntq:/tmp/foo.log ./nginx-deployment-5f54d4d754-wbntq-foo.log
kubectl cp nginx-deployment-5f54d4d754-xzwj6:/tmp/foo.log ./nginx-deployment-5f54d4d754-xzwj6-foo.log

```
