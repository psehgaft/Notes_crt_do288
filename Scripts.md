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


# 

--as-deployment-config	 Configura el comando oc new-app para crear un recurso DeploymentConfig en lugar de Deployment.
--image-stream / -i      Proporciona el flujo de imágenes que se usará como la imagen de compilador S2I para una compilación S2I o para implementar una imagen de contenedor.
--strategy	             docker o pipeline o source
--code	                 Proporciona la URL a un repositorio de git que se usará como entrada para una compilación S2I.
--docker-image	         Proporciona la URL a una imagen de contenedor que se implementará.
--dry-run	               Se establece en true para mostrar el resultado de la operación sin realizarla.
--context-dir	           Proporciona la ruta a un directorio para tratar como directorio raíz.
