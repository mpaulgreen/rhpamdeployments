- Creating extension image
```
virtualenv ~/cekit
source ~/cekit/bin/activate
cd cekit
unzip ../rhpam-7.9.0-openshift-templates.zip
curl -O https://raw.githubusercontent.com/cekit/cekit/3.2.0/requirements.txt
pip install -r requirements.txt
pip install -U "cekit==3.2"
pip install docker
pip install docker-squash
pip install behave
pip install lxml
cd templates/contrib/jdbc/cekit
pip install odcs
make mssql
```

- Storing the images in OpenShift registry
```
OCP_REGISTRY=$(oc get route -n openshift-image-registry | grep image-registry | awk '{print $2}')
docker login  -u `oc whoami` -p  `oc whoami -t` ${OCP_REGISTRY}

docker login quay.io
docker pull quay.io/ecosystem-appeng/rhpam-kieserver-rhel8-custom:7.9.0
docker tag quay.io/ecosystem-appeng/rhpam-kieserver-rhel8-custom:7.9.0 \
    ${OCP_REGISTRY}/`oc project -q`/rhpam-kieserver-rhel8-custom:7.9.0
docker tag kiegroup/jboss-kie-mssql-extension-openshift-image:7.2.2.jre11 \
    ${OCP_REGISTRY}/`oc project -q`/jboss-kie-mssql-extension-openshift-image:7.2.2.jre11
    
docker push ${OCP_REGISTRY}/`oc project -q`/rhpam-kieserver-rhel8-custom:7.9.0
docker push ${OCP_REGISTRY}/`oc project -q`/jboss-kie-mssql-extension-openshift-image:7.2.2.jre11
```
- Deployment Descriptor for custom image and extension image
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
  objects:
    console:
      replicas: 1
    servers:
      - build:
          extensionImageStreamTag: jboss-kie-mssql-extension-openshift-image:7.2.2.jre11
          extensionImageStreamTagNamespace: mpaul-rhpam-custom
        database:
          type: external
          externalConfig:
            driver: mssql
            dialect: org.hibernate.dialect.SQLServer2012Dialect
            jdbcURL: jdbc:sqlserver://172.30.205.227:31433;DatabaseName=rhpam
            nonXA: "true"
            username: sa
            password: msSql2019
            minPoolSize: '10'
            maxPoolSize: '10'
            connectionChecker: >-
              org.jboss.jca.adapters.jdbc.extensions.mssql.MSSQLValidConnectionChecker
            exceptionSorter: org.jboss.jca.adapters.jdbc.extensions.mssql.MSSQLExceptionSorter
            backgroundValidation: "true"
        image: rhpam-kieserver-rhel8-custom
        imageContext: mpaul-rhpam-custom
        imageTag: 7.9.0
        id: kieserver-1
        name: custom-kieserver
        deployments: 1
        replicas: 1
```