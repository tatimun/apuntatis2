---
title: Openshift + GitOps
date: 2024-09-23
description: Openshift
tags:
  - Kubernetes
categories:
  - Kubernetes
  - Openshift
dg-publish: true
draft: "false"
banner: image.png
---
~~~
crc setup
crc config set memory 16384 
crc config set cpus 4
~~~
Para poder avanzar y levantar con Argo

~~~
crc start
~~~

1. Instalar operador de GitOps
2. Crea un namespace default, y algunos service accounts

Si no corre, fijarse los pods
~~~
oc get pods -n openshift-gitops
~~~


3. Crear service account 


~~~
# Obtener los service accounts 
oc get sa -n openshift-gitops


#Sacar token para el service account
oc sa create token test-service-account -n openshift-gitops 

#login con el token
oc login --token=<TOKEN> --server=https://api.crc.testing:6443


#Obtener role binding 
oc get rolebinding gitops-admin-binding -n openshift-gitops -o yaml




~~~
## ¿ Que es un role binding?

Tenes roles, que serian los permisos
y el rol binding que asocia los roles con las personas

Podes ver en los role bindings, quien tiene permiso para hacer que sobre que namespace


Se ve algo asi:

~~~YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2024-09-23T17:26:18Z"
  name: gitops-admin-binding
  namespace: openshift-gitops
  resourceVersion: "36927"
  uid: 6df50da5-5fd0-40d4-b213-08f009dd89cb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: admin
subjects:
- kind: ServiceAccount
  name: gitops-admin
  namespace: openshift-gitops
- kind: ServiceAccount
  name: test-service-account
  namespace: openshift-gitops
  ~~~

## ClusterRoleBinding

Tambien existe el clusterole binding, que es para configuraciones del cluster, roles superiores, el role se limita solo a namespaces

---

~~~
oc create rolebinding gitops-admin-binding  --clusterrole=admin  --serviceaccount=openshift-gitops:test-service-account  --namespace=openshift-gitops

oc auth can-i list pods -n openshift-gitops --as system:serviceaccount:openshift-gitops:test-service-account







~~~


Ir a argo y a settings

![[Pasted image 20240923163810.png]]


Generar key para Gitlab y subirla aca https://gitlab.com/-/user_settings/ssh_keys/

Con el comando: 

~~~
ssh-keygen -t rsa -b 4096 -C "tu_correo@example.com"


```
argocd repo add git@gitlab.com:tatimunoz/gitops.git --ssh-private-key-path ~/.ssh/id_rsa
```


~~~

