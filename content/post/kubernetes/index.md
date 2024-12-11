---
title: Kubernetes
description: Parte 1. Bases de K8S (RHEL)
date: 2024-08-19
categories:
  - Kubernetes
  - RHELCapacitacion
tags:
  - Kubernetes
  - Capacitacion
draft: "true"
---
Que es un pod?

Agrupacion logica de contenedores
Podemos tener 1 o varios contenenodres, en esta agrupacion, todos los contenedores compartiran la misma IP y todo el espacio, recursos del pod

Al pod se le añade, un uso de cpu y memoria al momento de crearlo, los containers dentro compartiran los recursos y el ciclo de vida. Es decir, si mato el pod, mueren todos los contenedores.

Deploymnet
Mira que nuestro pod, siempre tenga el estado que le hemos definido 

Service
Nuestras IP y DNS estatico, es como una especie ed balanceador, agrupador

Volumenes, los contenedores son efimeros, al igual que su disco A MENOS que utilicemos un volumen persistente

Label
Duplas clave valor, añadir etiquetas a los objetos 

## Kubernetes Cluster Node

Primary node:
Nodos encargados de gestionar el cluster

api:
etc: base de datos distribuida se guarda el estado del cluster
scheduler: kubernetes recibe las instrucciones de desplegar al recibir la instruccion, busca el mejor nodo para desplegarlo 
controllers: verificar lo que dije, realmente este pasando 

Worker nodes:
Los nodos donde vamos a ejecutar los contenedores


