# IBM-Cloud-Object-Storage-Clients

_Instructivo de configuración para configuración de Cloud-Object-Storage en una VM como cliente_

Para la configuación de un cliente COS se debe tener en cuenta el sistema operativo a configurar, por lo cual se divide en Linux y Windows

## Contenido 

[Aprovisionamiento de ICOS Cloud-Object-Storage](#aprovisionamiento-de-cloud-object-storage)

[Opción 1 - conexión por Ubuntu](#opción-1---conexión-por-ubuntu)

1. [Configuración de máquina virtual con SO Linux Ubuntu](#configuración-de-máquina-virtual-con-so-linux-ubuntu)

[Opción 2 - conexión por Windows](#opción-2---conexión-por-windows)
1. [Creación de VSI en VPC](#creación-de-vsi-en-vpc)
3. [Configuración de disco virtual con FileMage en Microsoft Windows](#configuración-de-disco-virtual-con-filemage-en-microsoft-windows)
4. [RClone (pendiente)](#rclone)

[Referencias](#referencias)

[Autores](#autores-blacknib)

## Aprovisionamiento de Cloud-Object-Storage
Para el aprovisionamiento del Cloud Object Storage y Bucket se requiere del acceso a la plataforma de IBM Cloud y al catálogo del mismo.
Una vez allí busque el servicio de **object storage** y selecciónelo, se redirigirá a una pestaña para establecer los atributos necesarios los cuales serán Nombre y Grupo de recursos.
El procedimiento se muestra a continuación:
<img width="800" alt="provision" src="Assets/provision_storage.gif"> 

Luego de ello se va a acceder al servicio y se creará un Bucket (_Cabe recalcar que el nombre del Bucket debe ser único_)
Y posterior a ello se almacenarán algunos archivos en dicho Bucket para poder tener una referencia al configurar el Cliente Remoto de IBM-Cloud-Object-Storage
El proceso se muestra a continuación:
<img width="800" alt="bucket_creation" src="Assets/bucket_creation.gif"> 

Dentro del servicio se deberán configurar "Service Credentials" para generar autenticación a la hora de configurar el acceso a los buckets dentro de una máquina virtual. Estas credenciales deben ser creadas con HMAC keys, las cuales configuran el **access key** y el **secret key** del ICOS
<img width="800" alt="credentials" src="Assets/credentials.gif"> 

A continuación se presentan dos opciones de conexión al ICOS. La primera es por medio de un sistema operativo Linux Ubuntu, y la segunda es por medio de un disco virtual en Microsoft Windows, esta segunda opción hace uso de una VSI y utiliza los servicios de Filemage.

## Opción 1 - conexión por Ubuntu

## Configuración de máquina virtual con SO Linux Ubuntu
Para una máquina con sistema operativo de Linux versión Ubuntu se requiere de la instalación del programa s3fs para la gestión de archivos, sin embargo la conexión se direccionará directamente al bucket en vez de a todo el servicio de Cloud Object Storage
Para la instalación se accederá a la terminal y se ingrsarán el siguiente comando:
```
    sudo apt-get install automake autotools-dev fuse g++ git libcurl4-openssl-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
```
Luego de ello se deberá crear un archivo de texto sin extensión con las credenciales HMAC de la siguiente manera:

credentials_file
|
|----- <access_key>:<secret_key>

Y se le agregan permisos de lectura y escritura con el siguiente comando:
```
    chmod 600 <credentials_file>
```

Finalmente se debe realizar la configuración, sin embargo la ejecución del comando se debe direccionar hacia una carpeta totalmente vacía con los siguientes comandos:
```
    mkdir <New_Folder>
    s3fs <bucket_name> ./<New_Folder>  -o url=http{s}://<endpoint> –o passwd_file=<credentials_file>
```
<img width="800" alt="final" src="Assets/final.gif">

Con ello se observa que la carpeta aparece con archivos aunque no los tenía, sin embargo para entender como funciona el Client COS se deberá subir un archivo desde la máquina en la cual se configuró y de igual manera directamente en la consola de IBM Cloud como se muestra a continuación:

<img width="1200" alt="prueba" src="Assets/prueba.gif">


## Opción 2 - conexión por Windows
A continuación se muestran los pasos a seguir para realizar la conexión del servicio ICOS desde un disco virtual en un sistema operativo Microsoft Windows

## Creación de VSI en VPC

**Creación de VPC**

Ingrese a su cuenta de IBM Cloud, en el menú desplegable del costado izquierdo seleccione la opción *VPC infrastructure* y dentro de esta el ítem *VPCs*.
Dé click en *Create*, seleccione la región y nombre para su VPC y finalmente seleccione *Create Virtual Private Cloud*.

Ingrese a la VPC que acaba de crear y baje hasta la sección *Subnets in this VPC*, seleccione una de las subnets que se encuentran allí y copie la cadena de caracteres presentada en ```subnet ID```, esta se usará más adelante en la configuración de FileMage.

**Generación de llaves SSH**

Posteriormente para generar las llaves de acceso SSH ingrese a una terminal, ya sea IBM Cloud Shell o la terminal de su computador. Ingrese el siguiente comando:

```
ssh-keygen
```
Puede ingresar la ubicación en la que desea almacenar las llaves SSH y la contraseña de las mismas, o almacenarlas en la ubicación actual presionando enter.

A continuación regrese a su cuenta de IBM Cloud, en el menú de la izquierda dé click en *SSH keys* y posteriormente en *Create*. Ingrese la siguiente información:
- **Location**: Seleccione la misma ubicación de la VPC que acaba de crear
- **Name**: Asigne un nombre a su llave SSH
- **Resource Group**: Seleccione el mismo grupo de recursos donde está su VPC.
- **Public Key**: Ingrese *todo* el contenido del archivo llamado ```id_rsa.pub```, el cuál estpa ubicado en la carpeta ```.ssh``` que creó desde la terminal en el paso anterior.

<img width="800" alt="SSH key" src="/img/SSH.png">

**Creación del servicio FileMage Gateway**

Ingrese al catálogo de IBM Cloud y busque el servicio *FileMage Gateway*, asígnele un nombre, grupo de recursos y ubicación y dé click en *Install*

<img width="800" alt="filemage" src="/img/filemage_catalogo.png">

Luego de crear su servicio de Filemage diligencie la siguiente información:
- **create_floating_ip**: true
- **vsi_instance_name**: ingrese el nombre que desea asignar a la Virtual Server Instance (VSI) que se creará.
- **vsi_security_group**: ingrese el nombre que sedea asignar al security group que se creará.
-**region**: escriba el nombre de la región en la cual creó su VPC.
-**ssh_key_name**: ingrese el nombre de la llave SSH que creó en IBM Cloud en el paso anterior.
-**subnet_id**: ingrese la subnet_id que copió al momento de crear su VPC. Este ID también lo puede encontrar ingresando por el menú desplegable de la izquierda, sección *VPC Infrastructure*, opción *VPCs*, ingresando a la VPC correspondiente y posteriormente seleccionando una de las subnets.

Finalmente, dé click en *Generate plan*

<img width="800" alt="filemage" src="/img/schematics.png">

Luego de que termina la creación del Filemage Schematics, dé click en *Apply plan*. Recuerde que debe tener los permisos adecuados para poder ejecutar el schematics.

Puede verificar la adecuada ejeución del Schematics ingresando en el menú desplegable de la izquierda en la sección *VPC Infrastructure*, opción *Virtual Server Instances*. Acá debería ver creada una VSI con el nombre que asgnó al configurar el schematics. 

<img width="800" alt="filemage" src="/img/VSI.png">

Finalmente, ingrese a la VSI recién creada y copie la dirección IP flotante que se encuentra en la sección **network interfaces**, esta será la dirección que se usará para acceder al servicio de FileMage.

## Configuración de disco virtual con FileMage en Microsoft Windows 

## RClone

## Referencias

- [SSH Key](https://cloud.ibm.com/docs/vpc?topic=vpc-ssh-keys)

## Autores :black_nib:
Equipo IBM Cloud Tech Sales Colombia.
<br />