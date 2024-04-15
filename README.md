Servicio web apache con PHP - Lambda - Python - Slack

Paso 1: Configurar una instancia EC2 en AWS
•	Inicia sesión en la consola de AWS y navega a EC2.
•	Haz clic en "Launch Instance" para lanzar una nueva instancia.
•	Selecciona una AMI de Amazon Linux 
•	Selecciona el tipo de instancia t2.micro 
•	Asegúrate de configurar adecuadamente los ajustes de seguridad ( permitir el tráfico HTTP y SSH).
•	Crea una nueva clave de par de claves o utiliza una existente para acceder a tu instancia mediante SSH.
•	Lanza la instancia.
Paso 2: Conectar a la instancia EC2 mediante SSH
•	Utiliza tu cliente SSH para conectarte a tu instancia EC2 utilizando la dirección IP pública proporcionada por AWS y la clave de par de claves en Cloud9
Paso 3: Instalar y configurar el servidor web Apache
•	Una vez conectado a la instancia EC2, ejecuta los sigueintes comandos:
o	sudo yum update -y
o	sudo yum install httpd -y
o	sudo service httpd start
o	sudo chkconfig httpd on
•	Navega al directorio del servidor web :
o	cd /var/www/html
•	Ejectuar los siguientes comandos
o	sudo yum install php  php-cli php-json  php-mbstring  –y
o	sudo  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
o	sudo  php composer-setup.php
o	sudo  php -r "unlink('composer-setup.php');"
o	sudo  php composer.phar require aws/aws-sdk-php
o	sudo service httpd restart

Paso 4: Crear un formulario HTML
•	Navega al directorio raíz del servidor web:
o	cd /var/www/html
•	Crea un archivo HTML llamado index.html con un formulario:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contact Form</title>
</head>
<body>
    <h1>Contact Form</h1>
    <form action="submit.php" method="POST">
        <label for="name">Name:</label><br>
        <input type="text" id="name" name="name" required><br>
        <label for="email">Email:</label><br>
        <input type="email" id="email" name="email" required><br>
        <label for="message">Message:</label><br>
        <textarea id="message" name="message" rows="4" required></textarea><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>




Paso 5: Crear un script PHP para procesar el formulario
•	Crea un archivo PHP llamado submit.php en el mismo directorio
o	Remplaza el variable “snsTopicArn” por el ARN de tu SNS que se va a crear en el ultimo punto de la practica.



<?php
require 'vendor/autoload.php';
 
use Aws\Sns\SnsClient;
use Aws\Exception\AwsException;
 
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $name = $_POST["name"];
    $email = $_POST["email"];
    $message = $_POST["message"];
 
    // Replace 'your-sns-topic-arn' with the ARN of your SNS topic
    $snsTopicArn = 'arn:aws:sns:us-east-1:XXXXXXX:test';
 
    // Initialize SNS client
    $snsClient = new SnsClient([
        'version' => 'latest',
        'region' => 'us-east-1' // Replace with your desired AWS region
    ]);
 
    // Create message to send to SNS topic
    $messageToSend = json_encode([
        'email' => $email,
        'name' => $name,
        'message' => $message
    ]);
 
    try {
        // Publish message to SNS topic
        $snsClient->publish([
            'TopicArn' => $snsTopicArn,
            'Message' => $messageToSend
        ]);
 
        echo "Message sent successfully.";
    } catch (AwsException $e) {
        echo "Error sending message: " . $e->getMessage();
    }
} else {
    http_response_code(405);
    echo "Method Not Allowed";
}
?>




•	Crea un archivo PHP de prueba en la ruta /var/www/html:
<?php phpinfo(); ?>



•	Accede al archivo PHP de prueba:
Abre un navegador web y navega a la dirección IP pública de tu instancia EC2 seguida de /info.php (por ejemplo, http://tu_ip_publica/info.php). Deberías ver la información de PHP en tu navegador si la configuración se realizó correctamente.
•	Verifica la configuración del servidor Apache:
Asegúrate de que el archivo de configuración de Apache (httpd.conf) incluya la directiva AddType para PHP. Puedes encontrar el archivo de configuración en /etc/httpd/conf/httpd.conf.
Añade este linea al archivo 
AddType application/x-httpd-php .php
•	Añadir Rol a la EC2
o	Vamos a la consola de la EC2, vamos a acciones -> seguridad -> Añadir Rol
o	Elegimos el Rol “LabRole”
o	Luego vamos al servicio IAM y buscamos este Rol, y añadimos la politica “AWSSNSFullAccess”

•	Crear lambda
o	Crea una lambda, elige Python 3.12, y en “permisos” elige el Rol “LabRol”
o	Añade el sigueinte codigo en la lambda 
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

o	Despliega la Lambda para confirmar su creacion.

•	Crear un topico SNS
o	Crear un topico SNS, elegir “lambda” como protocolo, y pega el ARN de la lambda.

Una vez hechos todos los pasos, accedemos a IP publica de la instancia en el navegador, y rellenamos el formulario, una vez enviado, debemos recibir un mensage en el canal de Slack creado para esta práctica con la informacion del formulario.
