# Práctica 2d. Volúmenes Lógicos(LVM)

## Contexto
El volúmen de información a manejar crece e muchas ocasiones de forma no prevista. Una previsón a la baja en la asignación del espacio de almacenamiento en unidades físicas puede ocasionar quedarnos sin espacio con el problema que ello conlleva, una asignación a la alta supone perdida de espacio para otros fines. Mediante LVM o Logical Volumen Management obtenemos mediante una capa de abstracción sobre el volumen físico mayor flexibilidad a la hora de incrementar, reducir el espacio inicialmente previsto para nuestros datos.

En esta práctica se trabajará sobre la creación y gestión de volúmenes lógicos.

## Objetivos
* Entender que es LVM y como mejora el esquema estático de asignación de espacio.
* Explicar de forma gráfica la arquitectura de LVM.
* Crear y configrar LVM mediante comandos.
* Ser capaces de plantear un hipotético caso de ampliación de espacio en un volúmen lógico
* Documentar de forma correcta y completa los pasos llevados para realizar la práctica.


## Definición y Arquitectura de LVM

El LVM (Logical Volume Manager) es una herramienta (de Linux) que proporciona una forma flexible y avanzada de gestionar el almacenamiento en discos duros. LVM permite crear, redimensionar y mover particiones de disco de manera dinámica y sin interrumpir el servicio, algo que no es posible con las particiones tradicionales. 

Componentes principales del LVM:

Physical Volumes (PVs): Son las unidades físicas de almacenamiento, como discos duros o particiones en el disco, que se utilizan dentro de LVM.

Volume Groups (VGs): Es un grupo de volúmenes físicos (PVs). Los volúmenes físicos que forman parte de un grupo de volúmenes se tratan como un único espacio de almacenamiento.

Logical Volumes (LVs): Son los volúmenes que realmente usas para crear sistemas de archivos, montar particiones, etc. Se crean dentro de un grupo de volumenes (VG). Los volumenes logicos son equivalentes a las particiones tradicionales, pero con la ventaja de que se pueden redimensionar fácilmente y administrar sin tener que trabajar directamente con particiones de disco físicas.

![image](./basic-lvm-volume.png)

## Desarrollo
## Introducción

La practica consiste en crear 4 volumenes lógicos que pertenecerán a un mismo VG(Grupo de volumenes) a partir de 3 PVs.  Tambien vamos a extender la VG añadiendo un disco más, además de extender una lv.

 En este caso los volumenes logicos tendran los nombres: lv-web, lv-db (base de datos), lv-net y lv-projects.


### Paso 1. Pasos previos

EL primer paso consiste en agregar a la maquina virtual de vagrant existente, nuevos volumenes físicos, para ello, en el archivo vagrantfile, añadimos las siguiente líneas al sector de Vagrant.configure:

```
  config.vm.disk :disk, size: "200MB", name: "extra_storage1"
  config.vm.disk :disk, size: "200MB", name: "extra_storage2"
  config.vm.disk :disk, size: "200MB", name: "extra_storage3"
  config.vm.disk :disk, size: "200MB", name: "extra_storage4"
  config.vm.disk :disk, size: "200MB", name: "extra_storage5"
  config.vm.disk :disk, size: "200MB", name: "extra_storage6"
```

Una vez iniciado la maquina virtual con los cambios guardados y aplicados, para comprobar que los discos físicos añadidos están, ejecutamos el comando:
```
lsblk
```

Y nos mostrara todos los discos y particiones del sistemas, en esta caso buscamos los 6 discos añadidos anteriromente (sdb - sdg):

```
sdb                         8:16   0   200M  0 disk
sdc                         8:32   0   200M  0 disk
sdd                         8:48   0   200M  0 disk 
sde                         8:64   0   200M  0 disk 
sdf                         8:80   0   200M  0 disk 
sdg                         8:96   0   200M  0 disk 

```

### Paso 2. Creación de los volúmenes físicos (PVs) y grupo de volumenes (VG)

Con la verificación de los discos añadidos en el sistema, podemos crear los PVs necesarios:

```
sudo pvcreate /dev/sdb 
sudo pvcreate /dev/sdc 
sudo pvcreate /dev/sdd 
```

Con los PVs necesarios creados: Creamos el VG:
```
vgcreate vg-sad000 /dev/sdb /dev/sdc
```
Los comandos pvscan y vgscan sirven para nombrar las pvs y vgs existentes, con estos comandos podemos comprobar que se crear las PVs y VG correctamente.

### Paso 3 Crear los volumenes lógicos.

Utilizamos los siguientes comandos para crear los volumenes lógicos:

```
sudo lvcreate -n lv-projects -L 80MB vg-sad000
sudo lvcreate -n lv-web -L 40MB vg-sad000
sudo lvcreate -n lv-bd -L 100MB vg-sad000
sudo lvcreate -n lv-net -l 100%FREE vg-sad000
```

> **NOTA**: La opcion -L indica el tamaño que va a tener, 100%FREE indica que utilize el tamaño que queda del disco.

### Paso 4 Extender el grupo de volumenes.
Ahora vamos a extender el grupo de volumen con el disco sdd que añadimos antes como PV:
```
sudo vgextend vg-sad000 /dev/sdd
```
Ahora extendemos la lv de db:
```
sudo lvextend -L 200MB /dev/vg-sad000/lv-bd 
```