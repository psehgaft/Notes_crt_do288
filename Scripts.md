# Git management

```sh
cd

git checkout -b source-build
git push -u origin source-build
```

# New APP 

Create a new application from sources in the Git repository. Name the application greet. Use the --build-env option with the oc new-app command to define the build environment variable with the npm modules URL.

```sh
oc new-app --name greet --build-env npm_config_registry=http://psehgaft/repository/nodejs nodejs:16-ubi8~https://github.com/psehgaft/DO288-apps#source-build --context-dir nodejs-helloworld

# imagestream
oc new-app php:7.0~http://gitserver.example.com/mygitrepo

oc new-app -i myis --strategy source --code http://gitserver.example.com/mygitrepo

oc new-app --strategy docker http://gitserver.example.com/mydockerfileproject

oc new-app --strategy source http://gitserver.example.com/user/mygitrepo
```

Create app, with template

```sh
oc new-app mysql -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=testdb -l db=mysql
```

Create app  as a Deployment Config

```sh
oc new-app --as-deployment-config --name hello -i php --code http://gitserver.example.com/mygitrepo
```

Inspection, no create resources -o

```sh
oc new-app -o json registry.example.com/mycontainerimage
```

# Import

```sh
oc import-image myis --confirm --from registry.acme.example.com:5000/acme/awesome --insecure
```

# Dockerfile

```sh
FROM registry.access.redhat.com/ubi8/ubi:8.0 
USER 1001
RUN   chown -R wildfly:wildfly /opt/app-root && chmod -R 700 /opt/app-root
RUN   chgrp -R 0 /opt/app-root && chmod -R g=u /opt/app-root
EXPOSE 3000
RUN mkdir -p /opt/app-root/
WORKDIR /opt/app-root
CMD bash -c "while true; do echo test; sleep 5; done"
ENV "OPENSHIFT_BUILD_NAME"="echo-1"
LABEL
COMMIT
```

ssh file:

```sh
oc new-app --template ${RHT_OCP4_DEV_USER}-common/php-mysql-ephemeral \
  -p NAME=quotesapi \
  -p APPLICATION_DOMAIN=quote-${RHT_OCP4_DEV_USER}.${RHT_OCP4_WILDCARD_DOMAIN} \
  -p SOURCE_REPOSITORY_URL=https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps \
  -p CONTEXT_DIR=quotes \
  -p DATABASE_SERVICE_NAME=quotesdb \
  -p DATABASE_USER=user1 \
  -p DATABASE_PASSWORD=mypa55 \
  --name quotes
```

# Copy files

```sh
oc cp frontend-1-zvjhb:/var/log/httpd/error_log /tmp/frontend-server.log
```

# Exe commands

```sh
oc rsh frontend-1-zvjhb ps -eaf
```

# Podman

```sh
podman build --layers=false -t do288-apache ./container-build
```

# Opciones soportadas

```sh
--as-deployment-config	 Configura el comando oc new-app para crear un recurso DeploymentConfig en lugar de Deployment.
--image-stream / -i      Proporciona el flujo de imágenes que se usará como la imagen de compilador S2I para una compilación S2I o para implementar una imagen de contenedor.
--strategy	             docker o pipeline o source
--code	                 Proporciona la URL a un repositorio de git que se usará como entrada para una compilación S2I.
--docker-image	         Proporciona la URL a una imagen de contenedor que se implementará.
--dry-run	               Se establece en true para mostrar el resultado de la operación sin realizarla.
--context-dir	           Proporciona la ruta a un directorio para tratar como directorio raíz.
```
