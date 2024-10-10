# Proyecto Ansible para la Instalación de Ceph (Quincy) en Debian
Esta documentación describe cómo utilizar Ansible para desplegar y configurar un clúster de Ceph en servidores Debian en su versión Quincy. A través de este proyecto, se busca automatizar la instalación de Ceph y sus dependencias, asegurando un despliegue eficiente y repetible.

## Requisitos Previos

Antes de comenzar, asegúrate de tener los siguientes requisitos en tu máquina local y en los servidores de destino:

Sistema Operativo: Debian (preferiblemente Debian 11 o superior).

Ansible: Instalado en la máquina de control.

Para instalar Ansible en Debian:

```
sudo apt update
sudo apt install ansible
```

## Configuración del Archivo /etc/hosts

Es esencial que el archivo /etc/hosts esté correctamente configurado para garantizar que los nombres de los servidores se resuelvan adecuadamente.

Pasos para configurar /etc/hosts:
Abre el archivo /etc/hosts en tu máquina local:

```
sudo nano /etc/hosts
```

Agrega las entradas para los nodos de Ceph, asegurándote de que cada dirección IP esté asociada con su nombre de host correspondiente. Por ejemplo:

```
192.168.24.101 ceph-01
192.168.24.102 ceph-02
192.168.24.103 ceph-03
```

Guarda y cierra el archivo.

## Generación de Claves SSH
Para facilitar la comunicación entre tu máquina local y los servidores remotos sin tener que ingresar una contraseña repetidamente, debes generar una clave SSH y distribuirla entre los servidores.

Pasos para generar y copiar claves SSH:
Genera una nueva clave SSH en tu máquina local:

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

o 

```
ssh-key-gen
```

Esto generará dos archivos:

~/.ssh/id_rsa: La clave privada.
~/.ssh/id_rsa.pub: La clave pública.

Copia la clave pública a los servidores remotos usando ssh-copy-id:

```
ssh-copy-id -i ~/.ssh/id_rsa.pub <server>
```

ejemplo:

```
ssh-copy-id ceph-01
```

Repite este paso para cada uno de los servidores (por ejemplo, ceph-01, ceph-02, ceph-03).

## Configuración de Inventarios

El inventario de Ansible define los servidores remotos que serán gestionados. Se recomienda tener el archivo de inventario en el directorio inventories/hosts.

Ejemplo de archivo de inventario:

```
[ceph_main]
ceph-01

[ceph]
ceph-01 ansible_host=192.168.24.51
ceph-02 ansible_host=192.168.24.52
ceph-03 ansible_host=192.168.24.53

[mon]
ceph-01
ceph-02
ceph-03

[osd]
ceph-01
ceph-02
ceph-03

[mgr]
ceph-01
ceph-02
```

5. Estructura de Archivos Importantes
Archivo ansible.cfg
Este archivo define la configuración global para Ansible y debe estar ubicado en la raíz del proyecto.

Ejemplo de archivo ansible.cfg:

```
[defaults]
inventory = inventories/hosts
remote_user = agetic
host_key_checking = False
roles_path = ./roles
private_key_file = ~/.ssh/id_rsa
```

Las variables para la instalación de Ceph deben estar definidas en group_vars/all.yml. Estas variables incluyen configuraciones específicas necesarias para la instalación.

Ejemplo de contenido para group_vars/all.yml:

```yaml
# Variables de instalación de Ceph
ceph_release: "quincy"

# URL para descargar cephadm
cephadm_download_url: "https://github.com/ceph/ceph/raw/{{ ceph_release }}/cephadm"

# Configuraciones de Ceph
ceph_initial_dashboard_user: "admin"
ceph_initial_dashboard_password: "ceph2024"
ceph_cluster_network: "192.168.24.0/26"
```

El archivo de inventario debe contener todos los nodos que formarán parte del clúster de Ceph, y se debe estructurar como se muestra en la sección anterior.

### Ejecución de Playbooks
Una vez que todo está configurado, puedes ejecutar los playbooks de Ansible para instalar y configurar Ceph.

Paso 1: Verificar la conexión con los servidores
Ejecuta el siguiente comando para asegurarte de que Ansible puede conectarse a todos los servidores especificados en el inventario:

```
ansible all -m ping
```

Paso 2: Ejecutar el playbook

Para iniciar la instalación de Ceph, utiliza el siguiente comando (suponiendo que has creado un playbook llamado ceph_install.yml en el directorio playbooks):

```
ansible-playbook -i inventories/hosts playbooks/ceph_install.yml
```

## Referencias

- [Documentación oficial de Ceph](https://docs.ceph.com/en/squid/)
- [Guía de instalación de Ceph](https://docs.ceph.com/en/latest/cephadm/install/)
