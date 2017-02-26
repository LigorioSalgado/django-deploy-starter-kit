# Django-Deploy-Starter-Kit

Pasos basicos para hacer deploy de una App en Django con EC2 y AWS

**Requisitos**

 * Cuenta en AWS ya activada 
 * Terminal con ssh (Linux,MacOs) y [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) (Windows) 	[Video](https://www.youtube.com/watch?v=0PtBREh74r4) para mas info con putty

## Tabla de Contenido** <br>
 
  - [Preparando todo](#preparando-todo) 
- [Creando Instancia](#creando-instancia) 
- [Conectando y Configurando](#conectando-y-configurando) 
	- [Configuración inical de nuestra instancia](#configuración-inical-de-nuestra-instancia)
	- [Instalando Postgres,Nginx,vitualenv](#instalando-postgres,nginx,vitualenv)
	- [Configuracion http](#configuracion-http)
- [Deploy](#deploy)
	- [Configurando Django](#configurando-django)
	- [Creando base de datos](#creando-base-de-datos)
	- [Probando django](#probando-django)
	- [Gunicorn y Nginx](#gunicorn-y-nginx)
 
 
## Preparando todo

* Instalar [python-dotenv](https://github.com/theskumar/python-dotenv) (Libreria para crear variables de entorno) 
```sh 
$ pip install -U python-dotenv
```
* Agregamos la configuracion de <b>python-dotenv</b> en el [manage.py](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/manage.py) y en el [wsgi.py](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/wsgi.py)
```python
#WSGI.py
from dotenv import load_dotenv
	try:
    dotenv_path = os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), '.env')
    load_dotenv(dotenv_path)
except:
    pass
    
#manage.py

from dotenv import load_dotenv

try:

    dotenv_path = os.path.join(os.path.dirname(__file__), '.env')
    load_dotenv(dotenv_path)
except:
    pass


```
  	
* Dentro del Proyecto al mismo nivel del manage.py crear documento requirements de dependencias de python
```sh
$ pip freeze > requeriments.txt
```

* Configurar [settings.py](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/settings.py)  y [local_settings.py](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/local_settings.py) (click para ver ejemplo)
* Crear .gitignore con  [gitignore.io](http://gitignore.io/)
* Subimos nuestro proyecto al repositorio remoto que tenemos en github

## Creando Instancia

* En el dashboard de AWS seleccionar la opcion EC2 en la pestaña de de "services"
![Alt text](/media/cap1.png?raw=true "Optional Title")
* Ubicados "EC2 Dashboard" seleciona "Launch Instance"
![Alt text](/media/cap2.png?raw=true "Optional Title")
* Selecciona la distribucion del servidor <b>NOTA:</b> Para este ejemplo utilizaremos "Ubuntu server 16.04 LTS"
* Seleccionamos la opcion "t2.micro" y damos click en "Review and Launch"
![Alt text](/media/cap3.png?raw=true "Optional Title")
* En la ventana de "Review Instance Launch" dar click en "Launch"
![Alt text](/media/cap4.png?raw=true "Optional Title")
* Aparece un dialogo para seleccionar o crear  "Key pair", escogemos la opcion de "Create a new key pair" y agregamos un nombre, por ultimo damos click  en "Download Key Pair" <b>NOTA:</b> No perder el  archivo "nombre.pem" ya que es la unica forma de entrar al servidor
![Alt text](/media/cap5.png?raw=true "Optional Title")
* Por ultimo damos click en "launch instance"
* En la pantalla de "Launch Status" selecionamos la opcion  de "View instance"
![Alt text](/media/cap6.png?raw=true "Optional Title")
* Finalmente seremos enviados de nuevo al dashboard y se vera corriendo nuestra nueva instancia
![Alt text](/media/cap7.png?raw=true "Optional Title")

## Conectando y configurando
* En el dashboard de la intancia selecionamos la opcion de "Connect" aparacera un dialogo con algunas instrucciones:
![Alt text](/media/cap8.png?raw=true "Optional Title")
	* Mac y Linux:
		* Copiar el "archivo.pem" en la carpeta ".ssh" que se encuentra en su directorio raiz
		* cambiar los permisos del archivo con
		```sh
        $ chmod 400 archivo.pem
        ```
        *Para conectarse utlizar el ejemplo que aparece en el dialogo de "connect" por ejemplo:
        ```sh
        $ ssh -i "prueba.pem" ubuntu@ec2-35-160-39-170.us-west-2.compute.amazonaws.com
        ```
        ![Alt text](/media/cap9.png?raw=true "Optional Title")
    * Windows:
    	* Seguir el siguiente tutorial para usar Putty -> https://www.youtube.com/watch?v=0PtBREh74r4

* Al entrar no pide crear un fingerprint escribimos "yes"
![Alt text](/media/cap10.png?raw=true "Optional Title")
* Sabremos si estamos conectados si aparece una pantalla como sigue:
* ![Alt text](/media/cap11.png?raw=true "Optional Title")

### Configuración inical de nuestra instancia

* En la consola escribimos los siguentes comandos para catualizar los repositorio y dependencias de ubuntu:
```sh
$ sudo apt-get update
$ sudo apt-get upgrade
```
* Tendremos que configurara los "locale"  atravez de los siguientes pasos:
	* ejecutamos el siguiente comando
	```sh
    $ sudo nano /etc/environment
    ```
    * agregamos la siguientes lines abajo de PATH
    ```vim
    LC_ALL="en_US.utf8"
	LANGUAGE="en_US.utf8"
    ```
    * Guardamos el archivo y reiniciamos la instancia con:
    ```sh
    sudo reboot
    ```
    * La conexion se perdera y esperamos unos minutos para reestablecer la conexion

### Instalando Postgres,Nginx,vitualenv
* Para instalar Postgres 9.5 en nuestra instancia corremos los siguientes comandos:
```sh
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
$ wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add 
$ sudo apt-get update
$ sudo apt-get install postgresql postgresql-contrib
```
* Para instalar nginx se tiene que realizar los siguiente:

```sh
$ sudo apt-get install nginx
```
* instalaremos PIP y  Virtualenv es:
```sh
$ sudo apt-get install python-virtualenv python-pip
```
### Configuracion http

* Regresamos al  dashboard  de AWS y seleccionamos "Security groups" que se encuentra en el menu izquierdo en la parte inferior 
![Alt text](/media/cap12.png?raw=true "Optional Title")
* Seleccionamos la opcion que diga "launch-wizard-XXXX" en la parte de "Group name"
* Ya con la opcion señalada ,damos clic en "Actions">"Edit inbound rules"
![Alt text](/media/cap13.png?raw=true "Optional Title")
* Aparecera un dialogo como el siguiente, damos click en "Add rule"
![Alt text](/media/cap14.png?raw=true "Optional Title")
* Se crea  una nueva opción llamada "Custom TCP Rule", seleccionamos la opcion y elegimos "HTTP", damos en "Save"
![Alt text](/media/cap15.png?raw=true "Optional Title")
* Por ultimo volvemos "Instances" (la opcion se encuentra en el menu izquierdo en la parte superior), copiamos  el "Public DNS (IPv4)" que se encuentra en la parte inferir de nuestra pantalla  y lo pegamos en una nueva pestaña del navegador, como resultado tendremos el "Welcome to Nginx" en el navegador
![Alt text](/media/cap16.png?raw=true "Optional Title")
![Alt text](/media/cap17.png?raw=true "Optional Title")

## Deploy
### Configurando Django
* Iniciamos nuestra consola de ssh 
* Una vez dentro creamos una carpeta Ex : "Proyecto"
```sh
mkdir proyecto
```
* Entramos a la nueva carpeta  y creamos un entorno virtual con python 3
```sh
$ cd proyecto
$ virtualenv entorno -p python3
````
* Clonamos nuestro repositorio remoto
```sh
git clone https://github.com/LigorioSalgado/Ejemplo
```
* <b> Nota:</b> Entes de empezar necesitamos intsalar lo siguiente:
```sh
$ sudo apt-get install libpq-dev python3-dev build-essential 
````
* Activamos nuestro entorno y Entramos en la carpeta de nuestro proyecto he instalamos las dependencias con :
```sh
$ pip install -r requirements.txt
```
### Creando base de datos

* Para ingresar a la linea de comandos de postgres  se debe ejecutar  lo siguiente:
```sh
$ sudo su - postgres
````
* Creamos la BD para nuestro proyecto
```sh
$ createdb (nombre de la BD)
$ createuser (nombre del usuario de la BD)
```
*Despues tendremos que ingresar a psql 
```sh
$ psql
````
*una vez en psql damos los siguientes comandos
```psql
postgres=# ALTER DATABASE (Nombre de la BD) OWNER TO (Nombre del usuario de la BD);
postgres=# ALTER USER  (nombre del usuario BD) WITH PASSWORD (password para el usuario)
postgres=# \l #lista las bd
postgres=# \q #sale de psql
````
* damos el siguiente comando para regresar a nuestro proyecto
```sh
$ exit
````
### Probando django
* Antes de empezar debemos crear un archivo llamado <b>.env</b> al mismo nivel que el <b>manage.py</b> con lo siguiente
<br><b>Nota:el archivo .env tambien se agrega al gitignore</b>
````
DBNAME = "nombre de la BD"
DBUSER = "nombre del usuario de la BD"
DBPASSWORD = "password de la BD"
# Tambien se pueden poner otros valores como api keys o secret keys
`````
* Hacemos la migraciones de nuestro proyecto, recogemos los staticos del proyecto y por ultimo creamos un superusuario
```sh
$ python manage.py migrate
$ python manage.py collectstatic
$ python manage.py createsuperuser
```
*Por ultimo corremos <b>check</b>  para comprobar errores como sigue:
```sh
$ python manage.py check
```
### Gunicorn y Nginx
* Se crea un archivo llamado [gunicorn.service](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/gunicorn.service) (ver ejemplo) con el comando 
```sh
sudo nano /etc/systemd/system/gunicorn.service
````
* Despues se ejecutan los siguientes comandos para activar el servicio
```sh
$ sudo systemctl start gunicorn
$ sudo systemctl enable gunicorn
$ sudo systemctl status gunicorn
````
* Una vez activado gunicorn procedemos a configurar nginx como lo muestra este [archivo](https://github.com/LigorioSalgado/django-deploy-starter-kit/blob/master/api) con el siguiente comando
```sh
$ sudo nano /etc/nginx/sites-available/<nombre-del-proyecto>
```
* creamos un link simbolico :
```sh
$ sudo ln -s /etc/nginx/sites-available/<nombre-del-proyecto> /etc/nginx/sites-enabled
```
* corremos un test para comprobar que no hay errores de sintaxis:
```sh
$ sudo nginx -t
````
* Reiniciamos nginx con:
```sh
$ sudo systemctl restart nginx
```
*Por ultimo  nos dirigimos a <b>la-public-dns-de-la-instancia.amazonaws.com/admin/ y comprobamos que el server ya esta corriendo y ya podemos usar el admin</b>
![Alt text](/media/cap18.png?raw=true "Optional Title")

