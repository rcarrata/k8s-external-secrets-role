External Secrets
================

Kubernetes External Secrets allows you to use external secret management systems, like AWS Secrets Manager.

This role permit to install external secrets on Openshift


Requirements
===========

The role do a final test to get a test secret. It is necessary to create a secret on aws secret manager

`aws secretsmanager create-secret --name hello-service/secrets --secret-string "1234"`


### Create a secret

To create secrets first needs to install in aws and then create a external secret object in Openshift , you can see how if it sync `oc get externalsecret hello-service`

`aws secretsmanager create-secret --name $SECRET_NAME/secrets --secret-string "1234"`

`oc apply -f external-secret.yaml`

external-secret.yaml

```
apiVersion: 'kubernetes-client.io/v1'
kind: ExternalSecret
metadata:
  name: hello-service
  namespace: myapp
spec:
  backendType: secretsManager
  data:
    - key: hello-service/secret
      name: secret
```


### Creation of an external secret and consumption

template-secret.yaml

```
apiVersion: v1
kind: Template
metadata:
  name: app-template
  annotations:
    description: "Description"
    iconClass: "icon-app"
    tags: "app,secretsmanager"
objects:
- apiVersion: 'kubernetes-client.io/v1'
  kind: ExternalSecret
  metadata:
    name: ${SECRET_NAME}
  spec:
    backendType: secretsManager
    data:
      - key: ${SECRET_NAME}/secrets
        name: secrets
parameters:
- description: Description
  name: SECRET_NAME
  value: rcs-api.secret
```

if you want to create more than password on a secret :

    data:
      - key: ${SECRET_NAME}/pass1
        name: pass1
      - key: ${SECRET_NAME}/pass2
        name: pass2

## Get openshift secret by password
```
oc get secret $SECRET_NAME -o json | jq -r .data.pass1 | base64 --decode
oc get secret $SECRET_NAME -o json | jq -r .data.pass2 | base64 --decode
```


```
secrets: {project}-{environment}/{repository-name}-secret

SECRET_FILE=rcs-dev/web
SECRET_NAME=rcs-dev-rcs-web
NAMESPACE=rcs-dev

aws secretsmanager create-secret --name $SECRET_NAME/secrets --secret-binary fileb://$SECRET_FILE.secrets
aws secretsmanager put-secret-value --secret-id $SECRET_NAME/secrets --secret-binary fileb://$SECRET_FILE.secrets
oc process -p SECRET_NAME=$SECRET_NAME  -f ./template-secret.yaml -n $NAMESPACE | oc apply -f -

oc get externalsecrets -n $NAMESPACE
oc get secret $SECRET_NAME -o json | jq -r '.data.secrets' | base64 --decode


aws secretsmanager put-secret-value --secret-id $SECRET --secret-binary fileb://$SECRET.secrets
aws secretsmanager put-secret-value --secret-id $SECRET --secret-binary fileb://$SECRET  --version-stage $VERSION

```

Role Variables
--------------
```
esecrets: true
esecret_namespace: infra-external-secrets
esecret_nodeselector: infra
esecret_chart_version: 5.0.0
esecret_replicas: 3
```


Dependencies
------------

community.kubernetes:1.0.0


Example Playbook
----------------

```
- hosts: localhost
  connection: local
  vars_files:
    - vars/vars.yml
  tasks:
    - role:
      when:
        - esecrets is defined
        - esecrets != False
```

More Info
=========
https://github.com/godaddy/kubernetes-external-secrets
# k8s-external-secrets-role
