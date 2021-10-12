# ProdElastic

This project is a template for running HA, Production-ready Elastic stack on K8S.

### Architecture

#### Local Environment

* 1 Master
* 1 Data
* 1 Client
* 1 Kibana

#### Production Environment

This is the setup I used for the production:

* 3 Master nodes
    * I used 3 to avoid split-brain & long election problems but feel free to make it more. 
* 2 Data nodes
* 2 Client nodes
* 1 Kibana
* 1 HPA component to auto scale our setup.
We also used single elasticsearch node per kubernetes host for pod affinity policy.

## Getting Started

### Pre-req

* For local development|deployment you can use KinD or Minikube.
* I used Minikube to test it but if you wanted to use [KinD](https://kind.sigs.k8s.io/) apply cluster config declared in `config.yml` with the command:
```
> kind create cluster --config config.yaml
```

### Installing

## Common steps between deploying for local & production

We begin by installing [cert-manager](https://cert-manager.io/):

```
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
```
And then creating namespace `es`:
```
$ kubectl create namespace es
```
Deploy certificate issuers by:
```
$ kubectl apply -f cert-issuer.yaml
```
And generate certificates using:
```
$ kubectl apply -f certs.yaml
```
Now after issuing certificates We deploy elastic search.
Start with master(s)
```
$ kubectl apply -f es-master.yaml
```
Then deploy data(s):
```
$ kubectl apply -f es-data.yaml
```
For the ease of use I first deploy clients with no TLS and after I generated passwords We roll-out clients with TLS based configmap:
```
$ kubectl apply -f es-client-configmap.yaml

$ kubectl apply -f es-client.yaml
```
Use this command to know if our cluster is healthy:
```
$ kubectl logs -f -n es $(kubectl get pods -n es | grep es-master | sed -n 1p | awk '{print $1}') | grep "Cluster health status changed from \[YELLOW\] to \[GREEN\]"
```
Then generate passwords with:
```
$ kubectl exec -it $(kubectl get pods -n es | grep es-client | sed -n 1p | awk '{print $1}') -n es -- bin/elasticsearch-setup-passwords auto -b
```
Now We can create a secret which holds `elastic` password:
```
$ kubectl create secret generic es-pw-elastic -n es --from-literal password=<password>
```
Enable TLS with:
```
$ kubectl apply -f es-client-configmap-ssl.yaml
```
And roll-out with:
```
$ kubectl rollout restart deployment.app/es-client -n es
```
It's time to deploy Kibana:
```
$ kubectl apply -f kibana.yaml
```
## Production environment considerations

When deploying with the HA architecture consider these things:

* You can use `Zen`,`GCE`,`EC2` or `Azure classic` discovery instead of default built-in discovery, use the example I provided in `master-ha-configmap.yaml` to have a general idea about `Zen` ping unicast and also note that if you're using hostnames your cluster should support dns.
* Set replica to 3 in `es-master.yaml`, 2 in `es-data.yaml` ,and 2 in `es-client`
* Change Kibana service configs to use `LoadBalancer` instead of `NodePort`.

* Deploy auto scaler for the clients:
```
$ kubectl apply -f autoscaler.yaml
```
* Use Let's Encrypt or other TLS providers supported by `cert-manager`.
### Verfification

Check all component are healthy using:
```
$ kubectl get all -n es
```
You can also check component logs to see if everything is running smoothly.

If everything goes well you can see you kibana dashboard in [this address](https://localhost:32159/)

Note that if you used Minikube instead of KinD, use this command to see which url to go to:
```
$ minikube service --url kibana-svc -n es
```

### To-Do

* Add memory limit to avoid starving.

## Version History

* 0.2
    * Added descriptions for local & production deployments
    * Added TLS and X-PACK security module
    * Added `Zen Discovery` configmaps


* 0.1
    * Initial Release