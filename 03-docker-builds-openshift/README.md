##  Performing a docker build within OpenShift

This is extending the material in exercise 1 (building from base container). There a base image was provided in box, and you
performed a docker build using that base image in your local machine. Here, that base image that was provided in exercise 1
will be built within OpenShift. The provided Dockerfile will help you with that.

To execute this exercise, clone this git repo to your local disk and then use that to create a publicly visible git rep on
github.com (the public one, not the IBM enterprise git repo). It is possible to get this working from the IBM enterprise git
repo, but that is a separate excercise. After this code is in the github.com repo, run the following instructions:

```
oc new-app --name=simple-stuff https://github.com/<your github.com id>/docker-builds-openshift.git --as-deployment-config
oc create route edge --service=simple-stuff
```

And then, `curl -k https://<hostname for route>/simple-stuff/simple/simon`. This should produce the string "/my-special-folder does not exist"

## Task

0. Delete all existing artifiacts from the previous steps. 
```
zaphod:sa-bootcamp$ oc delete all -l app=simple-stuff
service "simple-stuff" deleted
deployment.apps "simple-stuff" deleted
buildconfig.build.openshift.io "simple-stuff" deleted
build.build.openshift.io "simple-stuff-1" deleted
imagestream.image.openshift.io "simple-stuff" deleted
imagestream.image.openshift.io "websphere-liberty" deleted
route.route.openshift.io "simple-stuff" deleted
zaphod:sa-bootcamp$ 
```

1. In the docker build, create a directory called /my-special-folder. Copy the Dockerfile in this git repo into that folder. Note that you will need to create this
folder as the root user, otherwise, your creation of the directory will fail during the build.

A Dockerfile.answer file is provided in this git repo which shows how this step is done.

2. Repeat the new-app, and "create route" commands provided at the top of this file.
```
zaphod:sa-bootcamp$ oc new-app --name=simple-stuff https://github.com/kstephen314159/day3-simple-stuff.git
--> Found container image 1db9786 (16 months old) from docker.io for "docker.io/websphere-liberty:javaee8"

    * An image stream tag will be created as "websphere-liberty:javaee8" that will track the source image
    * A Docker build using source code from https://github.com/kstephen314159/day3-simple-stuff.git will be created
      * The resulting image will be pushed to image stream tag "simple-stuff:latest"
      * Every time "websphere-liberty:javaee8" changes a new build will be triggered

--> Creating resources ...
    imagestream.image.openshift.io "websphere-liberty" created
    imagestream.image.openshift.io "simple-stuff" created
    buildconfig.build.openshift.io "simple-stuff" created
    deployment.apps "simple-stuff" created
    service "simple-stuff" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/simple-stuff' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/simple-stuff' 
    Run 'oc status' to view your app.
zaphod:sa-bootcamp$ oc create route edge --service=simple-stuff
route.route.openshift.io/simple-stuff created
zaphod:sa-bootcamp$
```
If you are using this repo without changes (i.e you haven't copied Dockerfile.answer on top of Dockerfile, or merged the two), then you will 
have to perform some additional steps. Perform the following command:
```
oc edit bc/simple-stuff
```
This will open a "vi" sesssion. Scroll down in the file until you come to the "dockerStrategy" element. Underneath it, as a child, add the following: "dockerfilePath: Dockerfile.answer". This is what that section of the yaml file will look like when you are done:
```
  source:
    git:
      uri: https://github.com/kstephen314159/docker-builds-openshift-answer.git
    type: Git
  strategy:
    dockerStrategy:
      dockerfilePath: Dockerfile.answer
      from:
        kind: ImageStreamTag
        name: websphere-liberty:20.0.0.5-full-java11-openj9-ubi
    type: Docker
```
After you have done this, restart the build by saying "oc start-build simple-stuff". This new build will build using the "Dockerfile.answer" dockerfile.
```
oc create route edge --service=simple-stuff
```
3. Repeat steps 1 and 2, but this time, call the app "even-simpler". What you have done with this step is to use the same git repo to create two different running instances (or applications) 
of the code.
```
zaphod:docker-builds-openshift-answer$ oc new-app --name=even-simpler https://github.com/kstephen314159/docker-builds-openshift-answer.git
--> Found container image 3e170c7 (8 months old) from docker.io for "docker.io/ibmcom/websphere-liberty:20.0.0.5-full-java11-openj9-ubi"

    Red Hat Universal Base Image 8 
    ------------------------------ 
    The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.

    Tags: base rhel8

    * An image stream tag will be created as "websphere-liberty:20.0.0.5-full-java11-openj9-ubi" that will track the source image
    * A Docker build using source code from https://github.com/kstephen314159/docker-builds-openshift-answer.git will be created
      * The resulting image will be pushed to image stream tag "even-simpler:latest"
      * Every time "websphere-liberty:20.0.0.5-full-java11-openj9-ubi" changes a new build will be triggered

--> Creating resources ...
    imagestream.image.openshift.io "even-simpler" created
    buildconfig.build.openshift.io "even-simpler" created
    deployment.apps "even-simpler" created
    service "even-simpler" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/even-simpler' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/even-simpler' 
    Run 'oc status' to view your app.
zaphod:docker-builds-openshift-answer$ oc edit bc/even-simpler
buildconfig.build.openshift.io/even-simpler edited
zaphod:docker-builds-openshift-answer$ oc create route edge --service=simple-stuff
```

5. Look for the image stream corresponding to "even-simpler". Use the "new-app" command and point to this imagestream and create a third instance of the application. Call this application
instance "simplest-of-all".
```
zaphod:docker-builds-openshift-answer$ oc get is
NAME                IMAGE REPOSITORY                                                                TAGS                              UPDATED
a-derived-image     image-registry.openshift-image-registry.svc:5000/kstephe-us/a-derived-image     latest,3.0,2.0,1.0                6 days ago
derived-app         image-registry.openshift-image-registry.svc:5000/kstephe-us/derived-app         latest                            
even-simpler        image-registry.openshift-image-registry.svc:5000/kstephe-us/even-simpler        latest                            About a minute ago
simple-stuff        image-registry.openshift-image-registry.svc:5000/kstephe-us/simple-stuff        latest                            16 minutes ago
websphere-liberty   image-registry.openshift-image-registry.svc:5000/kstephe-us/websphere-liberty   20.0.0.5-full-java11-openj9-ubi   19 minutes ago
zaphod:docker-builds-openshift-answer$ oc new-app --name=simplest-of-all even-simpler
--> Found image cb9942b (2 minutes old) in image stream "kstephe-us/even-simpler" under tag "latest" for "even-simpler"

    Red Hat Universal Base Image 8 
    ------------------------------ 
    The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications, middleware and utilities. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly.

    Tags: base rhel8


--> Creating resources ...
    imagestreamtag.image.openshift.io "simplest-of-all:latest" created
    deployment.apps "simplest-of-all" created
    service "simplest-of-all" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/simplest-of-all' 
    Run 'oc status' to view your app.
zaphod:docker-builds-openshift-answer$
zaphod:docker-builds-openshift-answer$ oc create route edge --service=simplest-of-all
route.route.openshift.io/simplest-of-all created
```
With this step, you have created a third instance of the application, but this time, rather than it being built from source, you are using the already built image which is being referenced by the image stream, and using that.

6. Provide the output of the "curl" command shown above for the "simple", "even-simpler", and "simplest-of-all" apps.
Example, below, shows the output of "simplest-of-all", but the outputs of the other two should be the same:
```
zaphod:docker-builds-openshift-answer$ curl -k https://simplest-of-all-kstephe-us.ose-bootcamp-1612539632-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/simple-stuff/simple/simon
FROM docker.io/ibmcom/websphere-liberty:20.0.0.5-full-java11-openj9-ubi
COPY target/simple-stuff.war /config/dropins/
COPY config/server.xml /config/
COPY config/server.env /config/
```
