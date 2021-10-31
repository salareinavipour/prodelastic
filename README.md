# ProdElastic

This is an Ansible based Kubernetes Operator that deploys an HA, production-ready instance of Elastic Stack.

## Architecture

* 3 Master
* 2 Data
* 2 Client
* 1 Kibana
* 1 HPA component to auto scale our setup.

I also used single elasticsearch node per kubernetes host for pod affinity policy.

### Pre-req

* For local development|deployment you can use KinD or Minikube.
* I used Minikube to test it but if you wanted to use [KinD](https://kind.sigs.k8s.io/) apply cluster config declared in `config.yml` with the command:
```
> kind create cluster --config config.yaml
```

## Getting Started

<!-- To be written. -->

### Production environment considerations

When deploying with the HA architecture consider these things:

* You can use `Zen`,`GCE`,`EC2` or `Azure classic` discovery instead of default built-in discovery, use the example I provided in `master-ha-configmap.yaml` to have a general idea about `Zen` ping unicast and also note that if you're using hostnames your cluster should support dns.

### Verfification

<!-- To be written. -->

### To-Do

* Add memory limit to avoid starving.
* Kibana ansible role is done, complete ElasticStack role.

## Version History

* 0.3
    * Made the whole project an Ansible Kubernetes Operator

* 0.2
    * Added descriptions for local & production deployments
    * Added TLS and X-PACK security module
    * Added `Zen Discovery` configmaps

* 0.1
    * Initial Release