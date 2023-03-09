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
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Compartment.png)

#### 1.2 Red:
**Menu principal > Networking > Virtual Cloud Networks**
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN.png)
Seleccionaremos la opción de Start VCN Wizard, dejaremos la selección por defecto “Create VCN with Internet Connectivity” e introduciremos los siguientes valores:
* **VCN name:** VCN
* **VCN IPv4 CIDR block:** 30.0.0.0/16
* **Public subnet - IPv4 CIDR block:** 30.0.0.0/24
* **Private subnet - IPv4 CIDR block:** 30.0.1.0/24
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN1.png)
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN2.png)

Una vez finalizada la creación de la VCN procederemos a configurar las listas de seguridad.

Ingresamos a la VCN creada
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN3.png)

Y nos ubicamos en las sudredes
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/VCN5.png)

**Subred Publica:** Navegar a Subnets -> public subnet-VCN -> Default Security List for VCN  -> Add Ingress Rule y agregar la siguiente ruta
* **Source CIDR:** 0.0.0.0/0
* **Destination Port Range:** 80,3306
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/PublicSL.png)

**Subred Privada:** Procedemos a regresar a las subredes y  Navegar a Subnets -> private subnet-VCN -> security list for private subnet-VCN  -> Add Ingress Rule y agregar la siguiente ruta
* **Source CIDR:** 30.0.0.0/16
* **Destination Port Range:** 80,3306
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/PrivateSL.png)

### 2. Creación de Container Instance
**Menu principal >Developer Services > Container instances**
![](https://github.com/jevargascr/ContainerInstances/blob/main/images/ContainerInstance.png)

#### 2.1 Configuración de la instancia
Debemos ingresar la informacion del nombre de la instancia, AD, Shape y capacidades de computo(OCPU y Memoria RAM)
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

Configuracion de las variables de ambiente necesarias para el despliegue del container Wordpress</br>
El valor de la variable WORDPRESS_DB_HOST corresponde a la IP seleccionada durante la creacion del Container Instance en la parte de Networking
Variable  | Value
------------- | -------------
WORDPRESS_DB_HOST  | 127.0.0.1
WORDPRESS_DB_USER  | wordpress
WORDPRESS_DB_PASSWORD  | wordpress
WORDPRESS_DB_NAME  | wordpress

![](https://github.com/jevargascr/ContainerInstances/blob/main/images/Container5.png)
Click en + Another container

##### 2.2.2 El primer container a crear es el de MySQL
Asignamos un nombre al container y seleccionamos la imagen a descargar desde el Docker Hub
![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_container_2.jpg)

Configuracion de las variables de ambiente necesarias para el despliegue del container MySQL

Variable  | Value
------------- | -------------
MYSQL_ROOT_PASSWORD | wordpressonmysql
MYSQL_DATABASE | wordpress
MYSQL_USER  | wordpress
YSQL_PASSWORD  | wordpress

Click en crear another container

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/mysql-vars.jpg)

### 2.1.2 El segundo container a crear es el de Wordpress
Asignamos un nombre al container y seleccionamos la imagen a descargar desde el Docker Hub

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_container_5.jpg)

Configuracion de las variables de ambiente necesarias para el despliegue del container Wordpress
El valor de la variable WORDPRESS_DB_HOST corresponde a la IP seleccionada durante la creacion del Container Instance en la parte de Networking
Variable  | Value
------------- | -------------
WORDPRESS_DB_HOST  | 127.0.0.1
WORDPRESS_DB_USER  | wordpress
WORDPRESS_DB_PASSWORD  | wordpress
WORDPRESS_DB_NAME  | wordpress

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_ci_11.jpg)

Click en Create

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_container_7.jpg)

# 2. Creación del Load Balancer

#### Para acceder de forma publica al servicio de Wordpress es necesario configurar un Load Balancer para recibir el trafico desde internet

### Menu principal > Networking > Load Balancers

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_1.jpg)

Asignamos el nombre del LB y seleccionamos las opciones de visibilidad publica e IP

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_2.jpg)

Seleccionamos la VCN y la subred publica ceeada durante los prerrequisitos

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_3.jpg)

Luego seleccionamos el algoritmo de balanceo del trafico y el protocolo y puerto para el Health Check

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_4.jpg)

Configuramos el listener de tipo HTTP (las peticiones ingresan por el puerto 80)

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_5.jpg)

Activamos los logs del LB

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_6.jpg)

Adicionamos el Backend, este corresponde a la Container Instance

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_7.jpg)

El backend corresponde a la IP privada del Container Instance

![](https://github.com/johncdoracle/RacingToCloud/blob/main/images/create_lb_8.jpg)






#### Material adicional:

- [Containers](https://docs.oracle.com/en/learn/manage-oci-container-instances/index.html#task-1-create-and-configure-a-container-instance/) - OCI Container Instances 
