# <div align="center">Bienvenidos a Oracle Racing to the Cloud </div>
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/red-bull-rb18.jpg)
## Laboratorio container instances 
### Despliegue solución Wordpress con MySQL
El laboratorio consiste en el despliegue de dos containers dentro de un container instances, el primer contenedor almacenara la solución de Wordpress y el segundo contenedor la base de datos MySQL. 
La seguridad es lo más importante por lo que estos contenedores estarán en una subred privada y la solución será expuesta por un balanceador de cargas público como protección a posibles ciberataques!!!

La siguiente imagen muestra la arquitectura objetivo:
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Arquitectura.png)

#### Prerrequisitos para la realizacion del laboratorio
* Creacion de un Compartment
* Creacion de VCN y subredes, una publica y una privada
* Configuracion de Security Lists:
  + Para la subred publica permitir el trafico por los puertos 80 y 3306 desde cualquier origen
  + Para la subred privada permitir el trafico por los puertos 80 y 3306 desde la VCN
  
### 1. Prerrequisitos
#### 1.1 Compartment:
**Menu principal > Identity & Security > Compartments > Create Compartment**</br>
Crearemos el compartment donde aprovisionaremos todos los recursos
* **Name:** LabContainers

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Compartment.png)

#### 1.2 Red:
**Menu principal > Networking > Virtual Cloud Networks**</br>
Asegurarnos que estamos utilizando el compartment previamente creado
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN.png)

Seleccionaremos la opción de Start VCN Wizard, dejaremos la selección por defecto “Create VCN with Internet Connectivity” e introduciremos los siguientes valores:
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN1.png)

* **VCN name:** VCN
* **VCN IPv4 CIDR block:** 30.0.0.0/16
* **Public subnet - IPv4 CIDR block:** 30.0.0.0/24
* **Private subnet - IPv4 CIDR block:** 30.0.1.0/24

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN2.png)

Una vez finalizada la creación de la VCN procederemos a ver la VCN y configurar las listas de seguridad.
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN3.png)

Y nos ubicamos en las sudredes
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN5.png)

**Subred Privada:** Navegar a Subnets -> private subnet-VCN -> security list for private subnet-VCN  -> Add Ingress Rule y agregar la siguiente ruta
* **Source CIDR:** 30.0.0.0/16
* **Destination Port Range:** 80,3306
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/PrivateSL.png)

**Subred Publica:** Procedemos a regresar a las subredes y Navegar a Subnets -> public subnet-VCN -> Default Security List for VCN  -> Add Ingress Rule y agregar la siguiente ruta
* **Source CIDR:** 0.0.0.0/0
* **Destination Port Range:** 80,3306
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/PublicSL.png)

### 2. Creación de Container Instance
**Menu principal >Developer Services > Container instances > Create container instance**
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/ContainerInstance.png)

#### 2.1 Configuración de la instancia
Debemos ingresar la informacion del nombre de la instancia, AD, Shape y capacidades de computo(OCPU y Memoria RAM)
* **Name:** WordPress
* **OCPUs:** 2
* **Memory:** 16

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container1.png)

En la parte de Networking seleccionamos la VCN y la subred privada, además en las opciones avanzadas habilitar Always
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container2.png)

#### 2.2 Configuración de los contenedores
En esta parte vamos a asignar los nombres de los contenedores, para ello seleccionamos las imagenes a utilizar y creamos las variables de ambiente que necesita el contenedor para funcionar adecuadamente. Para el laboratorio vamos a utilizar las imagenes publicas del Docker Hub

##### 2.2.1 El primer container a crear es el de Wordpress
Asignamos un nombre al container y seleccionamos la imagen a descargar desde el Docker Hub presionando Select Image
* **Name:** wordpress
* **Image:** wordpress
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container3.png)
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container4.png)

A continuación, configuremos algunas variables de entorno como se describe en la sección Cómo usar esta imagen en [la documentación de la imagen](https://hub.docker.com/_/wordpress/ "") . Se establecen las siguientes variables y valores.
Variable  | Value
------------- | -------------
WORDPRESS_DB_HOST  | 127.0.0.1
WORDPRESS_DB_USER  | wordpress
WORDPRESS_DB_PASSWORD  | wordpress
WORDPRESS_DB_NAME  | wordpress

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container5.png)
Click en + Another container

##### 2.2.2 El segundo container a crear es el de MySQL
Asignamos un nombre al container y seleccionamos la imagen a descargar desde el Docker Hub presionando Select Image
* **Name:** mysql
* **Image:** mysql
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container6.png)
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container7.png)

A continuación, configuremos algunas variables de entorno como se describe en la sección Cómo usar esta imagen en [la documentación de la imagen](https://hub.docker.com/_/mysql/ "") . Se establecen las siguientes variables y valores.

Variable  | Value
------------- | -------------
MYSQL_ROOT_PASSWORD | wordpressonmysql
MYSQL_DATABASE | wordpress
MYSQL_USER  | wordpress
YSQL_PASSWORD  | wordpress

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container8.png)
Además también se debe de expandir las opciones avanzadas > Statup options y agregar el siguiente comando en la sección Entrypoint arguments
```sh
--default-authentication-plugin=mysql_native_password 
```
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container9.png)
Click Next

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container10.png)
Click en Create

Una vez finalizado el flujo de creación podremos ver la información de nuestro Conteiner Instance y los Containers creados.
Favor tomar nota de la Private IP address que utilizaremos en los balanceadores
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container11.png)

### 3. Creación del Load Balancer

Para acceder de forma publica al servicio de Wordpress es necesario configurar un Load Balancer para recibir el trafico desde internet
**Menu principal > Networking > Load Balancers**
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_1.png)

Asignamos el nombre del LB y seleccionamos las opciones de visibilidad publica e IP
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_2.png)

Seleccionamos la VCN y la subred publica ceeada durante los prerrequisitos
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_3.png)

Luego seleccionamos el algoritmo de balanceo del trafico y el protocolo y puerto para el Health Check
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_4.png)

Configuramos el listener de tipo HTTP (las peticiones ingresan por el puerto 80)
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_5.png)

Activamos los logs del LB
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/create_lb_6.png)

Una vez completado el flujo de creación entraremos al balanceador, navegaremos a la sección de Backend Sets, ingresaremos al creado por default 
![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_7.png)

El backend corresponde a la IP privada del Container Instance
![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_8.png)



#### Material adicional:

- [Containers](https://docs.oracle.com/en/learn/manage-oci-container-instances/index.html#task-1-create-and-configure-a-container-instance/) - OCI Container Instances 
