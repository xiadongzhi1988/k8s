#
# ![图片](https://uploader.shimo.im/f/TSR0PeuhLO48FKQg.png!thumbnail)



# 部署ingress-controller

[https://blog.csdn.net/networken/article/details/85881558](https://blog.csdn.net/networken/article/details/85881558)
[https://www.cnblogs.com/dingbin/p/9754993.html](https://www.cnblogs.com/dingbin/p/9754993.html)



# 下载nginx-ingress-controller配置文件
wget [https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml)


# 修改镜像地址
image: willdockerhub/nginx-ingress-controller:0.21.0

image: 10.0.0.11:5000/wuxingge/nginx-ingress-controller:0.21.0



[root@k8s-master1 ingree-nginx]# vim mandatory.yaml 
```
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: 10.0.0.11:5000/wuxingge/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1

---
```





# 执行yaml文件部署

```
kubectl create -f mandatory.yaml
```



```
[root@k8s-master1 ingree-nginx]# kubectl get pods -o wide -n ingress-nginx 
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-ingress-controller-c9b47ff67-7pwqh   1/1     Running   0          89s   10.254.40.4   10.0.0.12   <none>           <none>
```




# nodeport方式对外提供服务
通过ingress-controller对外提供服务，现在还需要手动给ingress-controller建立一个servcie，接收集群外部流量。
# service-nodeport配置文件

wget [https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml)




vim service-nodeport.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      protocol: TCP
      nodePort: 30080
    - name: https
      port: 443
      protocol: TCP
      nodePort: 30443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```



    
    

# 执行
```
kubectl create -f service-nodeport.yaml  
```


# 查看
```
[root@k8s-master1 ingree-nginx]# kubectl get service -n ingress-nginx 
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.254.150.28   <none>        80:30080/TCP,443:30443/TCP   8s
```







&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

# 使用Ingress发布tomcat

# 准备名称空间

vi testing-namespace.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: testing
  labels:
    env: testing
```

    



```
[root@k8s-master1 ingree-nginx]# kubectl create -f testing-namespace.yaml 
namespace/testing created
[root@k8s-master1 ingree-nginx]# kubectl get namespaces testing 
NAME      STATUS   AGE
testing   Active   21s
```




# 部署tomcat实例

vi tomcat-deploy.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploy
  namespace: testing
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.0.50-jre8-alpine
        ports:
        - containerPort: 8080
          name: httpport
        - containerPort: 8009
          name: ajpport
```
          
          


```
kubectl create -f  tomcat-deploy.yaml
```


```
[root@k8s-master1 ingree-nginx]# kubectl get pods -n testing -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
tomcat-deploy-5c55c48479-mxfgv   1/1     Running   0          6m55s   10.254.88.5    10.0.0.12   <none>           <none>
tomcat-deploy-5c55c48479-wswjl   1/1     Running   0          7m40s   10.254.102.2   10.0.0.13   <none>           <none>
```




# 创建Service资源

vi tomcat-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-svc
  namespace: testing
  labels:
    app: tomcat-svc
spec:
  selector:
    app: tomcat
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```
    
    

```
[root@k8s-master1 ingree-nginx]# kubectl create -f  tomcat-svc.yaml
service/tomcat-svc created
[root@k8s-master1 ingree-nginx]# kubectl get service -n testing -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
tomcat-svc   ClusterIP   10.254.198.130   <none>        80/TCP    25s   app=tomcat
```





# 创建Ingress资源

vi tomcat-ingress.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat
  namespace: testing
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: tomcat.wuxingge.com
    http:
      paths:
      - path:
        backend:
          serviceName: tomcat-svc
          servicePort: 80
```



```
kubectl create -f  tomcat-ingress.yaml
```


          
```
[root@k8s-master1 ingree-nginx]# kubectl describe ingresses.extensions -n testing 
Name:             tomcat
Namespace:        testing
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  tomcat.wuxingge.com  
                          tomcat-svc:80 (<none>)
Annotations:
  kubernetes.io/ingress.class:  nginx
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  44s   nginx-ingress-controller  Ingress testing/tomcat
```
  


```
[root@k8s-master1 ingree-nginx]# kubectl get ingresses.extensions -n testing 
NAME     HOSTS                 ADDRESS   PORTS   AGE
tomcat   tomcat.wuxingge.com             80      76s
```





# 查看ingress-default-backend的详细信息
```
kubectl exec -n ingress-nginx -ti nginx-ingress-controller-c9b47ff67-7pwqh -- /bin/sh
$
$ cat nginx.conf
```




# hosts解析
```
[root@k8s-master1 ingree-nginx]# kubectl get pods -o wide -n ingress-nginx 
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-ingress-controller-c9b47ff67-vq9cc   1/1     Running   0          85m   10.254.88.3  ** 10.0.0.12**   <none>           <none>
[root@k8s-master1 ingree-nginx]# kubectl get service -o wide -n ingress-nginx 
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE   SELECTOR
ingress-nginx   NodePort   10.254.150.28   <none>        80:30080/TCP,443:30443/TCP   77m   app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx
```


**10.0.0.12**  tomcat.wuxingge.com




# 访问
[http://tomcat.wuxingge.com:30080/](http://tomcat.wuxingge.com:30080/)






&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&




# wordpress  ingress


# 部署MySQL
## 创建Secret对象
```
kubectl create secret generic mysql-pass --from-literal=password=Wxg@123.com
```


```
kubectl get secrets
```


## pv
```
[root@k8s-master1 wordpress]# vim mysql_nfs_pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1-nfs
  labels:
    app: pv1-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 10.0.0.11
    path: /data/db
```

## pvc
```
[root@k8s-master1 wordpress]# vim mysql_nfs_pvc.yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
```

## deployment_service
```
[root@k8s-master1 wordpress]# vim mysql-deployment.yaml 
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```




官方MySQL镜像
```
/etc/mysql/mysql.conf.d/mysqld.cnf
```




# 部署wordpress
## pv
vi wordpress_nfs_pv.yaml 
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2-nfs
  labels:
    app: pv2-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 10.0.0.11
    path: /data/web
```



## pvc
vi wordpress_nfs_pvc.yaml 
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
```



## deployment-service
vi wordpress-deployment.yaml 
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: wordpress:4.8-apache
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - name: wordpress
          containerPort: 80
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - name: wordpress
      port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: wordpress
    tier: frontend
```
    
    
    


    
## ingress    
vi wordpress-ingress.yaml 
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: wordpress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: wordpress.wuxingge.com
    http:
      paths:
      - path:
        backend:
          serviceName: wordpress
          servicePort: 80
```
## hosts解析
```
[root@k8s-master1 ingree-nginx]# kubectl get pods -o wide -n ingress-nginx 
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-ingress-controller-c9b47ff67-vq9cc   1/1     Running   0          85m   10.254.88.3   **10.0.0.12**   <none>           <none>
[root@k8s-master1 ingree-nginx]# kubectl get service -o wide -n ingress-nginx 
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE   SELECTOR
ingress-nginx   NodePort   10.254.150.28   <none>        80:30080/TCP,443:30443/TCP   77m   app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx
```
        
**10.0.0.12**  wordpress.wuxingge.com


## 访问
[http://wordpress.wuxingge.com:30080/](http://wordpress.wuxingge.com:30080/)


# ingress-nginx配置https转发dashboard

# 生成ingress-secret证书
```
root@k8s-master1 ingree-nginx]# kubectl -n kube-system  create secret tls ingress-secret --key /certs/dashboard.key --cert /certs/dashboard.crt
secret/ingress-secret created
```




# 创建ingress服务
vi k8s.yaml 
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
  annotations:
    nginx.ingress.kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  tls:
  - hosts:
    - dashboard.wuxingge.com
    secretName: ingress-secret
  rules:
    - host: dashboard.wuxingge.com
      http:
        paths:
        - path: /
          backend:
            serviceName: kubernetes-dashboard
            servicePort: 443
```



```
kubectl create -f k8s.yaml       
```
            


```
[root@k8s-master1 yaml]# kubectl get ingresses.extensions -n kube-system 
NAME                HOSTS                    ADDRESS   PORTS     AGE
dashboard-ingress   dashboard.wuxingge.com             80, 443   35s
```


```
[root@k8s-master1 yaml]# kubectl describe ingresses.extensions -n kube-system dashboard-ingress 
Name:             dashboard-ingress
Namespace:        kube-system
Address:          
Default backend:  default-http-backend:80 (<none>)
TLS:
  ingress-secret terminates dashboard.wuxingge.com
Rules:
  Host                    Path  Backends
  ----                    ----  --------
  dashboard.wuxingge.com  
                          /   kubernetes-dashboard:443 (10.254.88.4:8443)
Annotations:
  nginx.ingress.kubernetes.io/ssl-passthrough:  true
  nginx.ingress.kubernetes.io/ingress.class:    nginx
  nginx.ingress.kubernetes.io/secure-backends:  true
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  3m32s  nginx-ingress-controller  Ingress kube-system/dashboard-ingress      
```

# hosts解析
```
[root@k8s-master1 yaml]# kubectl get service -n ingress-nginx 
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.254.150.28   <none>        80:30080/TCP,443:**30443**/TCP   14h
[root@k8s-master1 yaml]# kubectl get pods -n ingress-nginx -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-ingress-controller-c9b47ff67-vq9cc   1/1     Running   1          14h   10.254.88.3   **10.0.0.12**   <none>           <none>
```


10.0.0.12  dashboard.wuxingge.com



# 访问
[https://dashboard.wuxingge.com:30443](https://dashboard.wuxingge.com:30443)
