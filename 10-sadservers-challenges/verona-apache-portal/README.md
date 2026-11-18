# SadServers – Verona: Apache Portal Won't Open

**Level:** Medium  
**Type:** Fix  
**Tags:** Apache · Realistic Interviews  
**Environment:** Debian Linux  
**Web server:** Apache2  
**DocumentRoot:** `/var/www/portal`  
**Expected result:** Portal homepage and legacy `/reports` redirect working  
**Estimated time:** 15 minutes

---

## 1. Descripción del problema

Un portal interno basado en Apache2 había sido migrado a un nuevo servidor Debian.

El directorio raíz del sitio web estaba configurado en:

```bash
/var/www/portal
```

Sin embargo, se presentaban dos problemas:

1. La página principal (`http://localhost/`) no mostraba el contenido esperado.
2. Una antigua dirección (`http://localhost/reports`) ya no redirigía a la página de estado (`/status/`).

El acceso directo a `/status/` funcionaba correctamente.

El objetivo era recuperar el funcionamiento del portal y conseguir que las siguientes peticiones devolvieran el contenido esperado:

**Página principal:**

```text
SadServers - Portal Ready
```

**Página de informes:**

```text
SadServers - Status OK
```

---

## 2. Diagnóstico inicial

### 2.1. Comprobación de Apache

Se comprobó el estado del servicio:

```bash
systemctl status apache2
```

También se puede verificar la configuración:

```bash
sudo apache2ctl -t
```

Resultado:

```text
Syntax OK
```

Aparecía un aviso `AH00558` indicando que Apache no podía determinar automáticamente el nombre FQDN del servidor.

Este aviso no impedía el funcionamiento del servicio, ya que la sintaxis de la configuración era correcta.

### 2.2. Comprobación de los puertos

```bash
ss -lntp
```

Se verificó que Apache estaba escuchando en el puerto TCP 80.

### 2.3. Comprobación de la página principal

Se ejecutó:

```bash
curl -s http://localhost/
```

La respuesta no contenía el texto esperado `SadServers - Portal Ready`.

### 2.4. Comprobación de la ruta /reports

Se ejecutó:

```bash
curl -L http://localhost/reports
```

Resultado:

```html
<title>404 Not Found</title>
```

Apache devolvía un error HTTP 404, indicando que la ruta solicitada no estaba disponible.

### 2.5. Inspección del directorio del portal

Se comprobó el contenido del DocumentRoot:

```bash
sudo ls -la /var/www/portal/
```

Resultado:

```text
total 16
drwxr-xr-x 3 www-data www-data 4096 .
drwxr-xr-x 4 root     root     4096 ..
-rw-r--r-- 1 www-data www-data  135 .htaccess
drwxr-xr-x 2 www-data www-data 4096 status
```

Se observaron dos elementos importantes:

- Existía un fichero `.htaccess`.
- Existía un directorio `status/`, pero no había ningún fichero `index.html` en el directorio raíz.

---

## 3. Investigación

### 3.1. Inspección del fichero .htaccess

Se consultó el contenido del fichero:

```bash
sudo cat /var/www/portal/.htaccess
```

Contenido:

```apache
# Legacy bookmark: /reports should reach the status page (applied only when AllowOverride permits)

RedirectMatch ^/reports/?$ /status/
```

La directiva `RedirectMatch` de Apache estaba configurada para redirigir las peticiones a `/reports` hacia `/status/`.

Sin embargo, la redirección no se estaba aplicando.

### 3.2. Comprobación de AllowOverride

Se inspeccionó la configuración del VirtualHost:

```bash
sudo grep -C 5 "AllowOverride" /etc/apache2/sites-available/portal.conf
```

Resultado:

```apache
ServerName localhost
DocumentRoot /var/www/portal

<Directory /var/www/portal>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Se encontró la causa del primer problema de configuración:

```apache
AllowOverride None
```

Esta directiva impedía que Apache aplicase las directivas permitidas mediante `.htaccess`.

Por tanto, la redirección `RedirectMatch` estaba definida correctamente, pero Apache ignoraba su configuración.

### 3.3. Comprobación de los logs

Se consultó el log específico del VirtualHost:

```bash
sudo tail -n 30 /var/log/apache2/portal-access.log
```

Y el log de errores:

```bash
sudo tail -n 30 /var/log/apache2/portal-error.log
```

En el log de errores se identificó un problema relacionado con la ausencia de un índice de directorio.

Esto apuntaba al segundo problema: el DocumentRoot no tenía un fichero de inicio que Apache pudiera servir.

Además, la configuración:

```apache
Options -Indexes +FollowSymLinks
```

deshabilitaba el listado automático del contenido del directorio.

---

## 4. Causa raíz

Se identificaron dos problemas independientes.

### Problema 1. Apache ignoraba las directivas del fichero .htaccess

El fichero `/var/www/portal/.htaccess` contenía:

```apache
RedirectMatch ^/reports/?$ /status/
```

Sin embargo, el VirtualHost tenía configurado:

```apache
AllowOverride None
```

Esto impedía que Apache aplicase las directivas del fichero `.htaccess`.

Como consecuencia, la ruta `/reports` no ejecutaba la redirección y devolvía un error 404.

### Problema 2. Ausencia de un documento índice

El directorio:

```bash
/var/www/portal/
```

no contenía ningún fichero `index.html`.

Apache no tenía un documento índice que servir al solicitar la raíz del portal.

Además, al estar deshabilitado el listado de directorios mediante `Options -Indexes`, no se generaba un listado automático del contenido.

Esto provocaba que la página principal no devolviera el contenido esperado.

---

## 5. Solución aplicada

### 5.1. Habilitar las directivas de .htaccess

Se modificó el VirtualHost:

```bash
sudo vi /etc/apache2/sites-available/portal.conf
```

Se localizó la directiva:

```apache
AllowOverride None
```

Y se habilitó la aplicación de las directivas necesarias para `.htaccess`.

Una configuración con permisos mínimos suficientes para la directiva `RedirectMatch` sería:

```apache
AllowOverride FileInfo
```

`FileInfo` permite utilizar directivas de configuración relacionadas con la gestión de documentos, entre ellas las redirecciones de `mod_alias`.

También se podría utilizar `AllowOverride All`, aunque es más permisivo de lo necesario para este caso.

**Nota:** el cambio realizado durante la práctica fue habilitar AllowOverride. `FileInfo` es la alternativa de mínimo privilegio recomendada para esta redirección.

### 5.2. Comprobar la sintaxis y recargar Apache

Después de modificar el VirtualHost:

```bash
sudo apache2ctl -t
```

Resultado:

```text
Syntax OK
```

Se recargó el servicio para aplicar los cambios:

```bash
sudo systemctl reload apache2
```

### 5.3. Comprobar la redirección

Se ejecutó:

```bash
curl -s -m 2 -L http://127.0.0.1/reports | head -n 1
```

Resultado:

```text
SadServers - Status OK
```

La opción `-L` permite seguir automáticamente las redirecciones HTTP.

También se comprobó la respuesta sin seguir la redirección:

```bash
curl http://127.0.0.1/reports
```

Apache devolvía:

```text
HTTP/1.1 302 Found
```

Y redirigía hacia:

```text
http://127.0.0.1/status/
```

Esto confirmó que la configuración de `.htaccess` ya estaba funcionando.

### 5.4. Crear el documento índice

Una vez corregida la redirección, se investigó el problema de la página principal.

Se creó el fichero:

```bash
sudo vi /var/www/portal/index.html
```

Con el siguiente contenido:

```html
SadServers - Portal Ready
```

El fichero `index.html` proporcionaba el documento de inicio que Apache necesitaba para servir la página principal.

---

## 6. Validación

El escenario incluía un script de comprobación:

```bash
sudo cat /home/admin/agent/check.sh
```

El script verificaba dos condiciones.

### 6.1. Validación de la página principal

```bash
curl -s -m 2 http://127.0.0.1/ | head -n 1
```

Resultado esperado:

```text
SadServers - Portal Ready
```

### 6.2. Validación de la redirección

```bash
curl -s -m 2 -L http://127.0.0.1/reports | head -n 1
```

Resultado esperado:

```text
SadServers - Status OK
```

### 6.3. Comprobación final

Se ejecutó el botón **Check My Solution** de SadServers.

Resultado:

```text
OK
```

<badge>Escenario resuelto</badge>

El portal quedó operativo y ambas comprobaciones devolvieron el contenido esperado.

---

## 7. Aprendizajes

Esta práctica permitió profundizar en diferentes aspectos de administración de Apache y Linux.

### Apache y VirtualHosts

- Configuración de VirtualHosts.
- Funcionamiento de `DocumentRoot`.
- Directivas `<Directory>`.
- Diferencia entre `AllowOverride None`, `FileInfo` y `All`.
- Uso de ficheros `.htaccess`.
- Recarga del servicio mediante `systemctl reload`.

### Redirecciones HTTP

- Funcionamiento de `RedirectMatch`.
- Uso del módulo `mod_alias`.
- Diferencia entre respuestas HTTP 302 y 404.
- Uso de `curl -L` para seguir redirecciones.

### Sistema de archivos

- Estructura de directorios de un sitio web.
- Funcionamiento de los documentos índice (`index.html`).
- Diferencia entre ausencia de un documento índice y ausencia de permisos.
- Funcionamiento de `Options -Indexes`.

### Diagnóstico y logs

- Interpretación de logs específicos de un VirtualHost.
- Identificación de errores relacionados con la ausencia de DirectoryIndex.
- Inspección de la configuración efectiva del servidor.
- Uso de scripts de comprobación para validar requisitos.

### Metodología de troubleshooting

La práctica permitió trabajar un procedimiento de diagnóstico por etapas:

1. Comprobar el estado del servicio Apache.
2. Verificar los puertos en escucha.
3. Comprobar las respuestas HTTP de las diferentes rutas.
4. Inspeccionar el contenido del DocumentRoot.
5. Revisar la configuración del VirtualHost.
6. Comprobar las directivas de `.htaccess`.
7. Consultar los logs de Apache.
8. Aplicar las correcciones necesarias.
9. Validar cada cambio mediante peticiones HTTP.
10. Ejecutar el test final del escenario.

---

## 8. Consideraciones para un entorno de producción

Aunque el escenario quedó resuelto, conviene tener en cuenta algunas consideraciones:

- Evitar `AllowOverride All` cuando solo se necesitan directivas específicas. Es preferible habilitar únicamente las clases necesarias.
- Siempre que sea posible, definir las redirecciones directamente en la configuración del VirtualHost, en lugar de depender de `.htaccess`.
- Mantener deshabilitado el listado de directorios cuando no sea necesario.
- Comprobar que los DocumentRoot disponen de un documento índice adecuado.
- Revisar los logs específicos de cada VirtualHost para facilitar el diagnóstico.
- Utilizar `apache2ctl -t` antes de recargar o reiniciar Apache.
- Preferir `systemctl reload apache2` cuando solo se necesite aplicar una modificación de configuración y la recarga sea compatible.

---

## 9. Referencias

- [SadServers – Verona: Apache Portal Won't Open](https://sadservers.com/scenario/verona)
- [Apache – Configuration Files (.htaccess)](https://httpd.apache.org/docs/2.4/howto/htaccess.html)
- [Apache – AllowOverride Directive](https://httpd.apache.org/docs/2.4/mod/core.html#allowoverride)
- [Apache – mod_alias / RedirectMatch](https://httpd.apache.org/docs/2.4/mod/mod_alias.html#redirectmatch)
- [Apache – DirectoryIndex](https://httpd.apache.org/docs/2.4/mod/mod_dir.html#directoryindex)
