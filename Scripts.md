# Git management

```sh
cd

git checkout -b source-build
git push -u origin source-build
```

# New APP 


oc new-app --name greet --build-env npm_config_registry=http://.../repository/nodejs nodejs:16-ubi8~https://github.com/.../DO288-apps#source-build --context-dir nodejs-helloworld

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
