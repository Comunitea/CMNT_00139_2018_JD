# Buildout base para proyectos con Odoo y PostgreSQL
Odoo 10 en el base, PostgreSQL 9.6.1 y Supervisord 3.0
- Supervisor ejecuta PostgreSQL, más info http://supervisord.org/
- También ejecuta la instancia de PostgreSQL
- Si existe  un archivo frozen.cfg es el que se debeía usar ya que contiene las revisiones aprobadas
- PostgreSQL se compila y corre bajo el usuario user (no es necesario loguearse como root), se habilita al autentificación "trust" para conexiones locales. Más info en more http://www.postgresql.org/docs/9.3/static/auth-methods.html
- Existen plantillas para los archivo de configuración de Postgres que se pueden modificar para cada proyecto.


## Utilización
En caso de no haberse hecho antes en la máquina en la que se vaya a realizar, instalar las dependencias que marca Anybox
- Añadir el repo a /etc/apt/sources.list:
```
$ deb http://apt.anybox.fr/openerp common main
```
- Si se quiere añadir la firma. Esta a veces tarda mucho tiempo o incluso da time out. Es opcional meterlo
```
$ sudo apt-key adv --keyserver hkp://subkeys.pgp.net --recv-keys 0xE38CEB07
```
- Actualizar e instalar
```
$ sudo apt-get update
$ sudo apt-get install openerp-server-system-build-deps
```
- Para poder compilar e instalar postgres
```
$ sudo apt-get install libreadline-dev
```

Clonar repositorio
> git clone https://github.com/comunitea/cmnt_bo_10.git

O el respositorio que corresponda 


- Crear un virtualenv dentro de la carpeta del respositorio. Esto podría ser opcional, obligatorio para desarrollo o servidor de pruebas, tal vez podríamos no hacerlo para un despliegue en producción. Si no está instalado, instalar el paquete de virtualenv. Es necesario tener la versión que se instala con easy_install o con pip, desinstalar el paquete python-virtualenv si fuera necesario e instalarlo con easy_install
```
$ sudo easy_install virtualenv
```

Dentro de la carpeta del repositorio
```
$ cd cmnt_bo_10
```
O la carpeta que corresponda
```
$ virtualenv sandbox --no-setuptools
```
Executar bootstrap. 
```
$ sandbox/bin/python bootstrap.py
```
Se creará la siguiente estrucura de directorios 

```bash

odoo
├── bin
├── bootstrap.py
├── buildout.cfg
├── default.cfg
├── develop-eggs
├── downloads
├── eggs
├── etc
├── nfe_xml
├── parts
├── Scenario
├── specific-parts
├── src
├── upgrade.py
├── README.md
└── var

```
- Lanzar buildout (el -c [archivo_buildout] se usa cuando no tiene el nombre por defecto buildout.cfg)
```
 $ bin/buildout -c [archivo_buildout]
 ```

- Puede que de error, si intenta crear la BD y  no está arrancado supervisor todava. Hay que lanzar el supervisor y volver a hacer bin/buildout:
```
$ bin/supervisord
$ bin/buildout -c [archivo_buildout]
```
- Conectarse al supervisor con localhost:9002 
- Si fuera necesario hacer update all, se puede parar desde el supervisor y en la consola hacer:
```
$ cd bin
$ ./upgrade_openerp
```
- odoo se lanza en el puerto 9069 (se pude configurar en otro)

## Securizar el acceso al supervisor
```
$ sudo apt-get install iptables
$ sudo iptables -A INPUT -i lo -p tcp --dport 9002 -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 9002 -j DROP
$ sudo apt-get install iptables-persistent (marcamos "yes" en las preguntas que nos hace al instalarse)
```

## Configurar Odoo
Archivo de configuración: etc/openerp.cfg, si sequieren cambiar opciones en  openerp.cfg, no se debe editar el fichero,
si no añadirlas a la sección [openerp] del buildout.cfg
y establecer esas opciones .'add_option' = value, donde 'add_option'  y ejecutar buildout otra vez.

Por ejemplo: cambiar el nivel de logging de OpenERP
```
'buildout.cfg'
...
[openerp]
options.log_handler = [':ERROR']
...
```

Si se quiere ejecutar más de una instancia de OpenERP, se deben cambiar los puertos,
please change ports:
```
openerp_xmlrpc_port = 9069  (8069 default openerp)
openerp_xmlrpcs_port = 9071 (8071 default openerp)
supervisor_port = 9002      (9001 default supervisord)
postgres_port = 5434        (5432 default postgres)
```

# Contributors

## Creators

Rastislav Kober, http://www.kybi.sk


### Para "congelar" una versión de cada módulo y dependencia

Después de jecutar con éxito el buildout, basta utilizar el siguiente comando:
```
$ bin/buildout -o odoo:freeze-to=odoo-freeze.cfg 
```
Se creará un archivo llamado `odoo-freeze.cfg` con las dependencias y revisionnes de cada mudulo. De este modo
tendremso el control de módulos y versiones usadas.

Para utilizar el `freeze` basta executar
```
$ bin/buildout -c odoo-freeze.cfg 
```
