- Authenticate Red Hat Registry
```
oc create -f rh.registry-secret.yaml
RH_SECRET_NAME=$(grep name rh.registry-secret.yaml | sed -e 's/.*: //')
oc secrets link default ${RH_SECRET_NAME} --for=pull
oc secrets link builder ${RH_SECRET_NAME} --for=pull
```

- Authenticate quay.io
```
oc create -f quay.io-secret.yaml
QUAYIO_SECRET_NAME=$(grep name quay.io-secret.yaml | sed -e 's/.*: //')
oc secrets link default ${QUAYIO_SECRET_NAME} --for=pull
oc secrets link builder ${QUAYIO_SECRET_NAME} --for=pull
```

- Pre-requisites
  - Install the business automation operator
- Create the custom-rhpam.yaml file
```
apiVersion: app.kiegroup.org/v2
kind: KieApp
metadata:
  name: custom-rhpam
  namespace: mpaul-rhpam-custom
spec:
  commonConfig:
    adminPassword: password
    adminUser: admin
    applicationName: custom-rhpam
    dbPassword: password
  environment: rhpam-production
  imageRegistry:
    registry: quay.io
  objects:
    console:
      replicas: 1
    servers:
      - database:
          type: mysql
        deployments: 1
        id: kieserver-1
        image: rhpam-kieserver-rhel8-custom
        imageContext: ecosystem-appeng
        imageTag: 7.9.0
        name: custom-kieserver
        replicas: 1
```

```
oc create -f custom-rhpam.yaml
```
