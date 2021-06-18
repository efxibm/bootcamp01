# working-with-pods

1.  Set the "latest" tag for the imagestream that was created in the previous exercise.
```
oc tag a-derived-image:1.0 a-derived-image:latest
```
2.  Use the "oc new-app" command to deploy the container referenced by the imagestream. Specify the "latest" tag. Use the "--as-deployment-config" flag in order to have the DeploymentConfig resource get created. Specify the name of the "app" as "derived-app".
```
oc new-app --name derived-app a-derived-image:latest --as-deployment-config
```
3.  Run <code>oc get pods</code> to validate that the pod is running. Use <code>oc logs -f <pod name\></code> to validate that the application has started up.
```
zaphod:working-with-pods-answer$ oc get pods
NAME                   READY   STATUS      RESTARTS   AGE
derived-app-1-deploy   0/1     Completed   0          31m
derived-app-1-lb9l9    1/1     Running     0          31m
zaphod:working-with-pods-answer$ oc logs -f derived-app-1-lb9l9

Launching defaultServer (WebSphere Application Server 20.0.0.5/wlp-1.0.40.cl200520200429-1655) on Eclipse OpenJ9 VM, version 11.0.7+10 (en_US)
[AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
[AUDIT   ] CWWKE0100I: This product is licensed for development, and limited production use. The full license terms can be viewed here: https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/wasdev/license/base_ilan/ilan/20.0.0.5/lafiles/en.html
[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ibm/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
[AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
[AUDIT   ] CWWKS4104A: LTPA keys created in 2.403 seconds. LTPA key file: /opt/ibm/wlp/output/defaultServer/resources/security/ltpa.keys
[AUDIT   ] CWPKI0803A: SSL certificate created in 4.098 seconds. SSL key file: /opt/ibm/wlp/output/defaultServer/resources/security/key.p12
[AUDIT   ] CWWKI0001I: The CORBA name server is now available at corbaloc:iiop:localhost:2809/NameService.
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[WARNING ] getInjectionClasses 
                                 WebAppConfig was null
[AUDIT   ] CWWKT0016I: Web application available (default_host): http://derived-app-1-lb9l9:9080/simple-stuff/
[AUDIT   ] CWWKZ0001I: Application simple-stuff started in 1.664 seconds.
[AUDIT   ] CWWKF0012I: The server installed the following features: [localConnector-1.0].
[AUDIT   ] CWWKF0013I: The server removed the following features: [json-1.0, jwt-1.0, microProfile-3.0, mpConfig-1.3, mpFaultTolerance-2.0, mpHealth-2.0, mpJwt-1.1, mpMetrics-2.0, mpOpenAPI-1.1, mpOpenTracing-1.3, mpRestClient-1.3, opentracing-1.3].
[AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 12.997 seconds.
```
4.  Use the command <code>oc create route edge --service=derived-app</code> in order to create a route which is edge terminated. View this using the "oc get" command and use that output to figure out the hostname of the route. Use the command <code>curl -k https://<route hostname\>/simple-stuff/simple/simon</code> to invoke this application.
```
zaphod:working-with-pods-answer$ oc create route edge --service=derived-app
route.route.openshift.io/derived-app created
zaphod:working-with-pods-answer$ oc get route derived-app
NAME          HOST/PORT                                                                                                               PATH   SERVICES      PORT       TERMINATION   WILDCARD
derived-app   derived-app-kstephe-us.ose-bootcamp-1612539632-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud          derived-app   9080-tcp   edge          None
zaphod:working-with-pods-answer$ curl -k https://derived-app-kstephe-us.ose-bootcamp-1612539632-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/simple-stuff/simple/simon
FROM localhost/a-base-image:1.0

USER root
RUN mkdir /my-special-folder 
COPY Dockerfile /my-special-folder
```
5.  Use the Dockerfile in this git repo to create a new build of the image. Create it using the tag a-derived-image:2.0.

First change to the directory containing the Dockerfile and then run the following command:
```
docker build -t a-derived-image:2.0 .
```
6.  Following the same procedure used in the previous exercise, push this image to the openshift registry. Use the 2.0 tag for the destination.
```
TOKEN=`oc whoami -t`
podman login -u kstephe -p ${TOKEN} https://default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud
podman push localhost/a-derived-image:2.0 default-route-openshift-image-registry.ose-bootcamp-1612539632-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:2.0
```
Note that with the docker cli, this single step push won't work. Instead, you will have to tag the image first, and then push:
```
docker tag localhost/a-derived-image:2.0 default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:2.0
docker push default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:2.0
```
Note also that in the case of the "docker tag" command, above, there are reports that this fails with some versions of docker. Please remove the "localhost/" in the local image tag and retry if you get a failure.

7.  Look at the pods on your system. Notice that nothing has changed since the deployment.
```
oc get pods
```
8.  Set the latest tag (like you did in step 1) on the newest image in the imagestream. Now look at the pods. You should see a new deployment in progress. When its done, at the pod has completed startup, re-run the curl command to validate that the new image that was pushed is running.
```
oc tag a-derived-image:2.0 a-derived-image:latest
oc get pods
```
9.  Use the "oc scale" command to increase the number of pod instances to 3. Examine the DeploymentConfig object and confirm that the number of replicas is set to 3.
```
zaphod:working-with-pods-answer$ oc scale dc/derived-app --replicas=3
deploymentconfig.apps.openshift.io/derived-app scaled

zaphod:~$ oc get pods
NAME                   READY   STATUS      RESTARTS   AGE
derived-app-1-4rwtm    1/1     Running     0          6d
derived-app-1-68p75    1/1     Running     0          6d
derived-app-1-deploy   0/1     Completed   0          6d1h
derived-app-1-lb9l9    1/1     Running     0          6d1h
zaphod:~$ 
```
The following command shows the DeploymentConfig in json format, and then uses the jq tool to extract the "spec" section. You can see the equivalent of this output in yaml by specifying "-o yaml" and then paging through the result.
```
zaphod:working-with-pods-answer$ oc get dc derived-app -o json | jq .spec | head
{
  "replicas": 3,
  "revisionHistoryLimit": 10,
  "selector": {
    "deploymentconfig": "derived-app"
  },
  "strategy": {
    "activeDeadlineSeconds": 21600,
    "resources": {},
    "rollingParams": {
zaphod:working-with-pods-answer$ 
```
