# Asociar Dominio Web y configurar SSL en instancia EC2 (Ubuntu 18.04.1 LTS / NGINX)

>
**Requisitos previos para continuar con el tutorial**
1. Tener una instancia EC2 (Ubuntu 18.04.1 LTS)
2. En la instancia EC2 configurar el servidor `NGINX`
3. Comprar un dominio web en algún sistema como goodaddy, hostgator, etc 

>


## Configurar dominio en EC2 (usando Route 53)

Primero que nada es importante tener creada la instancia EC2 (Ubuntu 18.04.1 LTS) y tener accesos a la consola de Amazon Web Services.

**1. Crear instancia EC2**

En el Security Group de la instancia agregar las reglas

```
HTTP 		| TCP 		| Puerto: 80 		| Source: 0.0.0.0/0
HTTPS		| TCP		| Puerto: 443		| Source: 0.0.0.0/0
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

1. Crear un `record set` para asociar el _tudominio.com_ a la IP pública de la instancia EC2
	
	Poner www en el campo “name”
	Poner la dirección IP en el campo “value” del “record set”
	
	![Img - Record set 1](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/record-set-1.png)
	
2. Crear un `record set` alias de _tudominio.com_ a _www.tudominio.com_
	
	Dejar el campo “name” en blanco
	
	Seleccionar el radiobutton Alias=yes
	
	En el campo ”Alias target” poner _www.tudominio.com_
	
	Routing policy=simple
	
	![Img - Record set alias](https://raw.githubusercontent.com/antonioxtasis/Domain-Ubuntu-EC2-Route53-SSL/master/imgs/record-set-alias.png)


¡Listo, probar desde una consola/terminal en tu equipo poniendo el dominio en lugar de la IP pública!


## Asegurar Servidor NGINX (Ubuntu 18.04.1 LTS) usando Let’s Encrypt


Instalar un cliente de `Let's Encrypt` en Ubuntu y emitir un certificado SSL para el dominio que se ejecuta en el servidor web NGINX.

**Paso 1 - Requisitos previos**

Antes de comenzar, asumiremos:

* Se tiene la instancia EC2 Ubuntu con privilegios `sudo` de acceso al shell.
* Un nombre de dominio registrado y señalado a la dirección IP pública de su servidor.
* El servidor web NGINX está configurado en la instancia EC2


**Paso 2 - Instalar Let´s Encrypt Client**

Descargar el `certbot-auto` (cliente de Let´s Encrypt) y guardarlo en el directorio `/usr/sbin`

```
$ sudo wget https://dl.eff.org/certbot-auto -O / usr / sbin / certbot-auto
$ sudo chmod a + x / usr / sbin / certbot-auto

```

**Paso 3 - Emita SSL para Nginx**

Let´s Encrypt realiza la Validación de Dominio (DV) automáticamente con múltiples desafíos. Una vez que la Autoridad de certificación (CA) verificó la autenticidad de su dominio, se emitirá el certificado SSL.

No es necesario crear VirtualHost para SSL/HTTPS, Let´s Encrypt lo creará. Tú sólo necesitas crear el VirtualHost para el puerto 80.

```
$ sudo certbot-auto --nginx -d example.com -d www.example.com
```

El comando anterior solicitará una dirección de correo electrónico, que se utiliza para enviar alertas de correo electrónico relacionadas con la renovación y caducidad de SSL. Además, hace algunas preguntas más. Una vez finalizado, emitirá un certificado SSL y también creará un nuevo archivo de configuración de VirtualHost en su sistema.

Al terminar saldrá este mensaje:

```
-------------------------------------------------------------------------
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2019-12-01. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again with the "certonly" option. To non-interactively renew *all*
   of your certificates, run "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
-------------------------------------------------------------------------

```

**Paso 4 - Configurar SSL Auto Renew**

Al final, configure el siguiente JOB en su servidor crontab para renovar automáticamente el certificado SSL si es necesario.

```
0 2 * * * sudo /usr/sbin/certbot-auto -q renew
```

-

### ¡Listo, ya tenemos nuestra instancia asociada a un Dominio Web y SSL funcionando!

😎



## Referencias
* [YouTube - Configure EC2 with your own domain and Route 53](https://www.youtube.com/watch?v=aHuQExY360I)
* [Tec Admin - How to Secure Nginx with Let’s Encrypt on Ubuntu 18.04 & 16.04 LTS](https://tecadmin.net/nginx-lets-encrypt-ssl-ubuntu/)

