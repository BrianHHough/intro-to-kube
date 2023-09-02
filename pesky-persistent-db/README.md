# RBAC To The Future - Role-Based Access Control

## Note

You can also follow along with the info through the Uplimit Course [here](https://uplimit.com/course/kubernetes-managing-containers-at-scale/v2/enrollment/enrollment_clj4nmkr201xv12aw259t4xtw/module/project-2-instructions).

## Background

🌱 Hey there, fellow adventurer at StartupCo! 🚀 Let me tell you a little story to describe why we need to add Role-Based Access Control (RBAC) as our company grows. And so RBAC to the Future we go. 🌟

At first, it was all good vibes and everyone could see the pods dancing around, spreading their digital wonders. 🎉 But as our microservice world expanded and more components joined the party, we realized the need for a touch of order and security. 🔒

So, we need the mighty power of Role-Based Access Control (RBAC) to bring stability to our company. The company decided to create multiple namespaces:

-  `dev` - Developers work on application code, test new features, and perform integration testing in this namespace
- `staging` - Before deploying to production, applications are tested and validated in this environment to catch potential issues.
- `prod` - This is the live production environment where your applications and services serve end-users.
-  `database` - Database systems like MySQL, PostgreSQL, or MongoDB are isolated in this namespace to manage database-related resources.
- `backup` - Resources and tools for backup and disaster recovery, such as Velero, are placed in this namespace.

Now that each namespace has been created, there was a need to design new roles to control who has read/write/delete access to any resources in these namespaces

- `KubeAdmin` - Full access to the entire cluster, including all namespaces and resources.
- `Developer` - Permissions to create, read, update, and delete pods, services, config maps, and other application-specific resources within a `dev`, `staging`, and `prod` namespaces.
- `DBAdmin` - Permissions to manage storage-related resources like backups and direct data in databases from the `database` and `backup` namespaces.
- `Intern` - Permissions to view resources but not modify or create them in the `dev` namespaces exclusively

Your task here will be to create these Role, ClusterRoles, RoleBindings, and ClusterRoleBindings for our newest team members:

- **Jesus** - KubeAdmin
- **Joey** - Developer
- **Jessica** - DBAdmin
- **Jules** - Intern

And with the power of these roles, we could grant these individuals with the proper access to weave their creativity into our ever-evolving universe. 🎨

May our StartupCo continue to grow, thrive, and spread its digital wonders throughout the land! 🌍💫

## Setup

Start up your `minikube` environment by running:

```
cd rbac-to-the-future
bash setup.sh
```

Next create the following users and contexts by running the `create_users.sh` script:
```
bash create_users.sh
```

Afterwards run `kubectl config view` and expect to see something like this under `contexts` and `users`
```

```

## Testing

We have configured [GitHub Actions](https://github.com/features/actions) with minikube! GitHub Actions works as a CI/CD tool but for our intents and purposes, we will use it to create an autograder for your `yml` file changes. To test, create a branch in our `intro-to-kube` repo prefixed with
`rbac-*` and push changes up to that branch. [See the output of your results here for reference](https://github.com/abanuelo/intro-to-kube/actions/workflows/rbac-to-the-future.yml).

```
# create your branch
git checkout -b rbac-<github-username>
# add your local changes
git add .
# commit your changes
git commit -m "<describe changes made>"
# push your changes to your upstream branch
git push -u origin rbac-<github-username>
```

## Main Project

0. Let's start with easiest [Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) and [RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to build. Let's give read only access to the `Intern` role by defining `get`, `watch`, and `list` actions to `Pod` and `Deployment` resources strictly to the `dev` namespace. Afterwards bind this `Role` to Jules via the `RoleBinding`. When you are ready run:

   ```
   kubectl config set-context --current --namespace=dev
   kubectl apply intern.yml
   ```

   To test if Jules has the proper restrictions in place by running:
   ```
   # change to the Jules user context
   kubectl config use-context Jules-context

   # listing the pods within the staging namespace should fail ❌
   kubectl get pods --namespace staging

   # creating resources should fail ❌
   kubectl run my-pod --image=nginx --namespace dev

   # listing the pods within the dev namespace should pass ✅ 
   kubectl get pods --namespace dev
   ```

You can run similar tests for the remaining Roles and RoleBindings we will create next or look at the [Testing](#testing) for more details.


1. Next we will work on defining the PodWriter. We have two files as well:

   - `podwriter-role.yml` will find a Role on the `Pod` resources with verbs `get`, `create`, `delete`, `update`, `edit`, and `exec`.
   - `podwriter-rolebinding.yml` will create the necessary RoleBinding to the podwriter user.

   When you are ready run `kubectl apply -f podwriter-role.yml` and `kubectl apply -f podwriter-rolebinding.yml`.

2. Next we are going to try some of operations for the podreader to determine if we can truly just read pods. Let's switch into the podreader context:

```
kubectl config use-context podreader-context
#should fail
kubectl create namespace test
# should pass
kubectl get pod #should pass
```

3. We will do the same for podwriter. Let's run the following lines of code

```
kubectl config use-context podwriter-context
#should pass
kubectl get pod
# should pass
kubectl edit pod <pod-name>
# should fail
kubectl create namespace test
```

## Challenge
