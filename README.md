# IBM-Cloud-Object-Storage-Clients

_Instructivo de configuración para configuración de Cloud-Object-Storage en una VM como cliente_

Para la configuación de un cliente COS se debe tener en cuenta el sistema operativo a configurar, por lo cual se divide en Linux y Windows

## Contenido 

1. Aprovisionamiento de Cloud-Object-Storage
2. Configuración de máquina virtual con SO Linux Ubuntu

## 1. Aprovisionamiento de Cloud-Object-Storage
Para el aprovisionamiento del Cloud Object Storage y Bucket se requiere del acceso a la plataforma de IBM Cloud y al catalogo del mismo.
Una vez allí busque el servicio de **object storage** y seleccionelo, se redirigirá a una pestaña para establecer los atributos necesarios los cuales serán; Nombre y Grupo de recursos.
El procedimiento se muestra a continuación:
<img width="800" alt="provision" src="Assets/provision_storage.gif"> 

Luego de ello se va a acceder al servicio y se creará un Bucket (_Cabe recalcar que el nombre del Bucket debe ser unico_)
Y posterior a ello se almacenarán algunos archivos en dicho Bucket para poder tener una referencia al configurar el Cliente Remoto de IBM-Cloud-Object-Storage
El proceso se muestra a continuación:
<img width="800" alt="bucket_creation" src="Assets/bucket_creation.gif"> 

Dentro del servicio se deberán configurar "Service Credentials" para generar autenticación a la hora de configurar el acceso a los buckets dentro de una máquina virtual. Estas credenciales deben ser creadas con HMAC keys las cuales configuran el **access key** y el **secret key** del COS
<img width="800" alt="credentials" src="Assets/credentials.gif"> 

## 2. Configuración de máquina virtual con SO Linux Ubuntu
Para una máquina con sistema operativo de Linux versión Ubuntu se requiere de la instalación del programa s3fs para la gestión de archivos, sin emabrgo la conexión se direccionará directamente al bucket en vez de a todo el servicio de Cloud Object Storage
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

Finalmente se debe realizar la configuración, sin embargo la ejecución del comando se debe direccionar hacia una carpeta totalmente vacia con los siguientes comandos:
```
    mkdir <New_Folder>
    s3fs <bucket_name> ./<New_Folder>  -o url=http{s}://<endpoint> –o passwd_file=<credentials_file>
```
<img width="800" alt="final" src="Assets/final.gif">

Con ello se observa que la carpeta aparece con archivos aunque no los tenía sin embargo para entender como funciona el Client COS se deberá subir un archivo desde la máquina en la cual se configuró y de igual manera directamente en la consola de IBM Cloud como se muestra a continuación:

<img width="1200" alt="prueba" src="Assets/prueba.gif">

