# Contenidos mini curso DevOps
## La máquina
**OS**: CentOS 7.4
**Usuarios**:
* user: nein
* root: nein

**Setup**:
* Tener VirtualBox, VMWare o cualquier gestor de MV que lea OVAs
* Añadir una interfaz de red desde la que acceder por ssh, se recomienda host-only (virtualbox)
* **Y ya está**, a menos que no tengas las extensiones de VirtualBox, _que en tal caso deberás replantearte la vida y si te merece la pena este curso._ Bueno venga, descarga e instala las extensiones [Downloads – Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
## Git
### ¿Qué es Git?¿Y Gitlab / Github?
### Comandos básicos
#### Set-up Git
```
git config --global user.name "DevOps"
git config --global user.email "operator@gfi.es"
```
#### Descargar un repositorio
```
git clone git@ec109008c1c1:root/bbva-psd2.git
```
#### Trabajar con el repositorio
```
git flow init # genera varias ramas para desarrollar
git status # comprueba el estado actual del repositorio
git checkout master # cambia a la rama de master
git checkout develop # cambia a la rama de develop
git branch testing # crea una nueva rama llamada 'testing'
git flow feature start holaMundo # comienza con la 'feature' de 'holaMundo'
git status
vim helloWorld.py
git status
git add -A # añade los cambios realizados
git commit -m "Implemented greeting in python" # ""guarda"" los cambios realizados con un comentario
git status
git flow feature finish helloWorld # finaliza la 'feature' y mezcla la rama con la de desarrollo, borrando la rama helloWorld
git status
git checkout master
git merge develop # mezcla master con develop
git status
```
## Ansible
### ¿Qué es Ansible?
_Ansible es un motor de automatización de TI radicalmente simple que automatiza el aprovisionamiento en la nube, la administración de configuración, la implementación de aplicaciones, la orquestación dentro del servicio y muchas otras necesidades de TI._
### ¿Para qué sirve Ansible?
* Realizar instalaciones de forma masiva, asegurando la consistencia entre máquinas y entornos
* Configurar entornos a gran escala
* Realizar tareas de administración (básicas y avanzadas) repetitivas o tediosas

_pero si ya tenemos shellscript, para qué vienes a contarnos esta milonga, eh? Pichón???_
### ¿Cómo usar Ansible?
* Lo primero es instalarlo:
  * {apt | yum} install ansible -y
* Lo segundo es configurar un inventario o utilizar el que viene por defecto

_/etc/ansible/hosts_
```
[learning]
master	ansible_connection=local ansible_user=user
slave	ansible_connection=ssh	ansible_host=192.168.56.102	ansible_user=user
```
#### Modulos
Hay que saber cómo buscarlos, no hay que trabajar el doble:
_"[Module Index — Ansible Documentation](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)"_
* apt
* yum
* file
* shell
* script
* copy
* lineinfile
* ec2
* aws_s3
#### Comandos básicos
```
ansible --version # muestra la versión
ansible all -m ping # comprueba que puede acceder a los hosts
ansible all -m ping -i inventory
ansible all -m setup -i inventory
```
#### Playbooks y el infierno
Los [playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html) son los archivos / scripts que contienen una concatenación de comandos y variables que puede leer Ansible y vienen escritos en **[YAML](https://learnxinyminutes.com/docs/yaml/)**.

_"Los playbooks pueden declarar configuraciones, pero también pueden organizar los pasos de cualquier proceso ordenado manualmente, incluso cuando diferentes pasos deben rebotar entre series de máquinas en órdenes particulares. Pueden iniciar tareas de forma síncrona o asíncrona."_

Ejemplo de un playbook:
```
---
# this playbook installs docker on a centOS machine
- hosts: learning
  become: true
  tasks:
  - name: Removes old versions of docker
    yum:
      name: "{{ item }}"
      state: absent
    with_items:
      - docker
      - docker-client
      - docker-client-latest
      - docker-common
      - docker-latest
      - docker-latest-logrotate
      - docker-logrotate
      - docker-selinux
      - docker-engine-selinux
      - docker-engine
    when: ansible_pkg_mgr == 'yum'
  - name: Installs required packages to add the docker-ce repository
    yum:
      name: "{{ item }}"
      state: latest
    with_items:
      - yum-utils
      - device-mapper-persistent-data
      - lvm2
    when: ansible_pkg_mgr == 'yum'

  - name: Enables the docker-ce repository
    yum_repository:
      name: docker-ce-stable
      description: docker-ce official repository
      baseurl: https://download.docker.com/linux/centos/7/$basearch/stable
      enabled: yes
    when: ansible_pkg_mgr == 'yum'

  - name: Installs docker-ce
    yum:
      name: docker-ce
      state: latest
      disable_gpg_check: yes
    when: ansible_pkg_mgr == 'yum'
```
#### Ejecución Playbooks y pruebas
```
ansible-playbook install-docker.yml
ansible-playbook playbook.yml -e "var1=hola"
```
#### Roles y porqué usarlos
Los roles son una estructura de carpetas que contienen distintos playbooks, para una ejecución modular y ordenada a la cuál se le pueden ir configurando variables.

Los roles suelen contener las siguientes carpetas:
* tasks - contiene los playbooks a ejecutar por el rol
* defaults - variables por defecto que utiliza el rol
* vars - carpeta adicional de variables que se suelen modificar para cada ejecución
* files - contiene los archivos que se utilizan durante la ejecución del rol
* templates - contiene las plantillas que se despliegan con el rol
* meta - define los metadatos del rol

#### Ejecución Roles
```
ansible-playbook role.yml
```
#### Ansible Tower
_DEMO TIME !_ :D
## Docker
### ¿Qué es Docker?¿Para qué sirve?
Docker es una plataforma de contenedores _(contenedoqué?)_ de código libre. Los contenedores son paquetes ligeros, autónomos y ejecutables de un software, que incluyen todo lo necesario para ejecutar dicho software: código, tiempo de ejecución, herramientas del sistema, bibliotecas del sistema, configuraciones.

A diferencia de las máquinas virtuales, los contenedores corren sobre el mismo kernel que los contiene, consumiendo menos recursos. Además, utilizan el propio sistema de archivos del anfitrión, llegando a ocupar menos espacio.

Las imágenes de Docker permiten un despliegue más rápido y homogéneo de plataformas, asegurando la estabilidad de la arquitectura a desplegar, lo que viene perfecto para microservicios y el desarrollo/despliegue contínuo.
### Administración y comandos básicos
```
docker run hello-world
# desplegar un cluster de mysql
docker network create cluster --subnet=192.168.0.0/16
docker run -d --net=cluster --name=management1 --ip=192.168.0.2 mysql/mysql-cluster ndb_mgmd
docker run -d --net=cluster --name=ndb1 --ip=192.168.0.3 mysql/mysql-cluster ndbd
docker run -d --net=cluster --name=ndb2 --ip=192.168.0.4 mysql/mysql-cluster ndbd
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 -e MYSQL_RANDOM_ROOT_PASSWORD=false MYSQL_ROOT_PASSWORD=nein mysql/mysql-cluster mysqld
docker logs mysql1 2>&1 | grep PASSWORD
# fin del despliegue
docker ps
docker ps -aq
docker kill container-id
docker rm -f $(docker ps -aq)
docker rm container-id
docker stats
```
### El dockerfile
El dockerfile es a Docker lo que el playbook a Ansible, _pero distinto_. Es donde se selecciona la imagen base, se configura la imagen, se instala y configura el software, et... Es decir, _es_ como un playbook de Ansible, _pero_ para Docker. Y sí, viene con su ~~infierno particular~~ yaml.
```
version: '2.2'
services:
  api-gateway:
    image: caapim/gateway
    cpus: 4
    mem_limit: 6g
    ports:
      - "8080"
      - "8443"
      - "9443"
    volumes:
     - /opt/SecureSpan/Gateway/node/default/etc/bootstrap/services/restman
    environment:
      ACCEPT_LICENSE: "false"
      SSG_ADMIN_USERNAME: "pmadmin"
      SSG_ADMIN_PASSWORD: "nein"
      SSG_DATABASE_JDBC_URL: "jdbc:mysql://mysql-server:3306/ssg"
      SSG_DATABASE_USER: "ssgdbuser"
      SSG_DATABASE_PASSWORD: "dbpassword"
      SSG_CLUSTER_HOST: "gfi.gateway.test"
      SSG_CLUSTER_PASSWORD: "clusterpassword"
      SSG_JVM_HEAP: "4g"
      EXTRA_JAVA_ARGS: "-XX:ParallelGCThreads=4 -Dcom.l7tech.bootstrap.autoTrustSslKey=trustAnchor,TrustedFor.SSL,TrustedFor.SAML_ISSUER"

  mysql-server:
    image: mysql:5.7
    mem_limit: 512m
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_USER=ssgdbuser
      - MYSQL_PASSWORD=dbpassword
      - MYSQL_DATABASE=ssg
    command:
      - "--character-set-server=utf8"
      - "--innodb_log_buffer_size=32M"
      - "--innodb_log_file_size=80M"
      - "--max_allowed_packet=8M"
```
### Trasteo vario
## Examen _prepare to cry_ edition
Sí, hay examen de lo que acabo de contar, hay que tener 5/7 para aprobar.

**Preguntas**
Para el final de clase ;)
