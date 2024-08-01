---
title: 1. Introducción a Ansible
date: 2024-07-31
description: Aprendiendo Ansible
tags:
  - Automatizacion
categories:
  - Automatizacion
dg-publish: true
draft: "false"
image: banner.jpg
---
### ¿Qué es Ansible?

### Instalación

~~~
sudo yum install ansible

ansible --version
~~~
### Comandos
Nos genera un archivo en /etc/ansible/hosts que contendra los servers a los cuales nos conectaremos

~~~
# Pinguea hacia los hosts en este archivo
ansible all -m ping

# Lista los hosts conectados
ansible all --list-hosts


~~~
### ¿Como funciona?



 

