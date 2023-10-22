# Week 3: RBAC To the Future & Calico Networking

## RBAC Project Notes:

## Configuring the users for the startup of the minikube instance:

Running `kubectl config view` logs this:

```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/codespace/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Sun, 22 Oct 2023 05:45:20 UTC
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: Jessica
  name: Jessica-context
- context:
    cluster: minikube
    user: Jesus
  name: Jesus-context
- context:
    cluster: minikube
    user: Joey
  name: Joey-context
- context:
    cluster: minikube
    user: Jules
  name: Jules-context
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Sun, 22 Oct 2023 05:45:20 UTC
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: Jessica
  user:
    client-certificate: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jessica.crt
    client-key: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jessica.key
- name: Jesus
  user:
    client-certificate: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jesus.crt
    client-key: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jesus.key
- name: Joey
  user:
    client-certificate: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Joey.crt
    client-key: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Joey.key
- name: Jules
  user:
    client-certificate: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jules.crt
    client-key: /workspaces/intro-to-kube/rbac-to-the-future/user_certs/Jules.key
- name: minikube
  user:
    client-certificate: /home/codespace/.minikube/profiles/minikube/client.crt
    client-key: /home/codespace/.minikube/profiles/minikube/client.key
```

To then set up the `intern.yml` file, I created the following:
- I separated the API Groups so that "pods" belongs to the core API group which is "", while "deployments" belongs to the "apps" API group.
- This way, each resource is separate and correctly associated within its API group.
- For the rolebindings for the the subjects, I am specifying specific rules for "Jules" (one person), then defining permissions for the Role named "Intern", and then I assign those permissions to the user "Jules" through RoleBinding.

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: Intern
  namespace: dev
rules:
# TODO:
# target pods with verbs get, watch, list
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: InternBinding
  namespace: dev
subjects:
# TODO: add Jules as target user
# note: this is case sensitive
- kind: User
  name: "Jules"
  apiGroup: rbac.authorization.k8s.io
roleRef:
# TODO: add target role to Intern
  kind: Role
  name: Intern
  apiGroup: rbac.authorization.k8s.io
```

After saving changes, I ran this command: `kubectl apply -f ./roles/intern.yml`

This printed out:
```bash
role.rbac.authorization.k8s.io/Intern unchanged
rolebinding.rbac.authorization.k8s.io/InternBinding created
```


### 🐞 Debugging: 
There was an error where I had this below, and it errored because of the -. The `roleRef` in a `roleBinding` (or `ClusterRoleBinding`) can only reference one single, specific role, and cannot be an array. So that's why this won't work:
```bash
roleRef:
- kind: Role
  name: Intern
  apiGroup: rbac.authorization.k8s.io
```

But this will:
```bash
roleRef:
  kind: Role
  name: Intern
  apiGroup: rbac.authorization.k8s.io
```


