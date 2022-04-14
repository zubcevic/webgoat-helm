# Helm chart deployment on OpenShift K8S clusters

This helm chart can be used on a OpenShift Code Ready Container environment or an OpenShift Cloud Container environment.

With the OpenShift CRC (Code Ready Container) cluster you run an entire environment on your local machine. (> 4 vCPU, >8GB mem)

See the Red Hat documentation for general understanding of OpenShift. Make sure helm is installed as well.

https://developers.redhat.com/developer-sandbox

## CRC commands

    crc config set cpus 6
    crc config set memory 12288
    crc setup
    crc start
    eval $(crc oc-env)
    oc login -u developer https://api.crc.testing:6443
    oc new-project demo-project
    crc console

The example without modification uses *demo-project* as the project/namespace for installing WebGoat and WebWolf.


## Helm install this example on your local Code Ready Container environment

    helm install goat1 ./webgoat
    helm install modsec1 ./modsec

## Helm install on single node Developer Sandbox (cloud)

    oc login --token=sha256~phDWy6Wm_oJQW6kmOHEbLkRdDIXU6b70hRVmdSYWolM --server=https://api.sandbox-m2.rz9k.p1.openshiftapps.com:6443 
    helm install --set namespace=renezubcevic-dev --set accessMode=ReadWriteOnce --set urlpostfix=.apps.sandbox-m2.rz9k.p1.openshiftapps.com goat1 ./webgoat

A code ready container looks the same for all developers on their local machine, but a developer sandbox requires other credentials from your account in the cloud and different namespace and urlpostfix and also a different access mode for the persistent storage.
Of course the token here is a fake.

## uninstall 

    helm uninstall goat1

The URL on a Code Ready Container is build from router name + namespace + default extension .apps-crc.testing:

+ [https://webgoat-1-goat-demo-project.apps-crc.testing/WebGoat/login](https://webgoat-1-goat-demo-project.apps-crc.testing/WebGoat/login)
+ [http://webgoat-1-wolf-demo-project.apps-crc.testing/WebWolf](http://webgoat-1-wolf-demo-project.apps-crc.testing/WebWolf)

+ [http://modsec-1-modsec-demo-project.apps-crc.testing/WebGoat/login](http://modsec-1-modsec-demo-project.apps-crc.testing/WebGoat/login)

## Explanation

Deployment.yaml describes how the container image is used. It refers to Persistent Volume Claim and use the same Volume mapping. 
The java.io.dir is also mapped to this persistent volume mapping. The number of pods is set to 1. References from WebGoat to WebWolf are set in the configmap yaml file as environment properties and as container image start up arguments.

persistent-storage-claim.yaml contains the OpenShift K8S extension for requestig a volume with Read-Write access that will survive any pod replacements.

service.yaml defines the service ports for both WebGoat and WebWolf

route-goat defines an https endpoint toward the 8080 port and an http port towards the 9090 port.

Additionally you can install the modsec Helm Chart in the same name space which acts as an application firewall which forwards internally to WebGoat. On the WebGoat through Modsec URL you will not be able to do SQL injection for example.
