# building-from-base-container

Steps for executing this exercise:
1.  Download base image from [Box](https://ibm.webex.com/ibm/url.php?frompanel=false&gourl=https%3A%2F%2Fibm.box.com%2Fs%2Fl7wanilt7k9fcjcw0s04pik8ljyi7oig) and un-gzip it.
```
gunzip a-base-image.gz
```
2.  Use podman / docker load to load this OCI image into your local image cache.
```
podman load -i=a-base-image
```
3.  Validate that you can run this by using the run command. The server will run on port 9080. <code>curl http://localhost:<port number\>/simple-stuff/simple/simon</code> will produce <code>/my-special-folder does not exist</code>
```
podman run -d --name=base -p 9080:9080 localhost/a-base-image:1.0
curl http://localhost:9080/simple-stuff/simple/simon
```
4.  Create a new image from the base image:
    1. The new image should have a directory called /my-special-folder. Note that you will need to create this folder as the root user, otherwise, your creation of the directory will fail during the build.
```
see Dockerfile in this git repo
```
    2. Perform the build. Call the resulting image "a-derived-image:1.0"
```
podman build -t a-derived-image:1.0 .
```
    3. Run the resulting image. Ensure that there are no port conflicts on your local machine by mapping the server port (9080) to a different port on your machine.
```
podman run -d --name=derived -p 9081:9080 localhost/a-derived-image:1.0
```
    4. Re-run the <code>curl</code> command (against the right port number). This time, the output should be your Dockerfile.
```
curl http://localhost:9081/simple-stuff/simple/simon
```
5.  Connect to your openshift cluster on the command line.
    1.  Login via the console via the provided console url
    2.  Use the "Copy Login Command" function from the top right of the console to get the command to log in via the command line
6.  Use the docker / podman command, along with the output of the <code>oc whoami -t</code> command (as the password) to log into the cluster
    1.  The image registry URL for your cluster will be https://default-route-openshift-image-registry.<cluster domain\>
*Example values used in the commands, below:*
```
TOKEN=`oc whoami -t`
podman login -u IAM#kstephe@us.ibm.com -p ${TOKEN} https://default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud
```
7.  You will already have an assigned project / namespace within the cluster. When logging in at step 5.2, the output would have shown you that you are in that project. Do a docker / podman push to the image registry with the tag of <code>default-route-openshift-image-registry.<cluster domain\>/<your namespace\>/derived:1.0</code>
```
podman push localhost/a-derived-image:1.0 default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:1.0
```
Note that with the docker cli, this single step push won't work. Instead, you will have to tag the image first, and then push:
```
docker tag localhost/a-derived-image:1.0 default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:1.0
docker push default-route-openshift-image-registry.ose-internal-1602064171-f72ef11f3ab089a8c677044eb28292cd-0000.sjc03.containers.appdomain.cloud/kstephe-us/a-derived-image:1.0
```
Note that in the case of the "docker tag" command, above, there are reports that this fails with some versions of docker. Please remove the "localhost/" in the local image tag and retry if you get a failure.
    
8.  Validate that your push was successful by doing <code>oc get is</code> within your project and observing that your newly built image is there.
