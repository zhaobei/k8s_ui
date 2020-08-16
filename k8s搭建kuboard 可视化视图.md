# k8s搭建kuboard 可视化视图

```
[root@k8s-master ~]#  docker pull eipwork/kuboard:latest
[root@k8s-master ~]#  docker images | grep kuboardeipwork/kuboard                                                   latest              9fccf2a94412        3 weeks ago         182MB
[root@k8s-master ~]#
docker save 9fccf2a94412 > kuboard.tar
```

将tar包传到k8s-node1节点上面

```
docker load < kuboard.``tar
docker tag 0146965e6475 eipwork``/kuboard``:latest
```

#### 在主节点写入如下的yaml文件---------------------------------------注意在第二十六行写你要在那个节点上面创建kuborad

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuboard
  namespace: kube-system
  annotations:
    k8s.kuboard.cn/displayName: kuboard
    k8s.kuboard.cn/ingress: "true"
    k8s.kuboard.cn/service: NodePort
    k8s.kuboard.cn/workload: kuboard
  labels:
    k8s.kuboard.cn/layer: monitor
    k8s.kuboard.cn/name: kuboard
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s.kuboard.cn/layer: monitor
      k8s.kuboard.cn/name: kuboard
  template:
    metadata:
      labels:
        k8s.kuboard.cn/layer: monitor
        k8s.kuboard.cn/name: kuboard
    spec:
      nodeName: your-node-name
      containers:
      - name: kuboard
        image: eipwork/kuboard:latest
        imagePullPolicy: IfNotPresent
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
 
---
apiVersion: v1
kind: Service
metadata:
  name: kuboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 32567
  selector:
    k8s.kuboard.cn/layer: monitor
    k8s.kuboard.cn/name: kuboard
 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kuboard-user
  namespace: kube-system
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kuboard-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kuboard-user
  namespace: kube-system
 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kuboard-viewer
  namespace: kube-system
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kuboard-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- kind: ServiceAccount
  name: kuboard-viewer
  namespace: kube-system
```

#### 运行yaml文件

```
kubectl apply -f kuboard.yaml
[root@k8s-master ~]# kubectl get pod -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
kuboard-5bc5658d74-fskmw             1/1     Running   0          21m
[root@k8s-master ~]# kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   2d11h
kuboard    NodePort    10.96.148.162   <none>        80:32567/TCP             22m
[root@k8s-master ~]#
```

#### 浏览器访问验证

![image-20200816174603542](/Users/zhaobei/Library/Application Support/typora-user-images/image-20200816174603542.png)

获取token

```
[root@k8s-master ~] echo $(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)
```

