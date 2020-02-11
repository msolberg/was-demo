# was-demo
Demonstrates deploying WebSphere Traditional workloads on OpenShift.

## Overview
IBM's WebSphere Traditional Docker Images (https://hub.docker.com/r/ibmcom/websphere-traditional/) enable deploying legacy WebSphere workloads into containers on OpenShift. Once deployed, these applications function like any other workload in OpenShift and can take advantage of rolling updates, advanced deployment techniques, dynamic routing, and all of the other capabilities that OpenShift provides. This guide describes the process of building and deploying applications using the WebSphere Traditional Docker Images.

## Assessing Cloud Readiness
Before you begin to migrate a WebSphere application to the OpenShift Container Platform it's a good idea to evaluate the application to identify potential issues with run in a container. These include the following items:
* Filesystem access (i.e. writing logs to a particular directory)
* Hard-coded service naming (i.e. accessing a database via an IP address)

There are a few tools available to assist in the evaluation of an application, including the WebSphere Application Server Migration Toolkit (https://developer.ibm.com/wasdev/downloads/#asset/tools-WebSphere_Application_Server_Migration_Toolkit) and the Red Hat Application Migration Toolkit (https://developers.redhat.com/products/rhamt/overview). If your application is currently running on a version of WAS previous to 9.x, the WAS Migration Toolkit can also assist in identifying potential issues with running the application in WAS 9.x. The WAS Migration Toolkit can also evaluate whether or not a Traditional WAS application is a good candidate for running in WebSphere Liberty. There are a number of advantages to running an application in Liberty, including a much smaller image size and more modular deployment.

## OpenShift Build Strategy
The Traditional images are designed to be used as a base layer for a WebSphere application image. Full instructions on the use of the images are available on the WASdev ci.docker.websphere-traditional repository (https://github.com/WASdev/ci.docker.websphere-traditional). The images can be imported into OpenShift in one of two ways.

First, you can use the instructions provided above to create a Docker container using your application EAR file and your properties. Once the resulting application container is created, it can be pushed to an image stream in OpenShift and deployed from there. Instructions on how to push the image from a local docker registry are available at https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html.

Second, you can use a Docker build strategy to either build the application image from a Git repository or from a local file or directory. This method simplifies the process of building, pushing, and pulling the image within OpenShift. An example build configuration which will build an example application from the ci.docker.websphere-traditional repository is given below.

```
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  creationTimestamp: null
  labels:
    name: was-demo
  name: was-demo
  nodeSelector: null
spec:
  output:
    to:
      kind: ImageStreamTag
      name: was-demo:latest
  postCommit: {}
  resources: {}
  runPolicy: Serial
  source:
    git:
      uri: https://github.com/msolberg/was-demo.git
    type: Git
  strategy:
    type: Docker
    dockerStrategy:
      from:
        kind: DockerImage
        name: ibmcom/websphere-traditional:latest-ubi
  triggers:
```
[openshift/BuildConfig.yaml](openshift/BuildConfig.yaml)

For a local binary build, the `oc new-build` command can be used. Create a directory with the EAR file, the WAS configuration files, and a Dockerfile at the top level. (Or use this repository as a template.) Then, create the build config and start the build.

```
$ git clone https://github.com/msolberg/was-demo.git
$ oc new-build --strategy docker --binary --docker-image ibmcom/websphere-traditional:latest-ubi --name <APP_NAME>
$ oc start-build <APP_NAME> --from-dir was-demo --follow
```

## Deploying the application image on OpenShift
Once an application image has been created from the base image, it functions like any other image within OpenShift. To create a deployment from the image, use the `oc new-app` command and refence either the image in a remote registry or the imagestream from the build.

```
$ oc new-app was-demo/was-demo:latest
$ oc expose svc/was-demo --port 9080
```

The Dockerfile included in this repository exposes ports 9043, 9080, and 9443.

