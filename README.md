# ProdElastic

This project is a template for running HA, Production-ready Elastic stack on K8S.

## Description

We assume that our infra layer is OpenStack IaaS therefore We use cinder as a back-end for the storage class We make.
Also services are exposed using LoadBalancer which uses Octavia(OpenStack LBaaS solution).

### Architecture

This is the setup I used for this project:

* 3 Master nodes
    * I used 3 to avoid split-brain & long election problems but feel free to make it more. 
* 2 Data nodes
* 2 Client nodes
* 1 Kibana
* 1 Elastic-HQ
* 1 HPA component to auto scale our setup.
We also used single elasticsearch node per kubernetes host for pod anti/affinity policy.

## Getting Started

### Dependencies

* OpenStack > Newton which has
    * Octavia
    * Cinder
* Magnum, Raw K8S or OpenShift on top of OpenStack.
* For local development you can use KinD or other local solutions.

### Installing

```
> kubectl|oc apply -f masters.yml
> kubectl|oc apply -f datas.yml
> kubectl|oc apply -f clients.yml
> kubectl|oc apply -f elastic-hq.yml
> kubectl|oc apply -f kibana.yml
> kubectl|oc apply -f autoscaler.yml
```

### Verfification

Check each component status using:
```
> kubectl|oc -n elasticsearch get pods -l role=<client|data|master|...>
```
You can also check component logs to see if everything is running smoothly.

### To-Do
* Implement PKI using letsencrypt & x.pack scurity module.
* Add memory limit to avoid starving.

## Version History

* 0.1
    * Initial Release
