# Guía de Configuración de Instancia EC2 en AWS

## Paso 1: Configurar una instancia EC2 en AWS

- Inicia sesión en la consola de AWS y navega a EC2.
- Haz clic en "Launch Instance" para lanzar una nueva instancia.
- Selecciona una AMI de Amazon Linux.
- Selecciona el tipo de instancia t2.micro.
- Asegúrate de configurar adecuadamente los ajustes de seguridad (permitir el tráfico HTTP y SSH).
- Crea una nueva clave de par de claves o utiliza una existente para acceder a tu instancia mediante SSH.
- Lanza la instancia.

## Paso 2: Conectar a la instancia EC2 mediante SSH

- Utiliza tu cliente SSH para conectarte a tu instancia EC2 utilizando la dirección IP pública proporcionada por AWS y la clave de par de claves en Cloud9.

## Paso 3: Instalar y configurar el servidor web Apache

- Una vez conectado a la instancia EC2, ejecuta los siguientes comandos:
```php
sudo yum update -y
sudo yum install httpd -y
sudo service httpd start
sudo chkconfig httpd on
```

- Navega al directorio del servidor web:
cd /var/www/html

- Ejecuta los siguientes comandos:
```php
sudo yum install php php-cli php-json php-mbstring -y
sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
sudo php composer-setup.php
sudo php -r "unlink('composer-setup.php');"
sudo php composer.phar require aws/aws-sdk-php
sudo service httpd restart
```

## Paso 4: Crear un formulario HTML

- Navega al directorio raíz del servidor web:
cd /var/www/html

- Crea un archivo HTML llamado `index.html` con un formulario (ver contenido en el archivo original).

## Paso 5: Crear un script PHP para procesar el formulario

- Crea un archivo PHP llamado `submit.php` en el mismo directorio (ver contenido en el archivo original).

- Crea un archivo PHP de prueba en la ruta `/var/www/html`:
```php
<?php phpinfo(); ?>
```
Accede al archivo PHP de prueba abriendo un navegador web y navegando a la dirección IP pública de tu instancia EC2 seguida de /info.php.

Verificar la configuración del servidor Apache
Asegúrate de que el archivo de configuración de Apache (httpd.conf) incluya la directiva AddType para PHP. Puedes encontrar el archivo de configuración en /etc/httpd/conf/httpd.conf. Añade la siguiente línea al archivo:
AddType application/x-httpd-php .php
Añadir Rol a la EC2
Ve a la consola de EC2, acciones -> seguridad -> Añadir Rol.
Elige el Rol “LabRole”.
Luego ve al servicio IAM y busca este Rol, y añade la política “AWSSNSFullAccess”.
Crear una lambda
Crea una lambda, elige Python 3.12, y en “permisos” elige el Rol “LabRol”.
Añade el siguiente código en la lambda:
o	Crea una lambda, elige Python 3.12, y en “permisos” elige el Rol “LabRol”
o	Añade el sigueinte codigo en la lambda:
```php
import urllib3
import json
http = urllib3.PoolManager()
def lambda_handler(event, context):
    url = "https://hooks.slack.com/services/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 "
    msg = {
        "channel": "#devops",
        "username": "abdessamad.ammi",
        "text": event['Records'][0]['Sns']['Message'],
        "icon_emoji": ""
    }
    
    encoded_msg = json.dumps(msg).encode('utf-8')
    resp = http.request('POST',url, body=encoded_msg)
    print({
        "message": event['Records'][0]['Sns']['Message'], 
        "status_code": resp.status, 
        "response": resp.data
    })
```
Y	Despliega la Lambda para confirmar su creacion.

-Crear un topico SNS
Crea un topico SNS, elige “lambda” como protocolo, y pega el ARN de la lambda.
Una vez hechos todos los pasos, accede a la IP pública de la instancia en el navegador, y rellena el formulario. Una vez enviado, deberías recibir un mensaje en el canal de Slack creado para esta práctica con la información del formulario.

