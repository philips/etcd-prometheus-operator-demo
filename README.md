# etcd-prometheus-operator-demo

Simple demonstration of using etcd and Prometheus operators together.

Pre-requisite:
- Working [minikube and kubectl](https://github.com/kubernetes/minikube#installation)

## Deploy Operators

Launch the [etcd Operator](https://coreos.com/blog/introducing-the-etcd-operator.html) and [Prometheus Operator](https://coreos.com/blog/the-prometheus-operator.html) in the cluster.


```
kubectl create -f https://coreos.com/operators/prometheus/latest/prometheus-operator.yaml
kubectl create -f https://coreos.com/operators/etcd/latest/deployment.yaml
```

## Run the demo-cluster and monitor

Create a three member etcd cluster called "demo-cluster"

```
kubectl create -f https://github.com/philips/etcd-prometheus-operator-demo/blob/master/demo-cluster.yaml
```

Confirm it is working by finding three pods listed

```
kubectl get pods -l etcd_cluster=demo-cluster
```

Create a Prometheus service to monitor this cluster

```
kubectl create -f https://raw.githubusercontent.com/philips/etcd-prometheus-operator-demo/master/demo-cluster-monitoring.yaml
```

## Try out Prometheus

If you are using minikube simply do 

```
minikube service demo-cluster-prometheus
```

Otherwise visit the nodePort on a node IP

```
kubectl describe service demo-cluster-prometheus
```

Try out a query in prometheus like graphing the total bytes sent between peers

```
rate(etcd_network_peer_sent_bytes_total[30s])
```

And now try and delete a peer and see what happens to the graphs

```
kubectl delete pod demo-cluster-0000
```

## Cleanup

```
kubectl delete -f https://github.com/philips/etcd-prometheus-operator-demo/blob/master/demo-cluster.yaml
kubectl delete -f https://raw.githubusercontent.com/philips/etcd-prometheus-operator-demo/master/demo-cluster-monitoring.yaml
```


## Putting some keys in

```
kubectl expose pod demo-cluster-0002 --selector='etcd_cluster=demo-cluster' --name=demo-cluster --type=NodePort  -l etcd_cluster=demo-cluster  --target-port 2379
```

```
EP=$(minikube service --url demo-cluster); while true; do etcdctl --endpoints ${EP}  put hello etcd; done
```
