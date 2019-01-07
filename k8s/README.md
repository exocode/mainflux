# Deploy Mainflux on Kubernetes - WIP
Scripts to deploy Mainflux on Kubernetes (https://kubernetes.io). Work in progress. Not ready for deployment.

## Steps

### 1. Setup NATS

- To setup NATS cluster on k8s we recommend using [NATS operator](https://github.com/nats-io/nats-operator). NATS cluster should be deployed on namespace `nats-io` under the name `nats-cluster-metrics`.

```
kubectl create namespace nats-io

kubectl apply -f k8s/mainflux/service-account.yaml

kubectl -n nats-io apply -f k8s/mainflux/role.yaml

kubectl -n nats-io apply -f k8s/mainflux/deployment.yaml

kubectl -n nats-io apply -f k8s/mainflux/nats-cluster-metrics.yaml
```

### 2. Setup gRPC services Istio sidecar

- To load balance gRPC services we recommend using [Istio](https://istio.io/docs/setup/kubernetes/download-release/) sidecar. In order to use automatic inject you should run following command:

```
kubectl create -f k8s/mainflux/namespace.yml
```

<span style="color:red">Istio part has been obviated for now, Nginx still used </span>


### 3. Install Redis for MQTT

For more information see [Basic Redis tutorial](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/).

```
kubectl create namespace redis

kubectl apply -n redis -f k8s/redis/redis-master-service.yaml

kubectl apply -n redis -f redis-master-deployment.yaml
```

<span style="color:red">Slave redis hosts not created yet </span>


### 4. Setup Users service

- Deploy PostgreSQL service for Users service to use:

```
kubectl create -f k8s/mainflux/users-postgres.yml
```

- Deploy Users service:

```
kubectl create -f k8s/mainflux/users.yml
```

### 5. Setup Things service

- Deploy PostgreSQL service for Things service to use:

```
kubectl create -f k8s/mainflux/things-postgres.yml
```

- Deploy Things service:

```
kubectl create -f k8s/mainflux/things.yml
```

### 6. Setup Normalizer service

- Deploy Normalizer service:

```
kubectl create -f k8s/mainflux/normalizer.yml
```

### 7. Setup adapter services

- Deploy adapter service:

```
#kubectl create -f k8s/mainflux/tcp-services.yml
kubectl create -f k8s/mainflux/mqtt.yml
kubectl create -f k8s/mainflux/http.yml
kubectl create -f k8s/mainflux/ws.yml
```

<span style="color:red">Not sure what for tcp-services are needed </span>

### 8. Setup Dashflux

- Deploy Dashflux service:

```
kubectl create -f k8s/mainflux/dashflux.yml
```

### 9. Create Addons

- Deploy Grafana

```
kubectl create -f k8s/addons/grafana.yml
```
- Deploy InfluxDb

```
kubectl create -f k8s/addons/influxdb.yml

kubectl create -f k8s/addons/writer.yml
```
### 10. Create Nginx Reverse Proxy for Mainflux HTTP Services (mainly port 80 and 443)

- Create TLS server side certificate and keys:

```
kubectl create secret generic mainflux-secret --from-file=k8s/nginx/ssl/certs/mainflux-server.crt --from-file=k8s/nginx/ssl/certs/mainflux-server.key --from-file=k8s/nginx/ssl/dhparam.pem
```

- Create Kubernetes configmap to store NginX configuration:

```
kubectl create configmap mainflux-nginx-config --from-file=k8s/nginx/nginx.conf
```

- Deploy NginX service:

```
kubectl create -f k8s/nginx/nginx.yml
```

### 11. Create MetalLB  L2 Load Balancer to provide external access to Mainflux Services

For more information see [MetalLB L2 tutorial](https://metallb.universe.tf/tutorial/layer2/)

```
kubectl apply -f k8s/metallb/metallb.yaml

kubectl apply -f k8s/metallb/layer2-config.yaml
```

### 7. Configure Internet access
Configure NAT on your Firewall to forward ports 80 (HTTP) and 443 (HTTPS) to nginx ingress service

