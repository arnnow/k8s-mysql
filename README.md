# Setup a Mysql in kubernetes
**For test purposes using [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)**

## Configure PersistentVolume & PersistentVolumeClaim
Set Volume using [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath).  

You can set the [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes) & [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).
```bash
{21:11}~/k8s-nginx-php:master ✗ ➭ kubectl apply -f definitions/storages/mysql_pv_volume.yaml
persistentvolume/mysql-pv-volume created
{21:11}~/k8s-nginx-php:master ✗ ➭ kubectl apply -f definitions/storages/mysql_pv_claim.yaml
persistentvolumeclaim/mysql-pv-claim created

{21:12}~/k8s-nginx-php:master ✗ ➭ kubectl get pv,pvc -o wide
NAME                               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE   VOLUMEMODE
persistentvolume/mysql-pv-volume   1Gi        RWO            Retain           Bound    default/mysql-pv-claim   manual                  63s   Filesystem

NAME                                   STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
persistentvolumeclaim/mysql-pv-claim   Bound    mysql-pv-volume   1Gi        RWO            manual         57s   Filesystem

```

## Set Secrets
[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) are generated using base64
```bash
{20:52}~ ➭ echo -n 'thisissecure' | base64
dGhpc2lzc2VjdXJl
```
You can add your base64 encoded secret to the yaml file and apply it
*This is clearly not secure and need a proper secret management tool*
```bash
{21:12}~/k8s-nginx-php:master ✗ ➭ kubectl apply -f definitions/secrets/mysql_secret.yaml
secret/mysql-secrets created

{21:13}~/k8s-nginx-php:master ✗ ➭ kubectl get secrets -o wide
NAME                  TYPE                                  DATA   AGE
mysql-secrets         Opaque                                1      9s

```

## Set [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
```bash
{21:15}~/k8s-nginx-php:master ✗ ➭ kubectl apply -f definitions/mysql_service.yaml
service/mysql created

{21:15}~/k8s-nginx-php:master ✗ ➭ kubectl get svc -o wide
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        6m57s   <none>
mysql        ClusterIP      10.111.20.106   <none>        3306/TCP       33s     app=mysql,tier=backend

```

## Launch [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
```bash
{21:16}~/k8s-nginx-php:master ✗ ➭ kubectl apply -f definitions/mysql_deployment.yaml
deployment.apps/mysql created

{21:16}~/k8s-nginx-php:master ✗ ➭ kubectl get deployments -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES        SELECTOR
mysql   0/1     1            0           30s   mysql        mysql:5.6     app=mysql,tier=backend
```

# Checking your deployments
## Checking on your pods
```
{21:18}~/k8s-nginx-php:master ✗ ➭ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
mysql-575c4778f5-jfzbk   0/1     ContainerCreating   0          2m35s
```
## Checking Mysql
```
{21:22}~/Seafile/Priv/Soft/k8s/k8s-nginx-php:master ✗ ➭ kubectl describe pods mysql-575c4778f5-jfzbk
Name:         mysql-575c4778f5-jfzbk
Namespace:    default
Priority:     0
Node:         minikube/192.168.99.107
Start Time:   Tue, 11 Feb 2020 21:16:28 +0100
Labels:       app=mysql
              pod-template-hash=575c4778f5
              tier=backend
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/mysql-575c4778f5
Containers:
  mysql:
    Container ID:   docker://1ce287bd0490f0a8b367dc3bda5a1a9e9f2d4209b330cc64039a14e9225199fe
    Image:          mysql:5.6
    Image ID:       docker-pullable://mysql@sha256:bef096aee20d73cbfd87b02856321040ab1127e94b707b41927804776dca02fc
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 11 Feb 2020 21:21:44 +0100
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'ROOT_PASSWORD' in secret 'mysql-secrets'>  Optional: false
    Mounts:
      /var/lib/mysql from mysql-pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5trjb (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  mysql-pv-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-pv-claim
    ReadOnly:   false
  default-token-5trjb:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-5trjb
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m44s  default-scheduler  Successfully assigned default/mysql-575c4778f5-jfzbk to minikube
  Normal  Pulling    5m44s  kubelet, minikube  Pulling image "mysql:5.6"
  Normal  Pulled     28s    kubelet, minikube  Successfully pulled image "mysql:5.6"
  Normal  Created    28s    kubelet, minikube  Created container mysql
  Normal  Started    28s    kubelet, minikube  Started container mysql
```
The mysql pod has been started.
Data should be in 
```bash
{21:21}~/k8s-nginx-php:master ✗ ➭ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ls -l /tmp/mysql/
total 110596
-rw-rw---- 1 999 999       56 Feb 11 20:21 auto.cnf
-rw-rw---- 1 999 999 50331648 Feb 11 20:21 ib_logfile0
-rw-rw---- 1 999 999 50331648 Feb 11 20:21 ib_logfile1
-rw-rw---- 1 999 999 12582912 Feb 11 20:21 ibdata1
drwx------ 2 999 999     1620 Feb 11 20:21 mysql
drwx------ 2 999 999     1100 Feb 11 20:21 performance_schema
```
Testing our Mysql pod
```
{21:33}~/k8s-nginx-php:master ✗ ➭ kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pthisisnotsecure
If you don't see a command prompt, try pressing enter.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
```

# Load testing Mysql using mysqlslap
## Creating the image
We will here create a docker image via a dockerfile and use it directly in minikube without pushing it onto a repository
the dockerfile is here
```bash
# Set docker env
{22:17}~/k8s-nginx-php/loadtest/my_mysqlslap:master ✗ ➭ eval $(minikube docker-env)

{22:17}~/k8s-nginx-php/loadtest/my_mysqlslap:master ✗ ➭ docker build -t mysqlslap:v1 .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM debian:buster
...
Successfully built 72d2ca4d287b
Successfully tagged mysqlslap:v1
```
Your image should be present in minikube
```bash
{22:23}~/Seafile/Priv/Soft/k8s/k8s-nginx-php:master ✗ ➭ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker image ls | grep mysqlsl
mysqlslap                                 v1                  72d2ca4d287b        2 minutes ago       212MB
```

## Importing the [employee](https://dev.mysql.com/doc/index-other.html) database & Running the test
```
# Run in minikube
kubectl apply -f loadtest/my_mysqlslap/mysqlslap_job.yaml

# Check that it's running
kubectl get jobs,pods

# Checking the logs
kubectl logs -f pod/mysqlslap-xxxx
```
