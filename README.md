# PPS-Unidad3Actividad--CSP
Implementación y Evaluación de Content Security Policy (CSP) 

# 🛡️ Implementación y Evaluación de Content Security Policy (CSP)

[![Seguridad Web](https://img.shields.io/badge/Seguridad-Web-blue)](#)
[![Estado](https://img.shields.io/badge/Estado-Implementado-green)](#)
[![CSP](https://img.shields.io/badge/CSP-Activa-brightgreen)](#)

## 📌 Tema
**Aplicar CSP nos permite protección contra XSS e inyección de scripts**

## 🎯 Objetivo
Aplicar una Content Security Policy (CSP) restrictiva y evaluar su impacto.

---

## ❓ ¿Qué es CSP?

**CSP (Content Security Policy)** es un mecanismo de seguridad que limita los orígenes de scripts, estilos e imágenes en una aplicación web para evitar ataques como **XSS**.

---
## Iniciar entorno de pruebas

-Situáte en la carpeta de del entorno de pruebas de nuestro servidor LAMP e inicia el escenario docker-compose

~~~
docker-compose up -d
~~~

Cómo estamos utilizando un escenario docker-compose, para acceder a nuestra máquina tendremos que ejecutar:

~~~
docker exec -it lamp-php83 /bin/bash
~~~

Nuestro contenedor que contiene el servicio web, como ves se llama lamp-php83. Si la carpeta donde tienes el escenario tiene otro nombre diferente de lamp, tendrás que cambiar el nombre.


## 🔧 Implementación


### 1. Crear una página web vulnerable sin CSP

Para probar, vamos a crear una página vulnerable sin CSP.

Creamos en nuestro servidor un host virtual con nombre `csp.edu`. Lo primero creamos la carpeta y el resto de archivos:

~~~
mkdir /var/www/html/CSP
~~~

Archivo `/etc/apache2/sites-available/csp.conf`:
~~~
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        ServerName csp.pps.edu

        DocumentRoot /var/www/html/CSP

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

~~~

Habilitamos el host virtual y reiniciamos el servicio

~~~
a2ensite csp.conf
service apache2 reload
~~~

Comprobamos también que hemos añadido en `/etc/hosts` el nombre del host virtual:

```
127.0.0.1       csp.pps.edu      www.csp.pps.edu
```

Y creamos los archivos para probar.

El archivo index.html nos mostrará una caja de texto en la que podremos introducir nuestro nombre para que nos salude.

Archivo `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Prueba sin CSP</title>
</head>
<body>
    <h2>Introduce tu nombre:</h2>
    <form action="xss.php" method="GET">
        <input type="text" name="xss">
        <button type="submit">Saludar</button>
    </form>
</body>
</html>
```

Creamos un ficher xss.php que es llamado desde `index.html` con el nombre introducido. Con él vamos a probar si la web es vulnerable a ataques **XSS**.


Archivo `xss.php`:

```php
<?php 
    echo "Hola, " . $_GET['xss']; 
?>
```

Y, si no está, cambiamos el propietario de todo el directorio a www-data:

```
chown -R  www-data:www-data CSP/
```

Al acceder al servidor `http://csp.pps.edu` se nos mostrará algo así:

![](images/csp1.png)


**Ataque XSS de prueba:**

Para probar si es vulnerable a ataques  **XSS** introducimos en el campo de saludo:

```html
<script>alert('XSS ejecutado!')</script>
```

![](images/csp2.png)

Si el `alert()` se ejecuta, la página es vulnerable.

![](images/csp3.png)

---

### 2. Implementar CSP

Editamos el archivo de configuración del sitio virtual`/etc/apache2/sites-availabe/csp.conf`:

```apache
<VirtualHost *:80>

        ServerName csp.pps.edu
        ServerAdmin webmaster@localhost

        DocumentRoot /var/www/html/CSP

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <IfModule mod_headers.c>
            Header always set Content-Security-Policy "default-src 'self'; scri>
        </IfModule>
</VirtualHost>
```


**Significado:**

- `default-src 'self'`: Solo recursos del mismo origen.
- `script-src 'self'`: Solo scripts locales.

⚠️ Bloquea cualquier script externo o inline.

---

### 3. Habilitar `mod_headers` y reiniciar Apache

Habilitamos el módulo **headers** y recargamos el servicio **apache2** para que surtan efectos los cambios

```bash
a2enmod headers
service apache2 reload
```

---

### 4. Comprobar si CSP bloquea XSS

Accede a:

```
http://csp.pps.edu/
```

Intenta:

```html
<script>alert('XSS ejecutado!')</script>
```

**La consola debe mostrar un error como:**

```
Refused to execute script because 'script-src' directive disallows it.
```

---

## ✅ Mitigación y Buenas Prácticas

- Verifica CSP con:
  
```bash
curl -I http://localhost/csp/index.html | grep Content-Security-Policy
```

- Dónde aplicar CSP:
  - `apache2.conf`: aplica a todo el servidor.
  - `.htaccess`: solo a una carpeta.
  - `000-default.conf`: al VirtualHost principal.

---

## 🔐 CSP Más Estricto

**Política avanzada:**

```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'nonce-random123'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'"
</IfModule>
```

**¿Qué hace esta política?**

- Bloquea `eval()` y scripts inline sin `nonce`.
- Impide iframes, objetos, y el uso de `<base>` externo.

**Ejemplo de script inline permitido con `nonce`:**

```html
<script nonce="random123">
    console.log("Este script sí se ejecutará porque tiene el nonce correcto.");
</script>
```

---

## 🔒 CSP Máxima Seguridad (Recomendada para Producción)

```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'nonce-ABC123'; style-src 'self' 'nonce-ABC123'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests"
</IfModule>
```

**Mejoras:**

- Evita inline scripts y estilos.
- Bloquea contenido HTTP en sitios HTTPS.
- Mayor defensa contra inyecciones.

---

## 🧪 Pruebas de BYPASS

1. Inline Script:
```html
<script>alert('XSS ejecutado!')</script>
```

2. Uso de `eval()`:
```html
<script>eval("alert('XSS ejecutado!')")</script>
```

3. Inyección de `iframe`:
```html
<iframe src="http://attacker.com"></iframe>
```

4. Evento malicioso en imagen:
```html
<img src="#" onerror="alert('XSS ejecutado!')">
```

✅ Si todos son bloqueados, ¡la política CSP está funcionando!

---
