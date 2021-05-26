# Activemq Artemis

## Introduction
Activemq Artemis is insalled using helm chart that has been slightly modified from the [original](https://github.com/vromero/activemq-artemis-helm).  Activemq Artemis runs in master-slave configuration with failover and failback features.  Persistence is enabled by default.  On AWS an internal load balancer (LB) is spawned as per settings in `values.yaml`.  The LB's address may be used by ABIS to connect to broker on port 61616.

For web console, enable access via MOSIP external facing LB via ingress.  See ingress settings in `values.yaml`

## Install
* Update `values.yaml`.  Make sure `ingress.hostname` is defined (see section below).
* Install
```
$ helm repo add mosip https://mosip.github.io/mosip-helm
$ helm repo update
$ helm -n activemq install activemq mosip/activemq-artemis -f values.yaml
```
* After successful install  update activeqm host that is assigned by the load balancer
```
$ ./cm_patch.sh
```

### Installation with Istio

* Use the same steps as above to add the helm repo.
* Use values like this (Check the end of the original values.yaml for `istioinjection` section):
  ```
  istioinjection:
    enabled: false
    hostname: activemq.xyz.net
    selector:
      istio: ingressgateway
  ```
  * These values change the istio virtual service and gateway files of activemq.
  * Change `istioinjection.enabled=true`.
  * Change the hostname: `istioinjection.hostname=activemq.so.and.so`
  * (Optional) Change the `istioinjection.selector.istio` selector field, in case, activemq is to be exposed on a different loadbalancer or different istio-ingressgateway.
* Also 61616 port is exposed in this gateway (so that other services, like abis, cli, etc, can use). But it wont work until the port 61616 is exposed in the istio-ingressgateway. Check out this document on how to do that [mosip-infra/deployment/v3/docs/istio-ingressgateway-changing-ports](https://github.com/mosip/mosip-infra/blob/develop/deployment/v3/docs/istio-ingressgateway-changing-ports.md)

## Web console
To access web console from outside cluster define a domain name like "activemq.sandbox.xyz.net". Make sure this domain points to the cluster external LB.
* Console url: `https://<activemq domain name>`.  
* Default username: `artemis`
* Password:  Run `get_pwd.sh`

## ABIS connection to broker
ABIS must connect to internal LB address over port 61616.

## CLI
Activemq command line utility may be downloaded from [here](https://activemq.apache.org/components/artemis/download/).  Note that since Activemq port 61616 is not accessible externally, you must run the same from a machine that has access to internal load balancer.