# [Scavenger Hunt](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/scavenger-hunt)

## Note

You can also follow along with the info through the Uplimit Course [here](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/scavenger-hunt).

## Background

🎉 Woohoo!!!! 🎉 We are thrilled to welcome you to our DevOps team at StartupCo! 🚀 Your expertise and skills will undoubtedly bring a fresh perspective to our innovative environment.

Your next ramp-up task is going to be an exciting one! Get ready for a Kubernetes Scavenger Hunt 🕵️‍♂️🔍, where you'll embark on an adventure to explore the resources within our existing cluster. This engaging exercise will enable you to dive deeper into Kubernetes and discover its hidden gems, all while having a ton of fun! Let the Kubernetes Scavenger Hunt begin! May the pods be ever in your favor! 🌟

## Setup

Create K8s resources by starting up minikube and running the following commands:

```
cd scavenger
./scavenger-setup.sh
```

## Testing

Add add your answers to `scavenger.json`

We have [GitHub Actions](https://github.com/features/actions) configured to report your score to our scavenger hunt. Just create your own branch prefixed with `scavenger-` and push to it. [Then you can see your results here](https://github.com/abanuelo/intro-to-kube/actions/workflows/scavenger.yml).

```
# create your branch
git checkout -b scavenger-<github-username>
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin scavenger-<github-username>
```

## Main Project

### Mark I - new beginngs

0. **What are the total number of pods?**
To get the pods, run `kubectl get pods --all-namespaces | wc -l`, which prints out: 13

1. **How many namespaces are there?**
To get the namespaces, run `kubectl get namespaces | wc -l`, which prints out: 6

2. **How many nodes do we have?**
To get the nodes, run `kubectl get nodes | wc -l`, which prints out: 2

3. **How many failing pods are there?**
To get the failing pods, run `kubectl get pods --all-namespaces --field-selector=status.phase!=Running`, which logs this:

```bash
NAMESPACE   NAME    READY   STATUS             RESTARTS   AGE
scavenger   nginx   0/1     ImagePullBackOff   0          7h48m
```

The answer is 1.

4. **How many deployments do we have?**
To get the deployments, run `kubectl get deployments --all-namespaces | wc -l`, which logs: 3. Then I need to -1 for the header row, so the answer is 2.


5. **For the deployments you discovered, how many ReplicaSets are there in total?**
To get the ReplicaSets, run `kubectl get replicasets --all-namespaces | wc -l`, which logs 3. Then I need to -1 for the header row, so the answer is 2.

### Mark II - not so new beginnings

6. **Locate a postgres pod within our cluster. Why is it failing? Please provide the variable name(s) that is missing.**

To describe the `postgres` pod logs to identify the issue, I ran `kubectl logs postgres -n scavenger` and this logged below. Basically, it's saying our database pod doesn't have a password set yet, or in kubernetes speak: "Database is uninitialized and superuser password is not specified" and the missing variable is `POSTGRES_PASSWORD`

```bash
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html
```

So we could update the `postgres-pod.yaml` file to look like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
    - name: postgres
      image: postgres
      env:
      - name: POSTGRES_PASSWORD
        value: "supersecret"
```

But this is not very secure and doesn't leverage secrets, so let's do that instead.

First, we will create a Secret for the PostgreSQL password:
`kubectl create secret generic pgpassword --from-literal=POSTGRES_PASSWORD=yourpassword -n scavenger`
- This creates a Secret named `pgpassword`:
    - key: `POSTGRES_PASSWORD`
    - value: `supersecret`


### Note:
Let's say we need to update our Secret... we can run: `kubectl delete secret pgpassword -n scavenger`

Then, recreate the secret: `kubectl create secret generic pgpassword --from-literal=POSTGRES_PASSWORD=supersecret -n scavenger`

But, we'll also need to restart the pod: `kubectl delete -f scavenger/k8s/postgres-pod.yml -n scavenger`

And then recreate the pod: `kubectl apply -f scavenger/k8s/postgres-pod.yml -n scavenger`

Now we can update our yaml file with the secret:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
    - name: postgres
      image: postgres
      env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pgpassword
              key: POSTGRES_PASSWORD
```

Finally, we will update the Pod definition:
`kubectl apply -f scavenger/k8s/postgres-pod.yml -n scavenger`

^ I got the following error when I tried to run this below. The reason is that pods are mostly immutable after creation, so we can't change their fields once they've been created.
```bash
Warning: resource pods/postgres is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
The Pod "postgres" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing tolerations),`spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
  core.PodSpec{
        Volumes:        {{Name: "kube-api-access-pzvms", VolumeSource: {Projected: &{Sources: {{ServiceAccountToken: &{ExpirationSeconds: 3607, Path: "token"}}, {ConfigMap: &{LocalObjectReference: {Name: "kube-root-ca.crt"}, Items: {{Key: "ca.crt", Path: "ca.crt"}}}}, {DownwardAPI: &{Items: {{Path: "namespace", FieldRef: &{APIVersion: "v1", FieldPath: "metadata.namespace"}}}}}}, DefaultMode: &420}}}},
        InitContainers: nil,
        Containers: []core.Container{
                {
                        ... // 5 identical fields
                        Ports:   nil,
                        EnvFrom: nil,
-                       Env:     nil,
+                       Env: []core.EnvVar{
+                               {
+                                       Name:      "POSTGRES_PASSWORD",
+                                       ValueFrom: &core.EnvVarSource{SecretKeyRef: &core.SecretKeySelector{...}},
+                               },
+                       },
                        Resources:    {},
                        ResizePolicy: nil,
                        ... // 13 identical fields
                },
        },
        EphemeralContainers: nil,
        RestartPolicy:       "Always",
        ... // 28 identical fields
  }
```

First we'll need to delete the existing pod and then re-apply the updated pod definition:

Delete: `kubectl delete pod postgres -n scavenger`

Re-apply: `kubectl apply -f scavenger/k8s/postgres-pod.yml -n scavenger`

Now if we run `kubectl logs postgres -n scavenger`, we get a much different log output:

```bash
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok


Success. You can now start the database server using:

    pg_ctl -D /var/lib/postgresql/data -l logfile start

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.
waiting for server to start....2023-10-03 05:17:17.041 UTC [47] LOG:  starting PostgreSQL 16.0 (Debian 16.0-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
2023-10-03 05:17:17.055 UTC [47] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-10-03 05:17:17.096 UTC [50] LOG:  database system was shut down at 2023-10-03 05:17:13 UTC
2023-10-03 05:17:17.112 UTC [47] LOG:  database system is ready to accept connections
 done
server started

/usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*

waiting for server to shut down....2023-10-03 05:17:17.484 UTC [47] LOG:  received fast shutdown request
2023-10-03 05:17:17.499 UTC [47] LOG:  aborting any active transactions
2023-10-03 05:17:17.508 UTC [47] LOG:  background worker "logical replication launcher" (PID 53) exited with exit code 1
2023-10-03 05:17:17.509 UTC [48] LOG:  shutting down
2023-10-03 05:17:17.521 UTC [48] LOG:  checkpoint starting: shutdown immediate
2023-10-03 05:17:17.649 UTC [48] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.036 s, sync=0.015 s, total=0.140 s; sync files=2, longest=0.009 s, average=0.008 s; distance=0 kB, estimate=0 kB; lsn=0/14EAA88, redo lsn=0/14EAA88
2023-10-03 05:17:17.653 UTC [47] LOG:  database system is shut down
 done
server stopped

PostgreSQL init process complete; ready for start up.

2023-10-03 05:17:17.763 UTC [1] LOG:  starting PostgreSQL 16.0 (Debian 16.0-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
2023-10-03 05:17:17.765 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-10-03 05:17:17.765 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2023-10-03 05:17:17.791 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-10-03 05:17:17.819 UTC [61] LOG:  database system was shut down at 2023-10-03 05:17:17 UTC
2023-10-03 05:17:17.836 UTC [1] LOG:  database system is ready to accept connections
```

✅ We fixed the database pod and it's ready to go! 

7. **Locate a failed nginx pod within our cluster. Why is it failing? Please copy and paste the specific `kubectl` log you find. Please escape `"` with `\"`.**

To describe the `nginx` pod logs to identify the issue, I ran `kubectl logs nginx -n scavenger` and this logged the below. 

```bash
Error from server (BadRequest): container "nginx" in pod "nginx" is waiting to start: trying and failing to pull image
```

The reason it is failing is because of the spelling of nginx in the `nginx-pod.yml` file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginz
```

The container's image name should be `nginx`, not `nginz`

The answer with escaped quotes is this: "Error from server (BadRequest): container \"nginx\" in pod \"nginx\" is waiting to start: trying and failing to pull image"


## Challenge

### Mark III - fixing our bugs

8. **For the postgres pod, can you fix the `./k8s/postgres-pod.yml` file to spin up the image correctly? Please add the environment variable needed with value "supersecret". Added your lines of code.**

`POSTGRES_PASSWORD`

Updated `postgres-pod.yml` file:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
    - name: postgres
      image: postgres
      env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pgpassword
              key: POSTGRES_PASSWORD
```


9. **For the failed nginx pod, can you fix the `./k8s/nignx-pod.yml` file to spin up the image correctly. Add the lines of code you added/corrected and please escape `"` with `\"`.**

`"image: nginx"`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
```

I had to do `kubectl delete -f scavenger/k8s/nginx-pod.yml -n scavenger`
and then this:
I had to do `kubectl apply -f scavenger/k8s/nginx-pod.yml -n scavenger`

# GitHub Actions

```bash
# create your branch
git checkout -b scavenger-brianhhough
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin scavenger-<github-username>
```