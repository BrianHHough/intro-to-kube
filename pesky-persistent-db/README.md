# Pesky Persistent Data

Once upon a time at StartupCo, an ingenious engineer embarked on a mission to create a 🐘PostgreSQL database that would hold the key to the company's data treasure trove. Despite their best efforts, the database refused to persist data. Later it was discovered that the PostgreSQL DB was stored as a Deployment! Deployments can create Pods with persistent data but that data needs to be brand new for each Pod every time or shared across all Pods. And when they performed maintenance on some Pods in this Deployment, it was hard to find a way to maintain the storage such that each pod is attached to the right set of data.

Undeterred by this setback, our determined engineer donned their virtual superhero cape and hatched a brilliant plan. With a wave of their coding wand, they transformed Deployment by attaching a Persistent Volume! A Persistent Volume is needed to determine where to safely mount data in their container. And of course if you have a Persistent Volume, you need to define a Persistent Volume Claim to determine a request for storage by a user.

As the pieces fell into place, our engineer activated a Service, allowing them to interact with the postgres database in all its glory. But given the heavy load on their plate from the latest sprint, they were unable to develop the Deployment, Persistent Volume, nor Persistent Volume Claim.

This is where a newest protoge, comes into existence. Our engineer's persistence paid off, and the team rejoiced, knowing that their data was in capable hands. 🎉💾

## Setup

Start up your `minikube` environment by running:

```
cd pesky-persistent-db
minikube start
```

We will see this on startup of minikube:
```bash
😄  minikube v1.31.2 on Ubuntu 20.04 (docker/amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Testing

We have configured [GitHub Actions](https://github.com/features/actions) with minikube! GitHub Actions works as a CI/CD tool but for our intents and purposes, we will use it to create an autograder for your `yml` file changes. To test, create a branch in our `intro-to-kube` repo prefixed with
`persistent-*` and push changes up to that branch. [See the output of your results here for reference](https://github.com/abanuelo/intro-to-kube/actions/workflows/pesky-persistent-db.yml).

```bash
# create your branch
git checkout -b persistent-<github-username>
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin persistent-<github-username>
```

## Main Project

Check the pods we have: `kubectl get pods -o wide`
This outputs:
```bash
NAME                                READY   STATUS                       RESTARTS        AGE   IP            NODE       NOMINATED NODE   READINESS GATES
nginx                               1/1     Running                      2 (3m50s ago)   12d   10.244.0.56   minikube   <none>           <none>
nginx-deployment-5f54d4d754-rzv4m   1/1     Running                      1 (3m50s ago)   12d   10.244.0.59   minikube   <none>           <none>
nginx-deployment-5f54d4d754-wbntq   1/1     Running                      1 (3m50s ago)   12d   10.244.0.54   minikube   <none>           <none>
nginx-deployment-5f54d4d754-xzwj6   1/1     Running                      1 (3m50s ago)   12d   10.244.0.63   minikube   <none>           <none>
nginx-pod                           1/1     Running                      1 (3m50s ago)   12d   10.244.0.62   minikube   <none>           <none>
postgres                            0/1     CreateContainerConfigError   0               12d   10.244.0.60   minikube   <none>           <none>
```

Services were created to connect to pods:
- ClusterIP: assignes IP from pool of IP reserved clusters (for internal services / non-public)
- NodePort: allocates port from ranges 30000-32767 to pods and forwards traffic to service



0. Luckily we have some resources: `postgres-service.yml`. The [Service](https://kubernetes.io/docs/concepts/services-networking/service/) was intended to provide a Domain Name System (DNS) to be able to connect with the Pods attached to the Deployment. Run the lines of code to create the Service

```bash
kubectl apply -f postgres-service.yml
```

The output is: `service/postgres created`

For now we are assuming that we want to provide a NodePort Service for our postgres database. To illustrate how the services work, please refer to the ./pesky-persistent-db/examples folder and run:

```bash
kubectl apply -f ./examples/nginx-deployment.yml
kubectl apply -f ./examples/nginx-service.yml
```


1. Let's start with the low-hanging fruit first. We are going to add values to a K8s ConfigMap needed for our Postgres Database. Please define the values in the `postgres-configmap.yml` file. [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) are intended to define non-confidential key-value pairs. Ideally, we would use a K8s [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) to store our DB connection info, but for the purposes of easy debugging and high visibility, this will work for now. To test your change please run:

```bash
kubectl apply -f postgres-configmap.yml
```

Output is: `configmap/postgres-config created`
Output is (after configuring values for postgres db): `configmap/postgres-config configured`

2. Now that we have this, let's work to define the [Persistent Volume and Persistent Volume Claim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) within the `postgres-persistent.yml` file. For the

   Persistent Volume you will

   - set the accessModes as ReadWriteMany
   - set the capacity to 5Gi
   - set the hostPath to /mnt/data
   - set the storageClassName to manual

   For the Persistent Volume Claim you will:

   - set the accessModes as ReadWriteMany
   - set the resource requests storage to 5Gi
   - set the storageClassName to manual

   You can create the objects by running the snippet below.

```bash
kubectl apply -f postgres-persistent.yml
```

The output is:
```bash
persistentvolume/postgres-pv-volume created
persistentvolumeclaim/postgres-pv-claim created
```

Hint: You may want to run kubectl explain pv.spec --recursive and kubectl explain pvc.spec --recursive to find where exactly to add these specifications in your YAML file.

^ See: ./w2/README.md for the logs from this!


3. Now we need to configure the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to point (a) specific a place to mount data on the containers for the pods, (b) add the key-value pairs defined from the `postgres-configmap.yml` file and (c) configure the persistent volume claim. More details are available within the `postgres-deployment.yml`. For now we will only configure a single replica but feel free to change that as you wish. Once you feel confident, run the snippet of code below:

```bash
kubectl apply -f postgres-deployment.yml
```

The output is: `deployment.apps/postgres created`


   For this step, we before we test out the connection to the Postgres DB via our service, give it a go by `exec`ing into the pod created by the Deployment to run the following command:

```bash
# grab the pod name via
kubectl get pod
```
Output:
```bash
NAME                                READY   STATUS                       RESTARTS       AGE
nginx                               1/1     Running                      2 (154m ago)   12d
nginx-deployment-5f54d4d754-rzv4m   1/1     Running                      1 (154m ago)   12d
nginx-deployment-5f54d4d754-wbntq   1/1     Running                      1 (154m ago)   12d
nginx-deployment-5f54d4d754-xzwj6   1/1     Running                      1 (154m ago)   12d
nginx-pod                           1/1     Running                      1 (154m ago)   12d
postgres                            0/1     CreateContainerConfigError   0              12d
postgres-595d76d4f8-r8ndp           1/1     Running                      0              26m
```

```bash
# check the postgres connection
kubectl exec -it <pod name> -- psql -h localhost -U admin --password -p 5432 postgresdb
```

What I ran was: `kubectl exec -it postgres-595d76d4f8-r8ndp -- psql -h localhost -U admin --password -p 5432 postgresdb`

I was able to log into the db with `psltest` like this:
```bash
kubectl exec -it postgres-595d76d4f8-r8ndp -- psql -h localhost -U admin --password -p 5432 po
stgresdb
Password for user admin: 
psql (10.1)
Type "help" for help.

postgresdb=# 
```


   If you can connect to the postgres pod then you have successfully (a) and (b). We will check (c) in the next step.

You are on the right path if when you kubectl describe your postgres Pod you see the volume mount in the Volumes: section:
```yaml
Volumes:
   postgredb:
      Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
      ClaimName:  postgres-pv-claim
      ReadOnly:   false
```

To see if data is actually getting populated into the PV run. The contents of this should be what is identical to whats inside the /var/lib/postgresql/data inside the Pod:

```bash
# See contents of PV
# command below opens up docker container acting as a Node
minikube ssh -p minikube 
sudo ls /mnt/data
```
Output is:
```bash
minikube ssh -p minikube
docker@minikube:~$ sudo ls /mnt/data
PG_VERSION  global        pg_dynshmem  pg_ident.conf  pg_multixact  pg_replslot  pg_snapshots  pg_stat_tmp  pg_tblspc    pg_wal   postgresql.auto.conf  postmaster.opts
base        pg_commit_ts  pg_hba.conf  pg_logical     pg_notify     pg_serial    pg_stat       pg_subtrans  pg_twophase  pg_xact  postgresql.conf       postmaster.pid
```

```bash
# Expected to equal contents inside Pod
kubectl exec -it <pod name> -- ls /var/lib/postgresql/data
```

I ran this: `kubectl exec -it postgres-595d76d4f8-r8ndp -- ls /var/lib/postgresql/data`
Output is:
```bash
base          pg_ident.conf  pg_serial     pg_tblspc    postgresql.auto.conf
global        pg_logical     pg_snapshots  pg_twophase  postgresql.conf
pg_commit_ts  pg_multixact   pg_stat       PG_VERSION   postmaster.opts
pg_dynshmem   pg_notify      pg_stat_tmp   pg_wal       postmaster.pid
pg_hba.conf   pg_replslot    pg_subtrans   pg_xact
```



To check if the ConfigMap has been properly configured run `kubectl describe deployment postgres` and see if the Environment Variables from: looks like:

```bash
Environment Variables from:
   postgres-config  ConfigMap  Optional: false
```

My output was:
```bash
Name:                   postgres
Namespace:              default
CreationTimestamp:      Mon, 16 Oct 2023 05:56:19 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=postgres
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=postgres
  Containers:
   postgres:
    Image:      postgres:10.1
    Port:       5432/TCP
    Host Port:  0/TCP
    Environment:
      POSTGRES_DB:        <set to the key 'POSTGRES_DB' of config map 'postgres-config'>        Optional: false
      POSTGRES_USER:      <set to the key 'POSTGRES_USER' of config map 'postgres-config'>      Optional: false
      POSTGRES_PASSWORD:  <set to the key 'POSTGRES_PASSWORD' of config map 'postgres-config'>  Optional: false
    Mounts:
      /var/lib/postgresql/data from postgredb (rw)
  Volumes:
   postgredb:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  postgres-pv-claim
    ReadOnly:   false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   postgres-595d76d4f8 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  55m   deployment-controller  Scaled up replica set postgres-595d76d4f8 to 1
```


4. Lastly let's test out to see if we can persist our database!

   Let's test the connection to the DB from outside the cluster from the service by running:

```bash
sudo apt update
sudo apt -y  install postgresql postgresql-contrib

psql  -h $(minikube ip) -p $(minikube service postgres --url --format={{.Port}}) -U admin -d postgresdb --password
```

   Let's create a table using the following command. Let's also try to INSERT some data into the table

```sql
CREATE TABLE roles(
   role_id serial PRIMARY KEY,
   role_name VARCHAR (255) UNIQUE NOT NULL
);
INSERT INTO PUBLIC.ROLES VALUES (1, 'student');
```


To validate that the table was created:

1. List all tables: `\dt`
Output is:
```bash
postgresdb=# \dt
       List of relations
 Schema | Name  | Type  | Owner 
--------+-------+-------+-------
 public | roles | table | admin
(1 row)

postgresdb=# 
```
^ Notice the `roles` table there, meaning the db was created

2. Query the `roles` table: `SELECT * FROM roles;`
Output is:
```bash
postgresdb=# SELECT * FROM roles;
 role_id | role_name 
---------+-----------
       1 | student
(1 row)
```
^ I see one row with the values (1, 'student').

3. Describe the table structure: `\d roles`
Output is:
```bash
postgresdb=# \d roles
                                        Table "public.roles"
  Column   |          Type          | Collation | Nullable |                Default                 
-----------+------------------------+-----------+----------+----------------------------------------
 role_id   | integer                |           | not null | nextval('roles_role_id_seq'::regclass)
 role_name | character varying(255) |           | not null | 
Indexes:
    "roles_pkey" PRIMARY KEY, btree (role_id)
    "roles_role_name_key" UNIQUE CONSTRAINT, btree (role_name)
```



   After running this command try to delete the Pod and wait for the Deployment to spin up a new one:

```bash
# get the pod name
kubectl get pod
# delete the pod
kubectl delete pod <pod-name>
# check to see if new pod spins up
kubectl get pod
```

   And lastly check if the "roles" table exists with the entry we inserted:

```bash
   SELECT * FROM PUBLIC.ROLES;
```

   If you see the output below, you have successfully created persistent data!

```bash
   role_id | role_name
---------+-----------
      1 | student
(1 row)
```

The creation of pods as one gets deleted:
```bash
@BrianHHough ➜ /workspaces/intro-to-kube (cyber-brianhhough) $ kubectl get pod
NAME                                READY   STATUS                       RESTARTS        AGE
nginx                               1/1     Running                      2 (3h23m ago)   13d
nginx-deployment-5f54d4d754-rzv4m   1/1     Running                      1 (3h23m ago)   12d
nginx-deployment-5f54d4d754-wbntq   1/1     Running                      1 (3h23m ago)   12d
nginx-deployment-5f54d4d754-xzwj6   1/1     Running                      1 (3h23m ago)   12d
nginx-pod                           1/1     Running                      1 (3h23m ago)   12d
postgres                            0/1     CreateContainerConfigError   0               13d
postgres-595d76d4f8-r8ndp           1/1     Running                      0               75m
@BrianHHough ➜ /workspaces/intro-to-kube (cyber-brianhhough) $ kubectl delete pod postgres-595d76d4f8-r8ndp
pod "postgres-595d76d4f8-r8ndp" deleted
^C@BrianHHough ➜ /workspaces/intro-to-kube (cyber-brianhhough) $ kubectl get pod
NAME                                READY   STATUS                       RESTARTS        AGE
nginx                               1/1     Running                      2 (3h24m ago)   13d
nginx-deployment-5f54d4d754-rzv4m   1/1     Running                      1 (3h24m ago)   12d
nginx-deployment-5f54d4d754-wbntq   1/1     Running                      1 (3h24m ago)   12d
nginx-deployment-5f54d4d754-xzwj6   1/1     Running                      1 (3h24m ago)   12d
nginx-pod                           1/1     Running                      1 (3h24m ago)   12d
postgres                            0/1     CreateContainerConfigError   0               13d
postgres-595d76d4f8-dnvbm           1/1     Running                      0               19s
postgres-595d76d4f8-r8ndp           1/1     Terminating                  0               76m
```



Inside of the pod, once it was deleted, I couldn't access it anymore and had to create a new psql connection which you'll see here:
```bash
postgresdb=# \d roles
                                        Table "public.roles"
  Column   |          Type          | Collation | Nullable |                Default                 
-----------+------------------------+-----------+----------+----------------------------------------
 role_id   | integer                |           | not null | nextval('roles_role_id_seq'::regclass)
 role_name | character varying(255) |           | not null | 
Indexes:
    "roles_pkey" PRIMARY KEY, btree (role_id)
    "roles_role_name_key" UNIQUE CONSTRAINT, btree (role_name)

postgresdb=# kubectl get pod
postgresdb-# SELECT * FROM roles;
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
psql (12.16 (Ubuntu 12.16-0ubuntu0.20.04.1), server 10.1)
postgresdb=# \q
@BrianHHough ➜ /workspaces/intro-to-kube (cyber-brianhhough) $ psql  -h $(minikube ip) -p $(minikube service postgres --url --format={{.Port}}) -U admin -d postgresdb --password
Password: 
psql: error: FATAL:  password authentication failed for user "admin"
@BrianHHough ➜ /workspaces/intro-to-kube (cyber-brianhhough) $ psql  -h $(minikube ip) -p $(minikube service postgres --url --format={{.Port}}) -U admin -d postgresdb --password
Password: 
psql (12.16 (Ubuntu 12.16-0ubuntu0.20.04.1), server 10.1)
Type "help" for help.

postgresdb=# SELECT * FROM PUBLIC.ROLES;
 role_id | role_name 
---------+-----------
       1 | student
(1 row)

```




## Challenge + Extensions

0. As mentioned in the in step 1, it is not recommended to keep highly secure values inside a ConfigMap. Rework the `postgres-deployment.yml` to reference a [K8s Secret](https://kubernetes.io/docs/concepts/configuration/secret/) instead. Feel free to refer to [this GitHub thread](https://github.com/kubernetes/kubernetes/issues/70241#issuecomment-434242145) for assistance. Also feel free to use the `./challenges/postgres-secret.yml` file to make any edits.
1. What if our volume gets corrupted! Well that can always be a risk in the world of Kubernetes. As an extra extension, we recommend developing a backup process for our data. Kubernetes offers [VolumeSnapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) that can faciliate the backup process. There is no one-size answer we can autograder here but we recommend to explore how to use VolumeSnapshots via [this blog](https://blog.palark.com/kubernetes-snaphots-usage/).