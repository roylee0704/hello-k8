## run-my-nginx

An experiment to see how pods can communicate(east-west) with each other within a cluster with `Service:ClusterIP`.


### Experiment/Setup

A deployment with 2 replicas of nginx. Then, expose pods to `Service: ClusterIP` to demonstrate east-west communications. Each nginx container has port:80.

Due to volatility of IP address in Pods (they died), port is being used as the sole bridge for communication.


### Getting Started

After gotten your kubernetes cluster hook up, run following commands,

```sh
$ make init
$ make apply
$ make validate
```

###  Service: ClusterIP

This create a Service which targets TCP port 80 on any Pod with the run: roy-nginx label.
```sh
$ kubectl -n roy-nginx expose deployment/roy-nginx
```


For validation,
```sh
$ kubectl -n roy-nginx get svc roy-nginx
$ kubectl -n roy-nginx get svc roy-nginx -ojsonpath='{.spec.clusterIP}'
```


#### East-West Communication

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