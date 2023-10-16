## SRI :yum:

## Practica DNS

###

Para crear el **servidor DNS** lo haremos en Docker con la imagen del *Internet System Consortium* y crearemos además un container de la imagen mas reciente de *Ubuntu* y le pondremos de dns la IP de nuestro servidor DNS.

###
## Volumenes
En el servidor DNS tendremos los siguientes **volumenes**:
```
volumes:
      - ./conf:/etc/bind/
      - ./zones/:/var/lib/bind/
```
Tendremos una carpeta conf en nuestro equipo que estara vinculado al directorio */etc/bind/* que será la que le dé la configuración a nuestro DNS, con su configuracion local, y si no encuentra en donde deberá resolver.

Y tendremos la carpeta zones, que será la que tendremos que configurar con los nombres que querramos y que los pueda resolver.

## Redes
En la *configuracion general de red* del docker compose tendremos lo siguiente:
```
networks:
  network:
    ipam:
      config:
        - subnet: 172.28.0.0/24
          gateway: 172.28.0.1
```
En la *configuracion de red del DNS* tendremos que poner lo siguiente:
```
networks:
      network:
        ipv4_address: 172.28.0.40
    ports:
      - '53:53/tcp'
      - '53:53/udp'
```
La ip de nuestro servidor DNS será **`172.28.0.40`**, que será donde debemos configurar el DNS en nuestro container ubuntu para que lo resuelva.

En la *configuracion de red del Ubuntu* tendremos lo siguiente:
```
networks:
      network: 
        ipv4_address: 172.28.0.70
dns:
      - 172.28.0.40
```
Nuestro container de Ubuntu tendra la ip **`172.28.0.70`** y donde estará el DNS será en **`172.28.0.40`**, es decir en nuestro servidor DNS por lo que el que resolverá las Ips será nuestro DNS y no nuestro localhost.


