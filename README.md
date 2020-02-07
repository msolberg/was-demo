# was-demo
Demonstrates deploying WebSphere Traditional workloads on OpenShift.

## Instructions
This demo assumes that you have an existing EAR file (or directory) to deploy onto WebSphere. To use, first clone this repository:

```
$ git clone https://github.com/msolberg/was-demo.git
```

In OpenShift, create a new project:

```
$ oc new-project was-demo
```

Create the ImageStream for was-demo

```
$ oc create -f openshift/ImageStream.yaml
```

Create the BuildConfig for was-demo

```
$ oc create -f openshift/BuildConfig.yaml
```

Run a build.

```
$ oc start-build was-demo --follow
```

Create the DeploymentConfig

```
$ oc create -f openshift/DeploymentConfig.yaml
```

Create the Service

```
$ oc create -f openshift/Service.yaml
```

Create the Route

```
$ oc expose svc/was-demo
```

Test the Route:

```
$ oc get route
$ curl <URL from previous command>/HelloWorld/hello
```


