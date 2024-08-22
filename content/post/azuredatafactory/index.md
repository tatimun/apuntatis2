---
title: Azure Data Factory y Azure DevOps
date: 2024-08-22
description: Como realizar la conexion entre ambas herramientas automaticamente
tags:
  - Azure
  - Data Factory
categories:
  - Azure
  - Cloud
  - AzureDevOps
dg-publish: true
draft: "false"
---

# ¿Qué es Data Factory?

Para tener una breve explicacion, ya que este post se trata de la conexion en si, Data Factory es como una central de distribucion de datos.

Imaginemos que es una gran ciudad, tiene diferentes fabricas alrededor de la ciudad (fuentes de datos) y cada fabrica produce diferentes tipos de productos (datos) que necesitan ser transportados a otras fabricas, tiendas o almacenes.

Data Factory seria el servicio de logistica que coordina y automatiza este transporte de productos (datos) desde las fabricas (origen de datos) hacia donde deban ir.

## ¿Por que necesito conectarlo a Azure DevOps?

En realidad, no es necesario. Pero, si brinda un esquema mas automatizado y permite no dar permisos de mas. 🤔 ¿A que se debe? 🤔

Bueno, para eso deberiamos ver como era el flujo anteriormente presentado.
### Flujo Tradicional 
[INSERTAR IMAGEN DE FLUJO ACTUAL]

En el flujo actual, el desarollador entra al ambiente de Data Factory y realiza los cambios que quiere, valida y realiza un Publish que se va a deployar automagicamente.

Esto requiere que el usuario que realice el Publish tenga permisos sobre los recursos en los cuales va a realizar cambios (Si los cambios del data factory estan conectados contra un azure storage, es necesario tener permisos sobre el Storage)

PERO EN EL NUEVO FLUJO... 🎉
### Nuevo flujo 

[INSERTAR IMAGEN DEL NUEVO FLUJO]

El desarollador, creara una rama en base a la main, donde realizara cambios, podra guardarlos y finalmente, crear un pull request hacia la main. Esto promptea una ventana en Azure DevOps para completar campos vacios del pull request. Una vez validado y aprobado, comienza el BUILD.

BUILD: Chequea la version de node, Chequea todo lo que hay actualmente y lo convierte en un ARM Template (Esto gracias al paquete NPM Utilities)

DEPLOY: Agarra lo que hay en tu pull request y compara con el ARM Template de lo que tenes actualmente, y hace los cambios para quedar acorde a la plantilla de los cambios. Agrega lo necesario y elimina lo que no esta 


