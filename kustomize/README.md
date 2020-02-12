# Tryout for kustomize
Here we try kustomize templating

```
{22:21}~/k8s-mysql/kustomize/env/prod:master ✗ ➭ kubectl apply -k ./
secret/prod-mysecrets-6dg4g995m8 created
service/prod-mysql created
deployment.apps/prod-mysql created
persistentvolume/prod-mysql-pv-volume created
persistentvolumeclaim/prod-mysql-pv-claim created

{22:28}~/k8s-mysql/kustomize/env/prod:master ✗ ➭ cd ../dev

{22:28}~/k8s-mysql/kustomize/env/dev:master ✗ ➭ kubectl apply -k ./
secret/dev-mysecrets-bbb45452ht created
service/dev-mysql created
deployment.apps/dev-mysql created
persistentvolume/dev-mysql-pv-volume created
persistentvolumeclaim/dev-mysql-pv-claim created
{22:28}~/k8s-mysql/kustomize/env/dev:master ✗ ➭
```
Runing in minikube
```
{22:28}~/Seafile/Priv/Soft/k8s/k8s-mysql:master ✗ ➭ kubectl get pods,svc,deployment,secret,job,pv,pvc -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod/dev-mysql-7b9b4b9cb8-mvggh   0/1     Error     2          66s   172.17.0.5   minikube   <none>           <none>
pod/prod-mysql-bf9cfcf68-26bgs   1/1     Running   3          73s   172.17.0.4   minikube   <none>           <none>

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
service/dev-mysql    ClusterIP   10.107.24.88     <none>        3306/TCP   66s   app=mysql,tier=backend
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    78m   <none>
service/prod-mysql   ClusterIP   10.102.119.110   <none>        3306/TCP   74s   app=mysql,tier=backend

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES      SELECTOR
deployment.apps/dev-mysql    0/1     1            0           67s   mysql        mysql:5.6   app=mysql,tier=backend
deployment.apps/prod-mysql   1/1     1            1           75s   mysql        mysql:5.6   app=mysql,tier=backend

NAME                               TYPE                                  DATA   AGE
secret/default-token-dnvkc         kubernetes.io/service-account-token   3      78m
secret/dev-mysecrets-bbb45452ht    Opaque                                1      67s
secret/prod-mysecrets-6dg4g995m8   Opaque                                1      75s

NAME                                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS
REASON   AGE   VOLUMEMODE
persistentvolume/dev-mysql-pv-volume    1Gi        RWO            Retain           Bound    default/dev-mysql-pv-claim    manual
67s   Filesystem
persistentvolume/prod-mysql-pv-volume   1Gi        RWO            Retain           Bound    default/prod-mysql-pv-claim   manual
74s   Filesystem

NAME                                        STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
persistentvolumeclaim/dev-mysql-pv-claim    Bound    dev-mysql-pv-volume    1Gi        RWO            manual         67s   Filesystem
persistentvolumeclaim/prod-mysql-pv-claim   Bound    prod-mysql-pv-volume   1Gi        RWO            manual         74s   Filesystem
```
Delete everything
```
{22:30}~/k8s-mysql/kustomize/env/dev:master ✗ ➭ kubectl delete -k ./
secret "dev-mysecrets-bbb45452ht" deleted
service "dev-mysql" deleted
deployment.apps "dev-mysql" deleted
persistentvolume "dev-mysql-pv-volume" deleted
persistentvolumeclaim "dev-mysql-pv-claim" deleted

{22:30}~/k8s-mysql/kustomize/env/dev:master ✗ ➭ cd ../prod
{22:31}~/k8s-mysql/kustomize/env/prod:master ✗ ➭ kubectl delete -k ./
secret "prod-mysecrets-6dg4g995m8" deleted
service "prod-mysql" deleted
deployment.apps "prod-mysql" deleted
persistentvolume "prod-mysql-pv-volume" deleted
persistentvolumeclaim "prod-mysql-pv-claim" deleted
```
