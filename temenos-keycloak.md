```
apiVersion: keycloak.org/v1alpha1
kind: Keycloak
metadata:
  name: example-keycloak
  labels:
    app: sso
  namespace: mpaul-rhpam-custom
spec:
  externalAccess:
    enabled: true
  extensions:
    - >-
      https://github.com/aerogear/keycloak-metrics-spi/releases/download/1.0.4/keycloak-metrics-spi-1.0.4.jar
  instances: 1
```

admin
aAEGB-_4GY3sCQ==
kie-server - f773b81e-006e-45c8-849e-ae37cd8d1324
business-central - f09c29fd-0d2a-45eb-a689-45fcb441b2c0