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

# DO288 Containerized Example Applications

This repository contains a collection of sample containerized applications.  To complete the course you need to fork this repo into your personal Github account.

## No. 1

```sh
oc new-app --name [name] --build-env npm_config_registry=[nexus-url] nodejs:16-ubi8~[git-url]#[git-branch] --context-dir[git-context-dir]
oc logs -f bc/[build-config-name]
# Fix package.json (:)
oc start-buld -follow bc/[build-config-name]
```
## No2. 

```docker
FROM registry.access.redhat.com/ubi8/ubi:8.0 
MAINTAINER Red Hat Training <training@redhat.com>
# DocumentRoot for Apache
ENV DOCROOT=/var/www/html

LABEL io.k8s.description="A basic Apache HTTP Server child image, uses ONBUILD" \
      io.k8s.display-name="Apache HTTP Server" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="apache, httpd"

RUN sed -i "s/Listen 80/Listen 8080/g" /etc/httpd/conf/httpd.conf \
RUN sed -i "s/#ServerName www.example.com:80/ServerName 0.0.0.0:8080/g" \
    /etc/httpd/conf/httpd.conf

RUN   yum install -y --no-docs --disableplugin=subscription-manager httpd && \ 
      yum clean all --disableplugin=subscription-manager -y && \
      echo "Hello from the httpd-parent container!" > ${DOCROOT}/index.html && \
      rm -rf /run/httpd && \
      mkdir /run/httpd

# Allows child images to inject their own content into DocumentRoot
ONBUILD COPY src/ ${DOCROOT}/ 

EXPOSE 8080

# This stuff is needed to ensure a clean start
RUN rm -rf /run/httpd && mkdir /run/httpd

RUN   chgrp -R 0 ${DOCROOT} /var/log/httpd /var/run/httpd && \
      chmod -R g=u ${DOCROOT}/ /var/log/httpd /var/run/http

# Run as the root user
USER 1001

# Launch httpd
CMD /usr/sbin/httpd -DFOREGROUND
```

Service Account

```sh
podman build --layers=false -t [image-name] ./
podman tag [image-name] [registry]/[image-name]
podman push --format v2s1 [registry]/[image-name]

oc create serviceaccount myserviceaccount
oc patch dc/demo-app --patch '{"spec":{"template":{"spec":{"serviceAccountName": "myserviceaccount"}}}}'
oc adm policy add-scc-to-user anyuid -z myserviceaccount
```

## No3. 

```sh
podman run --name test -it rhscl/httpd-24-rhel7 bash

vi [dir].s2i/bin/assemble

CUSTOMIZATION STARTS HERE 

echo "---> Installing application source"
cp -Rf /tmp/src/*.html ./

DATE=date "+%b %d, %Y @ %H:%M %p"

echo "---> Creating info page"
echo "Page built on $DATE" >> ./info.html
echo "Proudly served by Apache HTTP Server version $HTTPD_VERSION" >> ./info.html

 CUSTOMIZATION ENDS HERE 

oc new-app --name [name] httpd:2.4~[git-url]#[git-branch] --context-dir[git-context-dir]
```

## No4. 

```sh
oc describe bc/[build-config-name]
oc set build-hook bc/[build-config-name] --post-commit --command -- python /opt/app-root/src/abc.py 
oc start-build bc/[build-config-name]
```

## No5. 

```sh
oc patch configmap/myconf --patch '{"data":{"key1":""}}'
oc patch secret/mysecret --patch '{"data":{"password":""}}'

oc set env deployment/mydcname --from configmap/myconf
oc set volume deployment/mydcname --add -t configmap -m /path/to/mount/volume --name myvol --configmap-name myconf

oc set env deployment/mydcname --from secret/mysecret
oc set volume deployment/mydcname --add -t secret -m /path/to/mount/volume --name myvol --secret-name mysecret
```

## No6. 

```sh

oc set probe deployment probes --liveness --get-url=http://:8080/healthz --initial-delay-seconds=2 --timeout-seconds=2
oc set probe deployment probes --readiness --get-url=http://:8080/ready --initial-delay-seconds=2 --timeout-seconds=2

```

## No7. 

```sh
helm create app1
```

vi values.yaml

```sh
mariadb:
  auth:
    username: quotes
    password: quotespwd
    database: quotesdb
  primary:
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
```

vi templates/deployment.yml

```sh
imagePullPolicy: {{ .Values.image.pullPolicy }}
env:
  {{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
  {{- end }}
```

vi chart.yaml

```sh
dependencies:
- name: mariadb
  version: 11.0.13
  repository: https://charts.bitnami.com/bitnami
```

```sh
helm dependency update
helm install app1 .
```
```

## No9. 

```sh
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
HOST=$(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')
podman login -u kubeadmin -p $(oc whoami -t) --tls-verify=false $HOST
oc policy add-role-to-user system:image-puller user_name -n project_name
```
