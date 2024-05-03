# DB Joint Purchase
## Instalación de entorno Prestashop para las pruebas

 - Crear carpeta de proyecto
 - Crear archivo 'docker-compose.yml' dentro de la carpeta del proyecto:
 - Copiar el siguiente contenido en el archivo 'docker-compose.yml':

   ```
   services:
     mysql:
         image: mysql:5.7
         container_name: prestashop-mysql
         tty: true
         ports:
             - "3308:3306"
         volumes:
             - "./var/lib/mysql/:/var/lib/mysql"
         environment:
             MYSQL_ROOT_PASSWORD: root
         networks:
             - cx-prestashop1.7-net
     phpmyadmin:
         image: phpmyadmin/phpmyadmin
         container_name: PhpMyAdmin
         environment:
             PMA_HOST: prestashop-mysql
             PMA_PORT: 3308
             PMA_ARBITRARY: 1
         restart: always
         ports:
             - "8082:80"
         networks:
             - cx-prestashop1.7-net
     server:
         image: prestashop/prestashop:1.7
         container_name: PS-apache
         ports:
             - "8083:80"
         volumes:
             - "./:/var/www/html"
         environment:
             DB_SERVER: prestashop-mysql
         depends_on:
             - mysql
         networks:
             - cx-prestashop1.7-net
    networks:
        cx-prestashop1.7-net:
            driver: bridge

- Establecer contraseña para mysql ( MYSQL_ROOT_PASSWORD: XXXXXX )
- Ejecutar el comando 'docker compose up -d' para levantar el servidor (tardará la primera vez)
- Realizar la instalación de prestashop, accediendo a 'localhost:8080'. En la configuración de la base de datos, el servidor es el nombre del contenedor de mysql 'PS-MySql'.
- Clonar este repositorio en la carpeta modules, dentro de prestashop, con el nombre 'dbjointpurchase'
- En el back office de prestashop, instalar este módulo.

## Descripción del módulo ##
Este módulo permite al administrador de la tienda seleccionar los productos que se mostrarán en el pack de productos que se ofrece en el front de cada producto.
   ## Adiciones ##
   -  ### scr/PurchaseJointHandler.php ###
        -  Clase que se encarga de gestionar la lógica de los productos que se mostrarán en el pack de productos.
        - Limitar el munero de productos que se pueden seleccionar en el backoffice hasta 3.
        - Dado un producto, obtiene los productos adheridos manualmente por el usuario.
        - Establece el estado de un joint del producto y lo crea/elimina/modifica en BD, si procede. Si el producto se queda sin joints, borra el registro en BD
        - Inserta un nuevo joint al producto (Si no hay hueco, retorna false).
        - Extrae un joint del producto (Si se queda vacío, eliminamos el producto en la tabla de joints).
   - ### controllers/front/save ###
       - Controlador encargado de guardar la seleccion del cliente
   -  ### sql/* ###
      - Creamos/Borramos la nueva tabla en BD que se encargara de guardar la configuración del cliente. 
   - views/templates/admin/configure.tpl


## Funcionamiento general del módulo ##
En el backoffice de los productos, aparecen una serie de productos alternativos seleccionables (productos más vendidos junto al producto principal + top ventas). 

Si no se selecciona ninguno (y máximo se pueden seleccionar 3), el módulo sigue su lógica habitual.

La lógica habitual del módulo es ofrecer en el front de cada producto, un pack de productos para comprar en un solo click). Este pack es el top 3 de productos más comprados junto al producto original, y si no hubiera, ofrece el top 3 de ventas generales.

Lo que hace nuevo el módulo en el back office es ofrecer todas las posibilidades (top ventas conjuntas + top ventas generales) para que el administrador pueda decidir qué productos ofrecer al cliente en dicho pack.