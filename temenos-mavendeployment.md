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
      env:
        - name: MAVEN_REPO_URL
          value: >-
            http://nexus3-mpaul-rhpam-custom.apps.mw-ocp4.cloud.lab.eng.bos.redhat.com/repository/rhpam
        - name: MAVEN_REPO_ID
          value: rhpam
        - name: MAVEN_REPO_USERNAME
          value: deployer
        - name: MAVEN_REPO_PASSWORD
          value: deployer123      
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
        env:
          - name: MAVEN_REPO_URL
            value: >-
              http://nexus3-mpaul-rhpam-custom.apps.mw-ocp4.cloud.lab.eng.bos.redhat.com/repository/rhpam
          - name: MAVEN_REPO_ID
            value: rhpam
          - name: MAVEN_REPO_USERNAME
            value: deployer
          - name: MAVEN_REPO_PASSWORD
            value: deployer123      
```