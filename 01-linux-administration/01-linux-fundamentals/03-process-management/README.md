
# Process Management

## 1. Objetivo

Comprender cómo funcionan los procesos en Linux y aprender a identificarlos, consultarlos y gestionarlos mediante comandos de administración del sistema.

## 2. Conceptos fundamentales

Un proceso es una instancia de un programa en ejecución.

Cada proceso dispone de identificadores que permiten distinguirlo de los demás.

- PID (Process ID): identificador único del proceso.
- PPID (Parent Process ID): identificador del proceso padre.

Los procesos suelen estar organizados en una jerarquía, donde un proceso puede crear otros procesos.

En sistemas Linux modernos, `systemd` suele ser el proceso inicial del sistema y utiliza el PID 1.

## 3. Consulta de procesos

### ps -ef

Muestra un listado de procesos en ejecución con información detallada.

```bash
ps -ef
```

Entre las columnas más relevantes:

- UID: usuario propietario del proceso.
- PID: identificador del proceso.
- PPID: identificador del proceso padre.
- CMD: comando que inició el proceso.

### pgrep

Permite localizar procesos a partir de su nombre.

```bash
pgrep nombre_proceso
```

### Identificar el PID de la shell actual

```bash
echo $$
```

La variable `$$` contiene el PID del proceso de la shell actual.

## 4. Ejecución de procesos en segundo plano

Linux permite ejecutar comandos en segundo plano utilizando `&`.

Ejemplo:

```bash
sleep 300 &
```

El comando inicia un proceso que permanece ejecutándose durante 300 segundos sin mantener la terminal ocupada.

Al ejecutarlo, la shell puede mostrar información como:

```text
[1] 3000
```

- `[1]`: número de trabajo o Job ID.
- `3000`: PID del proceso.

Los valores son ilustrativos y pueden variar en cada ejecución.

## 5. Gestión de trabajos

### jobs

Muestra los trabajos asociados a la shell actual.

```bash
jobs
```

Permite consultar los procesos iniciados como trabajos desde esa sesión.

Es importante distinguir entre:

- Job ID: identificador del trabajo dentro de la shell.
- PID: identificador del proceso en el sistema.

### kill

Permite enviar señales a un proceso.

Ejemplo:

```bash
kill PID
```

Por defecto, `kill` envía la señal SIGTERM, solicitando al proceso que termine de forma controlada.

Si fuera necesario, existen otras señales, como SIGKILL, aunque debe evitarse utilizarla salvo que sea imprescindible.

## 6. Comprobaciones

Al trabajar con procesos se pueden combinar diferentes comandos:

```bash
ps -ef
pgrep sleep
jobs
echo $$
```

Estos comandos permiten consultar procesos, localizar ejecuciones concretas y comprobar los trabajos asociados a la terminal.

## 7. Conclusiones

En esta práctica se trabajaron los fundamentos de gestión de procesos:

- Identificación mediante PID y PPID.
- Consulta de procesos con `ps`.
- Localización mediante `pgrep`.
- Identificación del PID de la shell.
- Ejecución de procesos en segundo plano.
- Consulta de trabajos mediante `jobs`.
- Finalización de procesos mediante `kill`.

La gestión de procesos es una habilidad fundamental para diagnosticar problemas de consumo de recursos, servicios que no responden y aplicaciones que permanecen ejecutándose en el sistema.

