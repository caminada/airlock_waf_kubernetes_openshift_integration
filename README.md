# Introduction
To use Airlock WAF as Reverse Proxy without human interaction in a Kubernetes or OpenShift environment. 
Our recommendations are:
* set Airlock WAF in front of a Kubernetes or OpenShift environment
* use Ingress in case of Kubernetes or Route in case of OpenShift
* listen on Ingress or Route events and create a Airlock WAF configuration via REST API

![Blueprint](docs/blueprint.png)


This application listen to Ingress and Route events and build and activate a new 
Airlock WAF configuration over the Airlock WAF REST API. It lives inside a Pod in a Kubernetes Worker Node.

# Disclaimer
Please understand, this Proof of Concept application is **NOT** for production use.

# Requirements
* Airlock WAF 7.1
* Airlock WAF JWT token (API Key)
* Kubernetes or OpenShift
* Airlock WAF and Kubernetes/OpenShift need to be in same sub network

# Software Architecture Hints
* Based on [Spring Boot](https://spring.io/projects/spring-boot)
* Application Entry Point is in case of
    * Kuberentes: _IngressEventWatcher.java_
    * Openshift: _RouteEventWatcher.java_
* The official Kubernetes [Java Client](https://github.com/kubernetes-client/java) is used to communicate with the API Server
* The OpenShift [Route REST API](https://docs.openshift.com/container-platform/3.7/rest_api/apis-route.openshift.io/v1.Route.html) 
has been implemented in _OpenShiftV1Api.java_
* It use Client Certificate authentication to authenticate against the Kubernetes API Server

# Tutorial
The following chapters describe how to test the setup in a local environment. In case of Kubernetes 
it requires minikube and the ingress addon and in case of OpenShift in requires minishift. As back-end application a
http echo server will be deployed, which mirror the http client request.

## Environment
* Linux environment
* [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) incl. kubectl
* [minishift](https://docs.okd.io/latest/minishift/getting-started/installing.html) incl. oc
* curl

## Airlock WAF
1. Start minikube or minishift
1. Copy the Airlock WAF JWT token (API Key) in `src/main/resources/airlock-waf-jwt.token`
1. The default IP of the WAF host is '192.168.99.50'. To change this edit `src/main/resources/application.properties` accordingly
1. Set environment variable 'AIRLOCK_WAF_IP' in the shell which will be used for the tasks below: `AIRLOCK_WAF_IP="192.168.99.50"`

## Kubernetes
1. Clean Up Airlock WAF configuration history and import initial configuration: `./airlock-cleanup.sh --waf "${AIRLOCK_WAF_IP}" --minikube`
1. Delete previous setup: `./kubernetes-setup.sh --clean`
1. Build and deploy the event listener application: `./kubernetes-setup.sh --add-event-listener`
1. Deploy mirror back-end application: `./kubernetes-setup.sh --add-backend`
1. Enable ingress to mirror application: `./kubernetes-setup.sh --add-ingress`
1. Verify that the event listener application collect the ingress event: `./kubernetes-setup.sh --show-event-listener-logs`
1. HTTP request over the Airlock WAF: `curl -vk "${AIRLOCK_WAF_IP}:8080" -H "Host: myminikube.info"`

## OpenShift
1. Clean Up Airlock WAF configuration history and import initial configuration: `./airlock-cleanup.sh --waf "${AIRLOCK_WAF_IP}" --minishift`
1. Delete previous setup: `./openshift-setup.sh --clean`
1. Build and deploy the event listener application: `./openshift-setup.sh --add-event-listener`
1. Deploy mirror back-end application: `./openshift-setup.sh --add-backend`
1. Enable ingress to mirror application: `./openshift-setup.sh --add-route`
1. Verify that the event listener application collect the ingress event: `./openshift-setup.sh --show-event-listener-logs`
1. HTTP request over the Airlock WAF: `curl -vk "${AIRLOCK_WAF_IP}:8080" -H "Host: myminishift.info"`
