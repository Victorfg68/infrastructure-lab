
# Systemd Services

## 1. Objetivo

Aprender a gestionar servicios Linux mediante `systemd` y su herramienta de administración `systemctl`.

## 2. ¿Qué es systemd?

`systemd` es el sistema de inicialización y gestor de servicios utilizado por Rocky Linux.

Entre sus funciones principales se encuentran:

- Iniciar servicios durante el arranque.
- Detener y reiniciar servicios.
- Consultar el estado de los servicios.
- Gestionar dependencias entre unidades.
- Configurar el inicio automático de servicios.

Un servicio es una unidad gestionada por systemd que normalmente controla la ejecución de un proceso o conjunto de procesos.

## 3. Gestión de servicios con systemctl

### Consultar el estado de un servicio

```bash
systemctl status servicio
```

Muestra información sobre el estado del servicio, incluyendo si está activo, inactivo o ha fallado.

### Iniciar un servicio

```bash
sudo systemctl start servicio
```

Inicia el servicio sin modificar su configuración de arranque automático.

### Detener un servicio

```bash
sudo systemctl stop servicio
```

Detiene el servicio actualmente en ejecución.

### Reiniciar un servicio

```bash
sudo systemctl restart servicio
```

Detiene y vuelve a iniciar el servicio.

Puede ser necesario cuando se realizan cambios de configuración que requieren reiniciar el proceso.

### Recargar la configuración

```bash
sudo systemctl reload servicio
```

Solicita al servicio que recargue su configuración sin realizar un reinicio completo.

Esta operación depende de que el servicio admita la recarga de configuración.

### Habilitar el inicio automático

```bash
sudo systemctl enable servicio
```

Configura el servicio para que se inicie automáticamente durante el arranque del sistema.

### Deshabilitar el inicio automático

```bash
sudo systemctl disable servicio
```

Elimina la configuración de inicio automático.

No detiene necesariamente un servicio que ya esté ejecutándose.

### Consultar si está habilitado

```bash
systemctl is-enabled servicio
```

Indica si el servicio está configurado para iniciarse automáticamente.

### Listar servicios activos

```bash
systemctl list-units --type=service
```

Muestra las unidades de tipo servicio que se encuentran cargadas y activas según los filtros utilizados.

## 4. Diferencias entre los comandos principales

Es importante distinguir entre el estado actual de un servicio y su configuración de arranque.

| Comando | Función |
|---|---|
| `start` | Inicia el servicio. |
| `stop` | Detiene el servicio. |
| `restart` | Reinicia el servicio. |
| `reload` | Solicita recargar la configuración. |
| `enable` | Configura el inicio automático. |
| `disable` | Deshabilita el inicio automático. |
| `status` | Consulta el estado actual. |
| `is-enabled` | Consulta si está habilitado para el arranque. |

Por ejemplo:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

El primer comando inicia Apache HTTP Server.

El segundo configura su inicio automático cuando arranque el sistema.

Son operaciones independientes.

## 5. Aplicación práctica en servidores

La gestión de servicios mediante systemd es aplicable a diferentes componentes de infraestructura.

Algunos ejemplos:

- Apache HTTP Server (`httpd`).
- SSH (`sshd`).
- Otros servicios de sistema.
- Servicios de aplicaciones que dispongan de una unidad systemd.

Apache, Tomcat y WebLogic pueden gestionarse mediante systemd cuando disponen de las correspondientes unidades de servicio y están configurados para ello.

## 6. Conclusiones

En esta práctica se trabajaron los fundamentos de administración de servicios Linux:

- Consulta del estado de servicios.
- Inicio, parada y reinicio.
- Recarga de configuración.
- Configuración del arranque automático.
- Diferencia entre estado de ejecución y habilitación.
- Identificación de servicios gestionados por systemd.

El conocimiento de systemd es fundamental para administrar servidores Linux y realizar tareas de diagnóstico y recuperación de servicios.

