# Calico Notes

## Executing the postgres namespace with kubectl

By running: `kubectl exec -it postgres -n namespace-c -- bash`
This outputed:
```bash
root@postgres:/# apt-get -y update; apt-get -y install curl
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [52.1 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:4 http://apt.postgresql.org/pub/repos/apt bookworm-pgdg InRelease [123 kB]
Get:5 http://deb.debian.org/debian bookworm/main amd64 Packages [8,780 kB]
Get:6 http://deb.debian.org/debian bookworm-updates/main amd64 Packages [6,408 B]
Get:7 http://deb.debian.org/debian-security bookworm-security/main amd64 Packages [86.2 kB]
Get:8 http://apt.postgresql.org/pub/repos/apt bookworm-pgdg/main amd64 Packages [296 kB]
Fetched 9,543 kB in 5s (2,040 kB/s)                  
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ca-certificates libbrotli1 libcurl4 libnghttp2-14 libpsl5 librtmp1 libssh2-1 publicsuffix
The following NEW packages will be installed:
  ca-certificates curl libbrotli1 libcurl4 libnghttp2-14 libpsl5 librtmp1 libssh2-1 publicsuffix
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,630 kB of archives.
After this operation, 3,740 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 ca-certificates all 20230311 [153 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 libbrotli1 amd64 1.0.9-2+b6 [275 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 libnghttp2-14 amd64 1.52.0-1 [72.3 kB]
Get:4 http://deb.debian.org/debian bookworm/main amd64 libpsl5 amd64 0.21.2-1 [58.7 kB]
Get:5 http://deb.debian.org/debian bookworm/main amd64 librtmp1 amd64 2.4+20151223.gitfa8646d.1-2+b2 [60.8 kB]
Get:6 http://deb.debian.org/debian bookworm/main amd64 libssh2-1 amd64 1.10.0-3+b1 [179 kB]
Get:7 http://deb.debian.org/debian-security bookworm-security/main amd64 libcurl4 amd64 7.88.1-10+deb12u4 [390 kB]
Get:8 http://deb.debian.org/debian-security bookworm-security/main amd64 curl amd64 7.88.1-10+deb12u4 [315 kB]
Get:9 http://deb.debian.org/debian bookworm/main amd64 publicsuffix all 20230209.2326-1 [126 kB]
Fetched 1,630 kB in 1s (1,917 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package ca-certificates.
(Reading database ... 12085 files and directories currently installed.)
Preparing to unpack .../0-ca-certificates_20230311_all.deb ...
Unpacking ca-certificates (20230311) ...
Selecting previously unselected package libbrotli1:amd64.
Preparing to unpack .../1-libbrotli1_1.0.9-2+b6_amd64.deb ...
Unpacking libbrotli1:amd64 (1.0.9-2+b6) ...
Selecting previously unselected package libnghttp2-14:amd64.
Preparing to unpack .../2-libnghttp2-14_1.52.0-1_amd64.deb ...
Unpacking libnghttp2-14:amd64 (1.52.0-1) ...
Selecting previously unselected package libpsl5:amd64.
Preparing to unpack .../3-libpsl5_0.21.2-1_amd64.deb ...
Unpacking libpsl5:amd64 (0.21.2-1) ...
Selecting previously unselected package librtmp1:amd64.
Preparing to unpack .../4-librtmp1_2.4+20151223.gitfa8646d.1-2+b2_amd64.deb ...
Unpacking librtmp1:amd64 (2.4+20151223.gitfa8646d.1-2+b2) ...
Selecting previously unselected package libssh2-1:amd64.
Preparing to unpack .../5-libssh2-1_1.10.0-3+b1_amd64.deb ...
Unpacking libssh2-1:amd64 (1.10.0-3+b1) ...
Selecting previously unselected package libcurl4:amd64.
Preparing to unpack .../6-libcurl4_7.88.1-10+deb12u4_amd64.deb ...
Unpacking libcurl4:amd64 (7.88.1-10+deb12u4) ...
Selecting previously unselected package curl.
Preparing to unpack .../7-curl_7.88.1-10+deb12u4_amd64.deb ...
Unpacking curl (7.88.1-10+deb12u4) ...
Selecting previously unselected package publicsuffix.
Preparing to unpack .../8-publicsuffix_20230209.2326-1_all.deb ...
Unpacking publicsuffix (20230209.2326-1) ...
Setting up libpsl5:amd64 (0.21.2-1) ...
Setting up libbrotli1:amd64 (1.0.9-2+b6) ...
Setting up libnghttp2-14:amd64 (1.52.0-1) ...
Setting up ca-certificates (20230311) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
Updating certificates in /etc/ssl/certs...
140 added, 0 removed; done.
Setting up librtmp1:amd64 (2.4+20151223.gitfa8646d.1-2+b2) ...
Setting up libssh2-1:amd64 (1.10.0-3+b1) ...
Setting up publicsuffix (20230209.2326-1) ...
Setting up libcurl4:amd64 (7.88.1-10+deb12u4) ...
Setting up curl (7.88.1-10+deb12u4) ...
Processing triggers for libc-bin (2.36-9+deb12u3) ...
Processing triggers for ca-certificates (20230311) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```


## Getting all of the pods for all namespaces:

```bash
This logged:
```bash
NAMESPACE     NAME                                       READY   STATUS             RESTARTS         AGE     IP             NODE       NOMINATED NODE   READINESS GATES
backup        backup-pod                                 1/1     Running            7 (73m ago)      19h     10.244.1.70    minikube   <none>           <none>
backup        postgres-pod                               1/1     Running            8 (72m ago)      21h     10.244.1.59    minikube   <none>           <none>
database      db-pod                                     1/1     Running            7 (73m ago)      20h     10.244.1.61    minikube   <none>           <none>
database      postgres-pod                               1/1     Running            8 (72m ago)      21h     10.244.1.66    minikube   <none>           <none>
default       nginx                                      1/1     Running            13 (73m ago)     19d     10.244.1.58    minikube   <none>           <none>
default       nginx-deployment-5f54d4d754-rzv4m          1/1     Running            12 (73m ago)     19d     10.244.1.55    minikube   <none>           <none>
default       nginx-deployment-5f54d4d754-wbntq          1/1     Running            12 (73m ago)     19d     10.244.1.45    minikube   <none>           <none>
default       nginx-deployment-5f54d4d754-xzwj6          1/1     Running            12 (73m ago)     19d     10.244.1.69    minikube   <none>           <none>
default       nginx-pod                                  1/1     Running            12 (73m ago)     19d     10.244.1.57    minikube   <none>           <none>
default       postgres-595d76d4f8-b56c4                  1/1     Running            10 (72m ago)     5d11h   10.244.1.63    minikube   <none>           <none>
dev           nginx-deployment-55f598f8d-2nw6k           1/1     Running            8 (73m ago)      21h     10.244.1.47    minikube   <none>           <none>
dev           nginx-deployment-55f598f8d-5qjsm           1/1     Running            8 (73m ago)      21h     10.244.1.68    minikube   <none>           <none>
dev           nginx-deployment-55f598f8d-szjpr           1/1     Running            8 (73m ago)      21h     10.244.1.44    minikube   <none>           <none>
kube-system   calico-kube-controllers-85578c44bf-ntjbs   1/1     Running            2 (73m ago)      119m    10.244.1.71    minikube   <none>           <none>
kube-system   calico-node-zhmjp                          1/1     Running            2 (73m ago)      119m    192.168.49.2   minikube   <none>           <none>
kube-system   coredns-5d78c9869d-csjvv                   1/1     Running            20 (72m ago)     20d     10.244.1.54    minikube   <none>           <none>
kube-system   etcd-minikube                              1/1     Running            21 (72m ago)     20d     192.168.49.2   minikube   <none>           <none>
kube-system   kube-apiserver-minikube                    1/1     Running            21 (72m ago)     20d     192.168.49.2   minikube   <none>           <none>
kube-system   kube-controller-manager-minikube           1/1     Running            21 (73m ago)     20d     192.168.49.2   minikube   <none>           <none>
kube-system   kube-proxy-868b4                           1/1     Running            20 (73m ago)     20d     192.168.49.2   minikube   <none>           <none>
kube-system   kube-scheduler-minikube                    1/1     Running            21 (72m ago)     20d     192.168.49.2   minikube   <none>           <none>
kube-system   storage-provisioner                        0/1     CrashLoopBackOff   44 (4m36s ago)   20d     192.168.49.2   minikube   <none>           <none>
namespace-a   nginx                                      1/1     Running            2 (73m ago)      119m    10.244.1.67    minikube   <none>           <none>
namespace-b   nginx                                      1/1     Running            2 (73m ago)      119m    10.244.1.65    minikube   <none>           <none>
namespace-c   postgres                                   1/1     Running            2 (72m ago)      118m    10.244.1.62    minikube   <none>           <none>
prod          nginx-deployment-55f598f8d-2wklw           1/1     Running            8 (73m ago)      21h     10.244.1.50    minikube   <none>           <none>
prod          nginx-deployment-55f598f8d-kv8lf           1/1     Running            8 (73m ago)      21h     10.244.1.60    minikube   <none>           <none>
prod          nginx-deployment-55f598f8d-tzlmd           1/1     Running            8 (73m ago)      21h     10.244.1.51    minikube   <none>           <none>
scavenger     nginx                                      1/1     Running            14 (73m ago)     19d     10.244.1.52    minikube   <none>           <none>
scavenger     nginx-deployment-55f598f8d-5dht4           1/1     Running            16 (73m ago)     20d     10.244.1.49    minikube   <none>           <none>
scavenger     nginx-deployment-55f598f8d-fgzch           1/1     Running            16 (73m ago)     20d     10.244.1.48    minikube   <none>           <none>
scavenger     nginx-deployment-55f598f8d-kdtbf           1/1     Running            16 (73m ago)     20d     10.244.1.43    minikube   <none>           <none>
scavenger     postgres                                   1/1     Running            14 (72m ago)     19d     10.244.1.53    minikube   <none>           <none>
staging       nginx-deployment-55f598f8d-5dwv5           1/1     Running            8 (73m ago)      21h     10.244.1.64    minikube   <none>           <none>
staging       nginx-deployment-55f598f8d-5gdxj           1/1     Running            8 (73m ago)      21h     10.244.1.56    minikube   <none>           <none>
staging       nginx-deployment-55f598f8d-9vgvm           1/1     Running            8 (73m ago)      21h     10.244.1.46    minikube   <none>           <none>
```

```