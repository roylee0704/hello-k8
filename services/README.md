# Services

An experiment to see how pods can perform communication with other pods (east-west) and external world (north-west). 

This experiment will be conducted over 3 types of services offered in k8s:

- **Service: ClusterIP**. Default service if you do not specify `--type=NodePort` flag on commands `kubectl expose ...`. It only provides east-west communication among pods across nodes/ec2 in the cluster.
- **Service: LoadBalancer**. An abstraction of `NodePort` that hides node-port ranges `:30000` to`:32767` with `port:80`. Lives outside of node/ec2 cluster.
- **Service: NodePort**. The simplest/cheapest way to establish north-south communication.

tips: Due to volatility of IP address in Pods (they died), port is being used as the sole bridge for communication.

## Getting Started

A deployment with 2 replicas of nginx.

After gotten your kubernetes cluster hook up, run following commands,

```sh
$ make init
$ make apply
$ make validate
```

## 1. Service: ClusterIP

Expose pods to `Service: ClusterIP` to demonstrate east-west communications. Each nginx container has `port:80`.


Following commands creates a `Service: ClusterIP` that targets any Pods within ec2/node cluster with `label:roy-nginx` and `port:80` (information obtained from `deployment/roy-nginx`).

```sh
$ kubectl -n roy-nginx expose deployment/roy-nginx
```


For validation,
```sh
$ kubectl -n roy-nginx get svc roy-nginx
$ kubectl -n roy-nginx get svc roy-nginx -ojsonpath='{.spec.clusterIP}'
```


### East-West Communication

`Service:ClusterIP` IP is completely virtual. It never hits the wire.

Run following scripts to spawn a new Pod that hosts busybox container.
```sh
# Create a variable set with the my-nginx service IP
$ export MyClusterIP=$(kubectl -n roy-nginx get svc roy-nginx -ojsonpath='{.spec.clusterIP}')

# Create a new deployment and allocate a TTY for the container in the pod
$ kubectl -n roy-nginx run -i --tty load-generator --env="MyClusterIP=${MyClusterIP}" --image=busybox /bin/sh


# Connecting to the nginx welcome page using the ClusterIP.
$# wget -q -O - ${MyClusterIP} | grep '<title>'
```


## 2. Services: Load Balancer

Currently Service doesn't have external/public IP, so lets now patch the Service to use a cloud load balancer, by updating the type of roy-nginx Service from `ClusterIP` to `LoadBalancer`.


```sh
$ kubectl -n roy-nginx patch svc roy-nginx -p '{"spec": {"type": "LoadBalancer"}}'
```

### North-South Communication

There are two ways of reaching the Pod from external world:
- via `Service:LoadBalancer`'s public IP or DNS.
- via `Service:NodePort`. (for instructions, check section below `Services: Node Port`)


Following commands showcasing accessing Pod via `Service:LoadBalancer`.
```sh
# extract loadbalancer public-ip
$ export loadbalancer=$(kubectl -n my-nginx get svc my-nginx -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')

# curl it.
$ curl -k -s http://${loadbalancer}:80 | grep title
```


## 3. Services: Node Port

Let's patch it to `NodePort`.

```sh
$ kubectl -n roy-nginx patch svc roy-nginx -p '{"spec": {"type": "NodePort"}}'
```

### North-South Communication

There only way to access the Pod is via Node's public IP address. However, for security reason, that is not doable unless you changed the node's firewall (security-group) to allow ingress access of port-range `:30000` to`:32767`.

If you're lazy and want quick result, you can use a technique called [port-forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

```sh
# kubectl port-forward your-unique-pod-name your-local-machine-port:node-high-port
$ kubectl port-forward pod/roy-nginx 4444:35200
```

However, if you want a long-term solution, go ahead and add a new TCP port-range of `:30000` to`:32767` on your nodes/ec2s in the cluster. Then you may access the pod with `https://node-public-ip:node-high-port`.

## References
- https://www.eksworkshop.com/beginner/130_exposing-service/connecting/
- https://medium.com/faun/learning-kubernetes-by-doing-part-3-services-ed5bf7e2bc8e