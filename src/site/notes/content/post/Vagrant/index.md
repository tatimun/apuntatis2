---
{"dg-publish":true,"permalink":"/content/post/vagrant/index/","title":"Vagrant","tags":["Vagrant","Hashicorp","DevOps"]}
---


### ¿Qué es Vagrant?
- Es una herramienta para crear y administrar maquinas virtuales o ambientes de estas
- Se crean en tu maquina local o dentro de un provider
- Puede utilizar shell scripts, Chef o Puppet
- Algunos providers:
	-  VirtualBox
	- Docker
	- HyperV
Incluso se puede usar custom providers



###  Comandos Importantes

Inicializa el directorio actual para crear un ambiente de Vagrant creando un vagrantfile
```
vagrant init 
```

 Crea y configura la maquina virtual siguiendo el vagrantfile
```
vagrant up
```
Detiene y destruye todo los recursos creados
```
vagrant destroy 
```

```
vagrant validate # Chequea la sintaxis del vagrantfile
```

```
vagrant provision # Lo corre en los providers configurados
```

```
vagrant reload # Detiene y vuelve a levantar 
```

```
vagrant status # Te dice el estado de tus maquinas 

```

```
vagrant ssh 
```


 Te conectas pos ssh a la maquina que se haya levantando en el directorio donde esta el vagrantfile
```




