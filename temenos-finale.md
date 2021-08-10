```
apiVersion: app.kiegroup.org/v2
kind: KieApp
metadata:
  name: custom-rhpam
  namespace: mpaul-rhpam-custom
spec:
  imageRegistry:
    registry: quay.io
  auth:
    sso:
      adminPassword: aAEGB-_4GY3sCQ==
      adminUser: admin
      realm: demo
      url: >-
        https://keycloak-mpaul-rhpam-custom.apps.mw-ocp4.cloud.lab.eng.bos.redhat.com/auth
  commonConfig:
    adminPassword: password
    adminUser: adminuser
    applicationName: custom-rhpam
    dbPassword: password
  environment: rhpam-production-immutable
  objects:
    console:
      image: rhpam-businesscentral-monitoring-rhel8
      imageContext: ecosystem-appeng
      imageTag: 7.9.0
      replicas: 1
      ssoClient:
        name: business-central
        secret: f09c29fd-0d2a-45eb-a689-45fcb441b2c0
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
        - name: MAVEN_MIRROR_URL
          value: >-
            http://nexus3-mpaul-rhpam-custom.apps.mw-ocp4.cloud.lab.eng.bos.redhat.com/repository/rhpam-mirror
        - name: MAVEN_MIRROR_OF
          value: 'external:*,!rhpam'
    servers:
      - database:
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
        image: rhpam-kieserver-rhel8-custom-mssql
        imageContext: ecosystem-appeng
        imageTag: mriganka
        id: kieserver-1
        name: custom-kieserver
        deployments: 1
        replicas: 1
        ssoClient:
          name: kie-server
          secret: f773b81e-006e-45c8-849e-ae37cd8d1324
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
          - name: MAVEN_MIRROR_URL
            value: >-
              http://nexus3-mpaul-rhpam-custom.apps.mw-ocp4.cloud.lab.eng.bos.redhat.com/repository/rhpam-mirror
          - name: MAVEN_MIRROR_OF
            value: 'external:*,!rhpam'
```