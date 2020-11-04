# Services

An experiment to see how pods can perform communication with other pods (east-west) and external world (north-west). 

This experiment will be conducted over 3 types of services offered in k8s:

- ClusterIP. Default service. It only provides east-west communication among pods across nodes/ec2 in the cluster.
- LoadBalancer. An abstraction of `NodePort` that hides node-port ranges `:30000` to`:32767` with `port:80`. Lives outside of node/ec2 cluster.
- NodePort. The simplest/cheapest way to establish north-south communication.

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

Expose pods to `Service: ClusterIP` to demonstrate east-west communications. Each nginx container has port:80.


This create a Service which targets TCP port 80 on any Pod with the run: roy-nginx label.
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

Currently Service doesn't have external IP, so lets now patch the Service to use a cloud load balancer, by updating the type of roy-nginx Service from `ClusterIP` to `LoadBalancer`.


```sh
$ kubectl -n roy-nginx patch svc roy-nginx -p '{"spec": {"type": "LoadBalancer"}}'
```



## References
- https://www.eksworkshop.com/beginner/130_exposing-service/connecting/