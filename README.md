# Notes_crt_do288

# Objectives DO288/EX288

Exam `DO288` objectives:

- Troubleshoot application deployment issues
- Create custom image streams
- Use OpenShift secrets and ConfigMaps to inject configuration data into applications
- Customize existing S2I builder images
- Implement hooks and triggers
- Work with multi-container templates
- Implement application health monitoring

# Deploying an app

## oc new-app

|Option|Description|
|------|-----------|
|--image-stream[-i]|image stream to be used|
|--strategy|docker or source|
|--code|URL to Git repo that will be used as input for S2I|
|--docker-image|URL to a container|

Examples:

```bash
# OS inspects the code to determine which image to use
oc new-app http://gitserver.example.com/mygitrepo

# use a PHP S2I builder image (both commands are the same)
oc new-app php~http://gitserver.example.com/mygitrepo
oc new-app -i php http://gitserver.example.com/mygitrepo
# again, same with tag
oc new-app php:7.0~http://gitserver.example.com/mygitrepo
oc new-app -i php:7.0 http://gitserver.example.com/mygitrepo

# command detects the container image URL as the input argument
oc new-app registry.example.com/mycontainerimage
# to avoid detection problems, you can force the type of URL (code vs Docker image)
oc new-app --code http://gitserver.example.com/mygitrepo
oc new-app --docker-image registry.example.com/mycontainerimage
# to force use of Dockerfile instead of S2I
oc new-app --strategy docker http://gitserver.example.com/mydockerfileproject
# to force use of S2I instead of Dockerfile
oc new-app --strategy source http://gitserver.example.com/user/mygitrepo
# other options, such as --image-stream and --code, can be used in the same command with --strategy
```

`oc new-app` command adds the following resources to the current project:

- build configuration
- image stream
- deployment configuration
- service for all ports exposed by the application container image
- `app` label based on the short name of the Git repo or `--name`

```bash
# following command deletes all resources created by the previous oc new-app command
oc delete all -l app=test
```

```bash
# if you want to inspect the resource definitions without creating the resources in the current project, you use the -o option
oc new-app -o json registry.example.com/mycontainerimage
```

## Image Streams and Tags

- Having container image metadata in an IS allows to perform operations based on this data instead of going to a registry server every time.
- Allows using either notification or pooling strategies to react to image content updates.
- IS resource can define multiple IS tags, tags can point to a different container image tag or even to a different container image name (e.g. the ruby IS from the openshift project defines the following IS tags: `ruby:2.0`, `ruby:2.2` both referring to different images in different registries).

```bash
# easiest way to create an image stream is by using the oc import-image command with the --confirm option
oc import-image myis --confirm --from registry.acme.example.com:5000/acme/awesome --insecure
```

```bash
# create new app using Dockerfile in Git repo using base image in an insecure registry
oc new-app --name echo --insecure-registry http://services.lab.example.com/rhel7-echo

# build configuration output is a container image sent to the internal registry through an image stream
oc describe bc echo | grep "Output to:"
# Output to: ImageStreamTag echo:latest

# deployment configuration has a trigger in the image stream. If the image stream changes, a new deployment is performed
oc describe dc echo | grep "Triggers:"

# change the app, rebuild the application and verify that OpenShift deploys the new container image
oc start-build echo
# show the history of image streams
oc describe is echo

```

## Managing Applications with the Web Console

- The logs from a deployment are **not** stored by OpenShift unless there were errors during deployment.
- Default OpenShift router implements session affinity (if you have multiple pods and you go to the router's URL and keep refreshing, you will be routed to the same pod every time).
- If the application source code repository is **not** configured to use OpenShift web hooks, you need to use either the web console or the CLI to trigger new builds, after pushing updates to the application source code.
- The details page for a deployment also provides the Deploy button, which performs the same function as the `oc rollout latest` command. You need to use the CLI to perform other `oc rollout` operations.
- Deleting a project deletes **all** resources inside the project.

## Managing Applications with the CLI

- The default configuration for an OpenShift cluster allows any user to create new projects.
- A user is automatically a project administrator for any project he creates.

- Access levels:

|Type|Permissions|
-----|------------
|Cluster admin|can manage projects, add nodes, create persistent volumes, assign project quotas, and perform other cluster-wide administration tasks.|
|Project admin|can manage resources inside a project, assign resource limits, and grant other users permission to view and manage resources inside the project.|
|Developer|can manage a subset of a project's resources. The subset includes resources required to build and deploy applications, such as build and deployment configurations, persistent volume claims, services, secrets, and routes. They cannot grant to other users any permission over these resources, and cannot manage most project-level resources such as resource limits.|

- Search for a template that uses PHP and MySQL: `oc get templates -n openshift | grep php | grep mysql`.
- When you create a new template (`oc create -f <name.json>`) not in openshift namespace (`-n openshift`), it will only be available in the project where you created it.


## Troubleshooting Builds, Deployments, and Pods Using the CLI

- Differences between `oc describe` and `oc get -o`:
  - The `oc describe` command can follow relationships between resources (e.g. describing a build configuration shows information about the most recent build).
  - The `oc get -o` command shows information only about the requested resource (e.g. build configuration does not show information about the recent builds).
- `oc edit` = `oc get -o` + `vim` + `oc apply`.
- The logs from a build configuration (`oc logs bc/myapp`) are the logs from the latest build, whether it was successful or not, but you can use `oc logs build/myapp-X` to see the logs from the Xth build.
- `oc logs myapp-build-X` to see the logs of the build pod created to perform the Xth build.
- to retrieve the Apache HTTP Server error logs stored inside an application pod called frontend, use: `oc cp frontend-1-abcde:/var/log/httpd/error_log /tmp/frontend-server.log`, unlike the UNIX cp command, the `oc cp` command cannot copy a source file to a destination folder, only folder -> folder and file -> file.
- `oc cp` command requires the application container image to provide the tar command. If not, the command will fail.
- `oc rsync` command synchronizes local folders with remote folders from a running container. It uses the local rsync command to reduce bandwidth usage, but does not require the rsync or ssh commands to be available in the container image.
- `oc rsh command` creates a remote shell to execute commands inside the container. It uses the OpenShift master API to create a secure tunnel to the remote pod, but does not use the ssh or the rsh UNIX commands.
- Start an interactive shell session (`-t`) inside the container: `oc rsh -t frontend-1-abcde`.
- Copy the SQL script to the database pod example: `oc cp ./DO288/labs/build-template/quote.sql quotesdb-1-hh2g9:/tmp/quote.sql`.
- Run the SQL script inside the database pod example: `oc rsh -t quotesdb-1-hh2g9` and then when inside: `mysql -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE < /tmp/quote.sql`.

Troubleshooting three-tier app (`HTTP/1.1 500 Internal Server Error` when `curl` the router URL):

1. Verify that the database service found the correct database pod: `oc describe svc db | grep Endpoints` and `oc describe pod db-1-abcde | grep IP`. Does it show the same IP?
2. Verify the database pod login credentials: `oc describe pod db-1-abcde | grep -A 4 Environment`. e.g. MYSQL_USER, MYSQL_PASSWORD etc.
3. Verify the database connection parameters in the application pod: `oc describe pod api-1-abcde | grep -A 5 Environment`. e.g. DATABASE_SERVICE_NAME, DATABASE_NAME etc.
4. Verify that the application pod can reach the database pod: `oc rsh api-1-abcde bash -c 'curl $DATABASE_SERVICE_NAME:3306'`.
5. Review application logs: `oc logs api-1-abcde`.

## Build and Deployment Environment Variables

- Add the `-e` option to the `oc new-app` command to provide values for environment variables. These values are stored in the deployment configuration and added to all pods created by a deployment.
- For a builder pod, you define environment variables using the `--build-env` option of the `oc new-app` command.
- Deployment configuration (`dc`) stores environment variables for application pods, while a build configuration (`bc`) stores environment variables for builder pods.


# Designing containerized apps for Openshift

- Preferred way to build images is S2I.
- OpenShift provides the s2i command-line tool that helps you bootstrap the build environment for creating custom S2I builder images. It is available in the source-to-image package from the Red Hat Software Collections RHSCL Yum repositories (rhel-server-rhscl-7-eus-rpms).
- Red Hat recommends to run the image as a non-root user for security reasons.
- OpenShift, by default, does **not** honor the `USER` instruction set by the container image. For security reasons, a random userid other than the root userid (0) is used by OpenShift to run containers.

## Building Container Images with Advanced Dockerfile Instructions

- Each `RUN` in `Dockerfile` creates a new layer. It is recommended to use `&&` on each line to create one long command and thus one layer.
- Openshift uses lots of `LABEL` instructions (`io.openshift.tags`, `io.k8s.description`, `io.openshift.expose-services`).
- When building images for Openshift, the namespace should be set to `io.openshift`.
- `WORKDIR` should be absolute path.
- It is a good practice to use `ENV` instruction(s) to define file and folder path(s) instead of hard-coding it. It is recommended to use one `ENV` and `\` on each line to only create one layer.
- `USER` sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD`, and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- `VOLUME` - mountpoint inside the container, used for persistent data, content is preserved even if the container is restarted or moved to a different node.
- `ONBUILD` instruction registers triggers in the container image, declare instructions to be executed only when building a child image. Useful when you have image that everyone in the organization uses and build child images from it. The **parent** Dockerfile has `ONBUILD` instructions in this case and when the build process of the **child** image starts, it triggers the execution of the `ONBUILD` instructions in the **parent** image.

### Adapting `Dockerfile`s for Openshift

- Directories and files that are read from or written to by processes in the container should be owned by the **root** group and have group read or group write permission (`RUN chgrp -R 0 <directory> && chmod -R g=u <directory>`).
- Files that are executable should have group execute permissions.
- The processes running in the container must **not** listen on privileged ports (that is, ports below 1024), because they are not running as privileged users.
- Example of the above:

```Dockerfile
# Use the do288/httpd-parent image as base
FROM registry.lab.example.com:5000/do288/httpd-parent

# Change the port to 8080
EXPOSE 8080

# Override the LABEL from parent
LABEL io.openshift.expose-services="8080:http"

# Change web server port to 8080
RUN sed -i "s/listen 80/listen 8080/g" /etc/nginx.conf

# Permission to allow container to run on Openshift
RUN chgrp -R 0 /var/opt/rh/rh-nginx18 && chmod -R g=u /var/opt/rh/rh-nginx18

# Run as a non-privileged user, Red Hat recommends this
USER 1001
```

### Running Containers as root Using Security Context Constraints (SCC)

- All containers created by OpenShift use the SCC named **restricted** by default, which ignores the userid set by the container image, and assigns a random userid to containers. You need `anyuid` SCC to allow containers run as root.
- All pods from a project run under a default service account, unless the pod, or its deployment configuration, is configured otherwise.
- Allow containers to run as the root user in an OpenShift project (**not** recommended):

```bash
oc create serviceaccount <myserviceaccount>
oc patch dc/demo-app --patch '{"spec":{"template":{"spec":{"serviceAccountName": "<myserviceaccount>"}}}}'
   OR oc edic dc/demo-app and add serviceAccountName attribute to the spec>template>spec section (below terminationGracePeriodSeconds)
oc adm policy add-scc-to-user anyuid -z <myserviceaccount>
```

## Injecting Configuration Data into an Application

- The recommended approach for containerized applications is to decouple the static application binaries from the dynamic configuration data and to externalize the configuration. This ensures the portability of applications across many environments.
- Secrets and configuration maps must be created **before** creating the pods that depend on them.

```bash
oc create configmap config_map_name \
--from-literal key1=value1 \
--from-literal key2=value2
```

---
*Last updated: Mon Feb 04 17:00:00 AEDT 2019 (notes from DO288-OCP3.6-en-1-20180130-ROLE.pdf, Progress 23.9% = page 95 of 394)*

# EX288 - Red Hat Certified Specialist in OpenShift Application Development

## OpenShift CLI

### Explain

`oc` can explain each object type definition using `oc explain <object>`. Drill
down into the object using JSON dot notation: `spec.containers`.

## Pull from private registry

Create pull secret in OpenShift:

```bash
oc create secret generic <name> \
    --from-file '.dockerconfigjson=<path-to-auth.json>' \
    --type kubernetes.io/dockerconfigjson
```

Link service account to pull secret:

```bash
oc secrets link default <name> --for pull
```

## `ONBUILD` Docker Directive

Parent image:

```dockerfile
FROM registry.access.redhat.com/rhscl/nodejs-6-rhel7
RUN mkdir -p /opt/app-root/
WORKDIR /opt/app-root
ONBUILD COPY package.json /opt/app-root
CMD [ "npm", "start" ]
```

Child image:

```dockerfile
FROM mynodejs-base
RUN echo "Started Node.js server..."
```

Child image will copy `./package.json` from the child source code to
`/opt/app-root` of the container on build.

## Expose Services LABEL

```dockerfile
LABEL io.openshift.expose-services="8080:http"
```

## Security Context Controls (SCC)

Add anyuid to service account:

```bash
oc adm policy add-scc-to-user anyuid -z myserviceaccount
```

## Creating ConfigMaps and Secrets

### Create a ConfigMap from name=value pairs

```bash
oc create configmap <name> \
    --from-literal key1=value1 \
    --from-literal key2=value2
```

### Create a ConfigMap from a file

```bash
oc create configmap <name> --from-file <path-to-file>
```

### Create a Secret from name=value pairs

```bash
oc create secret generic <name> \
    --from-literal key1=value1 \
    --from-literal key2=value2
```

### Create a Secret from YAML

```bash
apiVersion: v1
kind: Secret
metadata:
    name: <name>
    type: Opaque
stringData:
    key1: value1
    key2: value2
```

**NOTE:** `stringData` is converted to base64 encoded `data` when applied.

## Using Secrets and ConfigMaps in Deployments

### Set as environment variable

```bash
oc set env deployment/<name> --from configmap/<name> # ConfigMap
oc set env deployment/<name> --from secret/<name>    # Secret
```

### Set ConfigMap as volume

```bash
oc set volume deployment/<name> --add \
    -t configmap \
    -m <path-on-container> \
    --name <name> \
    --configmap-name <name>
```

### Set Secret as volume

```bash
oc set volume deployment/<name> --add \
    -t secret \
    -m <path-on-container> \
    --name <name> \
    --secret-name <name>
```

## Deploy Applications

### Deploy Application from a Container Image

```bash
oc new-app \
    --name <name> \
    --docker-image quay.io/image:latest
```

### Deploy Application from Git Repo

```bash
oc new-app \
    --name <name> \
    --as-deployment-config \ # Use DeploymentConfig instead of Deployment
    --build-env KEY=VALUE \  # Environment variable for build time only
    --env KEY=VALUE \        # Environment variable for build and run time
    --context-dir <dir-path> # Directory inside the Git repo with app code
    php:7.3~https://github.com/repo#branch
```

**FORMAT: `image:tag~repo#branch`**

### Scale a Deployment

```bash
oc scale dc/<name> --replicas=3
```

**NOTE:** Might be easier to `oc edit dc/<name>` and edit the # of replicas.

### Edit Deployment Method

DeploymentConfig supports custom deployment methods. Look for `.spec.strategy`
in the DeploymentConfig object. After update, force rollout with: `oc rollout
latest dc/<name>`.

### Rollout/Rollback Deployment

Rollout latest deployment version:

```bash
oc rollout latest dc/<name>
```

Rollback to last successful deployment:

```bash
oc rollback dc/<name>
```

Reset deployment triggers after rollback:

```bash
oc set triggers dc/<name> --auto
```

## Internal Registry

### Expose Internal Registry Outside of Cluster

```yaml
$ oc edit config.imageregistry cluster -n openshift-image-registry
# Set spec.defaultRoute to true
```

**NOTE:** There is a patch command to do this but patch is annoying outside of
scripting.

### Grant User Access to Internal Registry

For minimum capabilities to push and pull **(More than likely you want this)**:

* `oc policy add-role-to-user system:image-puller <username> -n <project>`
* `oc policy add-role-to-user system:image-pusher <username> -n <project>`

For enterprise use cases (grants more than above):

* `oc policy add-role-to-user registry-viewer <username> -n <project>`
* `oc policy add-role-to-user registry-editor <username> -n <project>`

Can also add these roles to groups, specifically for services accounts in a
project:

```bash
oc policy add-role-to-group \
    system:image-puller \
    system:serviceaccounts:<project> \
    -n <project>
```

## ImageStreams

### Import External Image

```bash
oc import-image imagestream-name:latest \
    --from quay.io/image:latest \
    --confirm
```

**NOTE:** Ommitting the tag on both the image stream and external image will
import all tags for that image on the external registry.

### Referencing ImageStreams

Image streams are namespace scoped. **Sometimes image streams need a project
prefix.** Full format looks like `project/imagestream:tag`.

## Source-to-image (S2I)

### Pruning Completed Builds

In the BuildConfig manifest, add:

```yaml
...
spec:
  successfulBuildsHistoryLimit: 5
  failedBuildsHistoryLimit: 5
  ...
```

### Set Build Log Level

```bash
oc set env bc/<name> BUILD_LOGLEVEL="4"
```

### Triggering Builds

#### On Image Change Trigger

```bash
oc set triggers bc/<name> --from-image=<imagestream>:tag
```

**NOTE:** S2I created containers will be triggered on S2I builder image change.
Example: `php:latest` image stream is updated, S2I applications built using
`php:latest` will be triggered.

#### On GitLab Webhook Trigger (for GitHub/Bitbucket, replace the from option)

```bash
oc set triggers bc/<name> --from-gitlab
```

### Post-Commit Build Hooks

```bash
oc set build-hook bc/<name> \
    --post-commit \
    --script="echo 'Hello World!'"
```

**NOTE:** This requires the base image to have `/bin/sh`. Probably safe to
assume most images have this.

### Set Build Environment Variables

```bash
oc set env bc/<name> KEY="VALUE"
```

### Overriding S2I Build Scripts

Scripts in the `./s2i/bin` directory of the source Git repo will override
builder image scripts. This is typically done to wrap the builder image scripts
to perform some action at a given point in the build.

## S2I Builder Images

### S2I Builder Image Labels

Set S2I scripts directory using Containerfile LABEL directive:

```Dockerfile
LABEL io.openshift.s2i.scripts-url="image:///usr/libexec/s2i"
```

### Create Builder Image with CLI

```bash
s2i create <image-name> <directory>
```

### Test Build using CLI

```bash
s2i build <repo> <image-name> <tag-name>
```

## OpenShift Templates

### Basic Template

```yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: <template-name>
labels:
  mylabel: value
parameters:
  - description: <description>
    name: KEY
    defaultValue: VALUE
    required: true
objects:
  - ... # K8s object 1
  - ... # K8s object 2
```

### Apply Template from File

Preferred:

```bash
oc new-app --file <template-file> \
    -p KEY1=VALUE1 \
    -p KEY2=VALUE2
```

Optional:

```bash
oc process -f <template-file> \
    -p KEY1=VALUE1 \
    -p KEY2=VALUE2 | oc create -f -
```

## Helm

`helm create <name>` gets you 80% to your goal.

### Add Dependencies

Chart.yaml:

```yaml
...
dependencies:
  - name: <chart-name>
    version: <chart-version>
    repository: <repo-url>
  ...
```

Then install/update dependencies:

```bash
helm dependency update
```

### Loop Through List in Template

*./template/deployment.yaml*:

```yaml
...
env:
  {{- range .Values.env }}
  - name: {{ .name }}
    value: {{ .value }}
  {{- end }}
...
```

*Values.yaml*:

```yaml
...
env:
  - name: KEY1
    value: VALUE1
...
```

## Kustomize

Directory layout:

```text
./base
./base/kustomize.yaml
./base/<manifest>.yaml
./overlays
./overlays/dev
./overlays/dev/kustomize.yaml
./overlays/dev/<manifest>.yaml
./overlays/prod
./overlays/prod/kustomize.yaml
./overlays/prod/<manifest-patches>.yaml
```

*./base/kustomize.yaml*:

```yaml
resources:
  - deployment.yaml
  - service.yaml
commonLabels:
  key1: value1
```

*./overlay/{dev,prod}/kustomize.yaml*:

```yaml
bases:
  - ../../base
patches:
  - ../../<manifest-patch>.yaml
```

### Applying Kustomizations

```bash
oc apply -k <kustom-directory>
```

## Monitoring Probes

Liveness Probe:

```bash
oc set probe deployment <name> \
    --liveness \
    --get-url=http://:<port>
```

Readiness Probe:

```bash
oc set probe deployment <name> \
    --readiness \
    --get-url=http://:<port>
```

Use `oc set probe --help` to get other options.

## Services

### Service DNS

A service is available via internal DNS at
`<service-name>.<project>.svc.cluster.local`. (You might be able to omit
`.cluster.local`.)

**NOTE: This information isn't provided in an `oc describe service <name>`
response. Remember the format so you can point applications at each other!!**

### External Services

To create a service that points to an external dependency:

```bash
oc create service externalname <service-name> --external-name <external-fqdn>
```

This can also be used to create aliases for internal services:

```
oc create svc externalname <alias-service-name> \
    --external-name=<orignial-service-name>.<project>.svc.cluster.local
```
