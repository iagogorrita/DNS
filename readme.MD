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
La subred que tendra esta red será **172.28.0.0/24** y el gateway será **172.28.0.1**

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
###
## Pruebas

Para hacer pruebas tendremos que utilizar el *dig* y el *ping* por lo que en los dos containers deberemos instalar lo siguiente:

**Attach shell**

`$ docker exec -it asir_cliente bash`

`$ apt update`

`$ apt install -y dnsutils`

`$ apt install -y iputils-ping`

###
## Configuración del DNS
Para configurar nuestro DNS deberemso tener un minimo de 2 carpetas, una conf que tendrá los archivos que irán en el directorio */etc/bind/* y otro que será zones que estará vinculado al directorio */var/lib/bind/*, dentro de conf tendremos cuatro archivos: **named.conf, named.conf.default-zones, named.conf.local y named.conf.options** y dentro de zones que tendremos el archivo db.web1asir.int, que será donde tendremos que meter todo lo que querramos que resuelva nuestro DNS.
### named.conf
Este archivo nos dice que archivos estan incluidos en el volumen de conf de nuestro dns, por lo cual si no se incluye un archivo, no se utilizará y no se tendrá en cuenta para la configuración del dns.
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
### named.conf.local

El archivo named.conf.local se utiliza para configurar zonas locales o específicas del servidor DNS. Cada zona representa un dominio o un conjunto de dominios para los cuales el servidor DNS es responsable.
```
zone "web1asir.int" {
	type master;
	file "/var/lib/bind/db.web1asir.int";
	allow-query {
		any;
		};
	};
```
### named.local.options

El archivo named.conf.options en BIND (Berkeley Internet Name Domain) se utiliza para configurar las opciones globales del servidor DNS. Este archivo contiene configuraciones que afectan el comportamiento general del servidor DNS, incluyendo parámetros de rendimiento, seguridad, resolución de nombres y otros aspectos clave.

```
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1; 
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```
### named.local.default-zones
En BIND (Berkeley Internet Name Domain), el archivo named.conf.default-zones se utiliza para definir las configuraciones predeterminadas de algunas zonas de nombres (domains) especiales y de importancia general en el sistema DNS. Estas configuraciones se aplican de forma predeterminada si no se especifican configuraciones específicas para esas zonas en otros archivos de configuración.
```
// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};
```
### db.web1asir.int
En este caso, el archivo podría llamarse db.web1asir.int y contendría los registros de recursos asociados con el dominio web1asir.int. Estos registros podrían incluir direcciones IP, registros de correo (MX), alias (CNAME), etc.
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.web1asir.int. igorrita.danielcastelao.org. (
				20231009   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.web1asir.int.
ns		IN A 	172.28.0.40
iago	IN A    10.0.9.5
mario 	IN A 	10.0.9.4
```