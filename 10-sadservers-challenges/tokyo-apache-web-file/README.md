# SadServers – Tokyo: Can't serve web file

**Level:** Medium  
**Type:** Fix  
**Tags:** Apache · Realistic Interviews  
**Environment:** Debian Linux  
**Web server:** Apache2  
**Expected result:** `curl 127.0.0.1:80` → `hello sadserver`  
**Estimated time:** 15 minutes

---

## 1. Descripción del problema

El servidor dispone de Apache2 instalado y ejecutándose. Existe un fichero web en:

```bash
/var/www/html/index.html
```

Su contenido es:

```text
hello sadserver
```

Sin embargo, al realizar una petición HTTP local:

```bash
curl 127.0.0.1:80
```

No se obtiene ninguna respuesta y la conexión termina en timeout.

**Objetivo:** diagnosticar y corregir el problema para conseguir que Apache sirva correctamente el fichero `index.html` y que la petición devuelva:

```text
hello sadserver
```

El escenario no requiere conocimientos específicos de configuración de Apache, sino conocimientos generales de administración Linux, networking, firewall y permisos del sistema de archivos.

---

## 2. Diagnóstico inicial

### 2.1. Comprobación de la configuración de Apache

Se ejecutó:

```bash
apache2 -t
```

Se obtuvo un error relacionado con la directiva `DefaultRuntimeDir`, indicando que la ruta no era válida.

Se comprobó la configuración:

```apache
DefaultRuntimeDir ${APACHE_RUN_DIR}
```

La variable se encontraba definida en `/etc/apache2/envvars`.

Al cargar manualmente las variables:

```bash
. /etc/apache2/envvars
echo $APACHE_RUN_DIR
```

El resultado fue:

```text
/var/run/apache2
```

El error inicial estaba relacionado con ejecutar `apache2 -t` directamente sin cargar previamente las variables de entorno de Debian. No era la causa del problema de conectividad.

### 2.2. Comprobación del servicio

```bash
systemctl status apache2
```

Resultado: Apache2 estaba activo y funcionando.

Se comprobó que:
- El proceso principal se ejecutaba como `root`.
- Los procesos hijo se ejecutaban como `www-data`.

Este comportamiento es el habitual en Debian.

### 2.3. Comprobación de los puertos

```bash
ss -lntp
```

Se comprobó que Apache estaba escuchando en:

```text
*:80
```

El puerto 443 no estaba escuchando, pero no era relevante para este escenario, ya que las pruebas se realizaban mediante HTTP en el puerto 80.

### 2.4. Comprobación de la petición HTTP

```bash
curl -v --max-time 5 http://127.0.0.1:80/
```

Resultado:

```text
Trying 127.0.0.1...
```

La conexión terminaba en timeout sin llegar a mostrar:

```text
Connected to 127.0.0.1
```

Esto indicaba que el problema se encontraba en el establecimiento de la conexión TCP, antes de que Apache pudiera procesar la petición HTTP.

### 2.5. Comprobación de los logs

```bash
tail -f /var/log/apache2/access.log /var/log/apache2/error.log
```

Al repetir la petición no aparecían nuevas entradas en los logs.

Esto reforzaba la hipótesis de que la petición no estaba llegando a Apache.

---

## 3. Investigación

### 3.1. Comprobación de la interfaz loopback

Se verificó la configuración de la interfaz local:

```bash
ip addr show lo
```

La interfaz loopback estaba correctamente configurada con la dirección:

```text
127.0.0.1/8
```

También se comprobó la tabla de rutas locales.

No se encontraron problemas en la configuración de loopback.

### 3.2. Investigación del firewall

Se revisaron las reglas de filtrado:

```bash
sudo iptables -L -n -v
```

Se encontró una regla que descartaba el tráfico TCP destinado al puerto 80.

La regla era:

```bash
-A INPUT -p tcp -m tcp --dport 80 -j DROP
```

La política general de INPUT era:

```bash
-P INPUT ACCEPT
```

Esto significaba que, aunque la política general permitía las conexiones entrantes, existía una regla específica que descartaba todo el tráfico TCP destinado al puerto 80.

Se comprobó también el backend de iptables:

```bash
iptables --version
```

Resultado: `nf_tables`.

Por tanto, iptables estaba utilizando nftables como backend.

Se inspeccionaron las reglas directamente mediante:

```bash
sudo nft list ruleset
```

Se encontró una cadena INPUT con una regla DROP para TCP destinado al puerto 80.

El servicio `nftables` aparecía como `inactive (dead)`, pero esto no significaba que no existieran reglas activas en el kernel.

**Conclusión:** el firewall estaba bloqueando las conexiones locales al puerto 80, incluso aquellas dirigidas a `127.0.0.1`.

---

## 4. Causa raíz

Durante el diagnóstico se identificaron dos problemas independientes.

### Problema 1. Regla DROP en el firewall

La regla:

```bash
-A INPUT -p tcp -m tcp --dport 80 -j DROP
```

Descartaba las conexiones TCP destinadas al puerto 80.

No existía una excepción previa para el tráfico de loopback (`lo`).

Por tanto, las peticiones realizadas desde el propio servidor no podían establecer correctamente la conexión con Apache.

### Problema 2. Permisos incorrectos en index.html

Una vez solucionado el bloqueo del firewall, Apache comenzó a responder, pero devolvía:

```text
HTTP/1.1 403 Forbidden
```

Se comprobaron los permisos del fichero:

```bash
ls -l /var/www/html/index.html
```

Resultado:

```text
-rw------- 1 root root index.html
```

Los permisos eran `600`.

Esto significaba que únicamente el propietario (`root`) podía leer y modificar el fichero.

Los procesos hijo de Apache se ejecutaban como `www-data`, por lo que no tenían permisos de lectura.

---

## 5. Solución aplicada

### 5.1. Permitir el tráfico de loopback

Se añadió una regla al principio de la cadena INPUT:

```bash
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

Explicación:
- `-I INPUT 1`: inserta la regla en la primera posición de INPUT.
- `-i lo`: identifica el tráfico que entra por la interfaz loopback.
- `-j ACCEPT`: permite ese tráfico.

Al situarla antes de la regla DROP, las conexiones locales podían llegar correctamente a Apache.

### 5.2. Corregir los permisos del fichero

Inicialmente se aplicaron estos permisos:

```bash
sudo chmod 604 /var/www/html/index.html
```

Esto permitió que Apache pudiera leer el fichero, ya que los permisos de otros usuarios incluían lectura.

Sin embargo, otra posible configuración un poco más permisiva podría ser:

```bash
sudo chmod 644 /var/www/html/index.html
```

Resultado esperado:

```text
-rw-r--r-- 1 root root index.html
```

De esta forma:
- Propietario: lectura y escritura.
- Grupo: lectura.
- Otros usuarios: lectura.

Apache puede servir el contenido sin disponer de permisos de escritura innecesarios.

---

## 6. Validación

Se volvió a ejecutar la petición HTTP:

```bash
curl 127.0.0.1:80
```

Resultado esperado:

```text
hello sadserver
```

Se confirmó que:
- La conexión TCP se establecía correctamente.
- Apache recibía y procesaba la petición HTTP.
- El fichero `index.html` podía ser leído por el usuario `www-data`.
- Desaparecía el timeout inicial.
- Desaparecía el error HTTP 403.

**Resultado final: escenario resuelto. Apache sirve correctamente el contenido solicitado.**

---

## 7. Aprendizajes

Esta práctica permitió trabajar diferentes áreas de administración de sistemas Linux.

### Apache2
- Comprobación del estado de Apache mediante systemd.
- Diferencia entre el proceso principal (`root`) y los procesos hijo (`www-data`).
- Comprobación de puertos en escucha.
- Interpretación de respuestas HTTP 403.

### Networking
- Diferencia entre conexión TCP y petición HTTP.
- Funcionamiento de la interfaz loopback (`lo`).
- Diagnóstico de conexiones mediante `curl -v`.
- Comprobación de puertos mediante `ss`.

### Firewall y seguridad
- Gestión de reglas mediante iptables.
- Funcionamiento del backend `nf_tables`.
- Inspección de reglas mediante `nft list ruleset`.
- Orden de evaluación de las reglas de firewall.
- Excepciones para el tráfico local.

### Permisos Linux
- Interpretación de permisos octales (`600`, `604`, `644`).
- Diferencia entre propietario, grupo y otros usuarios.
- Relación entre los permisos del sistema de archivos y el usuario que ejecuta Apache.
- Aplicación del principio de mínimo privilegio.

### Metodología de troubleshooting

Una de las principales conclusiones es la importancia de diagnosticar los problemas por capas:

1. Comprobar que el servicio está activo.
2. Verificar que el puerto está escuchando.
3. Comprobar que la conexión TCP puede establecerse.
4. Investigar las reglas del firewall.
5. Comprobar que Apache procesa la petición.
6. Revisar los permisos del recurso solicitado.

Un timeout y un HTTP 403 son síntomas diferentes y requieren investigaciones distintas.

---

## 8. Consideraciones para un entorno de producción

Aunque el escenario queda resuelto, en un entorno real sería necesario completar algunas comprobaciones:

- Investigar qué servicio o script introduce la regla DROP.
- Comprobar cómo se persisten las reglas de firewall.
- Verificar el acceso HTTP desde otros equipos de la red.
- Mantener una política de firewall restrictiva, permitiendo únicamente el tráfico necesario.
- Evitar permisos de escritura innecesarios sobre los recursos web.
- Comprobar que las reglas de loopback y acceso remoto están correctamente ordenadas.

**Importante:** la regla introducida mediante `iptables -I` modifica el firewall en tiempo de ejecución. Puede perderse al reiniciar el servidor si no se guarda mediante el mecanismo de persistencia correspondiente.

---

## 9. Referencias

- [SadServers – Scenario Tokyo](https://sadservers.com/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [Debian Apache2 Documentation](https://wiki.debian.org/Apache)
- [iptables Manual](https://man7.org/linux/man-pages/man8/iptables.8.html)
- [nftables Documentation](https://wiki.nftables.org/)
