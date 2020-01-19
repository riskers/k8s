## 准备私有镜像

```bash
cd k8s-demo-image-0.1
docker build -t k8s-demo:0.1 .
docker tag k8s-demo:0.1 riskers/k8s-demo:0.1
```

## k8s 能够 pull 私有镜像

创建一个名为 regsecret 的 Secret :

```bash
kubectl create secret docker-registry <registry-name> --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

> https://k8smeetup.github.io/docs/tasks/configure-pod-container/pull-image-private-registry/

这里我用的是:

```bash
kubectl create secret docker-registry regsecret --docker-server=locahost --docker-username=riskers --docker-password=123 --docker-email=617273330@qq.com
```

## pod

```bash
kubectl create -f pod.yaml

kubectl get pods
```

## service

Pod 是不能从外部直接访问的 (除非用 kubectl port-forward 等方案)，要把服务暴露出来给用户访问，需要创建一个服务 (Service)。Service 的作用主要是做反向代理 (Reverse Proxy) 和负载均衡 (LB)，负责把请求分发给后面的 Pod。

```bash
kubectl create -f svc.yaml

kubectl get svc k8s-demo-svc
```

访问 service:

```bash
# 1. 得到 internal-ip 地址
kubectl get nodes -o wide

# 2. 访问 31000 端口 (在 svc.yaml 中指定)
curl <internal-ip>:31000
```

## Deployment

ReplicaSet 是副本的意思，简称 RS ，实际上多使用 Deployment 。

> 一个 Deployment 拥有多个 Replica Set，而一个 Replica Set 拥有一个或多个 Pod

1. 创建 deployment:

  ```bash
  kubectl create -f deployment_0.1.yaml
  kubectl get deployments
  ```

2. 构建新镜像:

  ```bash
  cd k8s-demo-image-0.2
  docker build -t riskers/k8s-demo:0.2 .
  ```

3. 更新版本:

  ```bash
  kubectl apply -f deployment_0.2.yaml
  ```

4. 访问 `<internal-ip>:31000` : `Hello k8s!`

5. 回滚:

  ```bash
  # 查看发布历史
  kubectl rollout history deployment k8s-demo-deployment

  # 回滚 (--to-revision 指定版本)
  kubectl rollout undo deployment k8s-demo-deployment --to-revision=2
  ```

6. 缩容 / 扩容:

  ```bash
  # 缩容
  kubectl scale --replicas=2 -f deployment_0.2.yaml

  # 扩容
  kubectl scale --replicas=7 -f deployment_0.2.yaml
  ```

  这是手动的，还有一些自动方案 ([HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)) 不做介绍

## Job / CronJob

### Job

```bash
kubectl apply -f job.yaml

# 获得 Job 的 Pod 名字
kubectl get pods --selector=job-name=k8s-demo-job

NAME                 READY   STATUS      RESTARTS   AGE
k8s-demo-job-868gz   0/1     Completed   0          2m40s # STATUS是完成的，因为这是一次性的

# 查看 Pod 日志，可以看到代码中 print 的语句
kubectl logs k8s-demo-job-868gz
```

### CronJob

```bash
kubectl apply -f cronjob.yaml

kubectl get cronjobs

kubectl delete -f cronjob.yaml
```

## ConfigMap / Secret

### ConfigMap

```bash
> cd k8s-demo-configmap

> kubectl create -f config_map.yaml

> kubectl apply -f mysql.yaml

# 进入容器
> kubectl exec -it mysql-server -- /bin/bash

> root@mysql-server:/# env | grep MYSQL_SERVER_SERVICE_PORT
MYSQL_SERVER_SERVICE_PORT=3308
```

### Secret

MySQL 账号密码如果也是这种明文的方式存储很不安全。就需要用到 Secret 这种资源对象了：Secret 用来保存敏感信息，例如密码、密钥、各种 Token 等等

```bash
> kubectl apply -f secret.yaml

> kubectl get secret k8s-demo-secret --output=jsonpath='{.data}'
map[passwd:cWF6eHN3]

> kubectl exec -it mysql-server -- /bin/bash

> root@mysql-server:/# env|grep PASS
MYSQL_ROOT_PASSWORD=qazxsw
```

## Volume

