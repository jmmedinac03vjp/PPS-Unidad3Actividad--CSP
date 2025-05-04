# PPS-Unidad3Actividad--CSP
Implementación y Evaluación de Content Security Policy (CSP) 

# 🛡️ Implementación y Evaluación de Content Security Policy (CSP)

[![Seguridad Web](https://img.shields.io/badge/Seguridad-Web-blue)](#)
[![Estado](https://img.shields.io/badge/Estado-Implementado-green)](#)
[![CSP](https://img.shields.io/badge/CSP-Activa-brightgreen)](#)

## 📌 Tema
**Protección contra XSS e inyección de scripts**

## 🎯 Objetivo
Aplicar una Content Security Policy (CSP) restrictiva y evaluar su impacto.

---

## ❓ ¿Qué es CSP?

CSP (Content Security Policy) es un mecanismo de seguridad que limita los orígenes de scripts, estilos e imágenes en una aplicación web para evitar ataques como XSS.

---

## 🔧 Implementación

### 1. Crear una página web vulnerable sin CSP

**Archivo `index.html`:**

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

**Archivo `xss.php`:**

```php
<?php 
    echo "Hola, " . $_GET['xss']; 
?>
```

**Ataque XSS de prueba:**

```html
<script>alert('XSS ejecutado!')</script>
```

Si el `alert()` se ejecuta, la página es vulnerable.

---

### 2. Implementar CSP

**Editar `000-default.conf`:**

```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'"
</IfModule>
```

**Significado:**

- `default-src 'self'`: Solo recursos del mismo origen.
- `script-src 'self'`: Solo scripts locales.

⚠️ Bloquea cualquier script externo o inline.

---

### 3. Habilitar `mod_headers` y reiniciar Apache

```bash
sudo a2enmod headers
sudo systemctl restart apache2
```

---

### 4. Comprobar si CSP bloquea XSS

Accede a:

```
http://localhost/csp/index.html
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
