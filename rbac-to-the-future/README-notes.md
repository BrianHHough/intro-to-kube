# Week 3: RBAC To the Future & Calico Networking

## RBAC Project Notes:

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