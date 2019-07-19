# Asociar Dominio Web y configurar SSL en instancia EC2 (Ubuntu 18.04.1 LTS / NGINX)

>
**Requisitos previos para continuar con el tutorial**
1. Tener una instancia EC2 (Ubuntu 18.04.1 LTS)
2. En la instancia EC2 configurar el servidor `NGINX`
3. Comprar un dominio web en algún sistema como goodaddy, hostgator, etc 
>


##Configurar dominio en EC2 (usando Route 53)

Primero que nada es importante tener creada la instancia EC2 (Ubuntu 18.04.1 LTS) y tener accesos a la consola de Amazon Web Services.

**1. Crear instancia EC2**
En el Security Group de la instancia agregar la regla 

```
HTTP | Puerto 80 | Source: 0.0.0.0/0
```

**2. Route53**
En la consola de Amazon Web Services dirigirse al servicio Route53 y crear una nueva `Hosted Zone` como se muestra a continuación

![Img - Create Hosted Zone](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/create-hosted-zone.png)

Una vez creada la Hosted Zone, se te proporcionarán valores como estos

![Img - Hosted Zone Values](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/hosted-zone-values.png)

**3. Panel Admin del dominio**
Dirigirte a panel de administración de tu dominio (ej. Godaddy) y actualiza tu registro del Dominio con los valores que proporciona la configuración  de Route53

Primero actualiza el Domain Nameserver y pon Server type: custom

![Img - Domain Nameserver Server type as custom](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/domain-nameserver-type-custom.png)

Luego copia los valores de los campos de la “Hosted Zone” de Route53 a cada uno de los nameservers (4)

![Img - Nameservers](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/nameservers-4.png)

>
Nota: estos cambios pueden tomar hasta 24h para propagarse, pero normalmente toman unos pocos minutos
>

**4. Crear los conjuntos de registro DNS en Route53**
Copiar la IP publica de la instancia EC2

![Img - Public IP](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/ec2-public-ip.png)


Dirijirte a la Hosted Zone en Route53

1. Crear un “record set” para asociar el tudominio.com a la IP pública de la instancia EC2
	
	Poner www en el campo “name”
	Poner la dirección IP en el campo “value” del “record set”
	
	![Img - Record set 1](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/record-set-1.png)
	
2. Crear un “record set” alias de tudominio.com a www.tudominio.com 
	
	Dejar el campo “name” en blanco
	Seleccionar el radiobutton Alias=yes
	En el campo ”Alias target” poner www.tudominio.com
	Routing policy=simple
	
	![Img - Record set alias](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/record-set-alias.png)


¡Listo, probar desde una consola/terminal en tu equipo poniendo el dominio en lugar de la IP pública!

