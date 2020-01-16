## 准备镜像

```bash
cd k8s-demo-image
docker build -t k8s-demo:0.1 .
```

## k8s 能够 pull 私有镜像

```bash
# 创建一个名为 regsecret 的 Secret :
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

> https://k8smeetup.github.io/docs/tasks/configure-pod-container/pull-image-private-registry/

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