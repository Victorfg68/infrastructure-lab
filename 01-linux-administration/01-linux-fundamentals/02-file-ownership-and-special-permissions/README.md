
# File Ownership and Special Permissions

## 1. Objetivo

Comprender la propiedad de archivos y directorios en Linux, así como los permisos especiales y la máscara de creación de archivos (`umask`).

## 2. Propietario y grupo

En Linux, cada archivo y directorio tiene asociado:

- Un propietario (user).
- Un grupo propietario (group).

Estos atributos permiten controlar quién puede acceder a los recursos.

### Comando chown

Permite modificar el propietario y el grupo de un archivo o directorio.

Ejemplo trabajado en el laboratorio:

```bash
chown Admin:developers /proyecto
```

En este caso:

- `Admin`: nuevo propietario.
- `developers`: nuevo grupo propietario.
- `/proyecto`: directorio al que se aplica el cambio.

Para consultar propietario y grupo:

```bash
ls -ld /proyecto
```

## 3. Permisos especiales

Linux dispone de tres permisos especiales: SUID, SGID y Sticky Bit.

### 3.1. SUID

El bit SUID aparece en la posición de ejecución correspondiente al propietario.

Se representa mediante `s` o `S`.

En archivos ejecutables, permite que el programa se ejecute con los privilegios efectivos del propietario del archivo.

### 3.2. SGID

El bit SGID aparece en la posición de ejecución correspondiente al grupo.

En directorios, permite que los archivos y subdirectorios creados dentro hereden el grupo propietario del directorio padre.

Comando utilizado:

```bash
chmod g+s directorio
```

Ejemplo de representación:

```text
drwxrws---
```

El grupo aparece con una `s` en lugar de `x`.

### 3.3. Sticky Bit

El Sticky Bit se representa mediante `t` o `T` en la posición de ejecución de otros usuarios.

En directorios compartidos, restringe el borrado y renombrado de archivos: normalmente solo puede hacerlo el propietario del archivo, el propietario del directorio o root.

## 4. Umask

La máscara `umask` determina qué permisos se eliminan de los permisos iniciales al crear archivos y directorios.

Se aplica al proceso o sesión y puede configurarse en los archivos de inicio del usuario.

Consulta de la máscara:

```bash
umask
```

Ejemplo:

```bash
umask 0022
```

Con esta configuración, habitualmente se obtienen:

| Tipo | Permisos iniciales | Resultado |
|---|---|---|
| Archivo | 666 | 644 |
| Directorio | 777 | 755 |

La umask elimina permisos de los valores iniciales; no concede permisos adicionales.

## 5. Estructura de directorios de una aplicación

En el laboratorio se preparó una estructura de directorios para una aplicación:

```text
/srv/dev-app/
├── app/
├── logs/
├── config/
└── scripts/
```

Cada directorio representa una función diferente:

- `app`: archivos de la aplicación.
- `logs`: registros de actividad.
- `config`: archivos de configuración.
- `scripts`: scripts y tareas auxiliares.

Se trabajó con permisos y SGID para facilitar el acceso compartido mediante el grupo `developers`.

## 6. Comprobaciones

Para consultar propietarios, grupos y permisos:

```bash
ls -ld /srv/dev-app
ls -l /srv/dev-app
```

Para consultar la máscara:

```bash
umask
```

## 7. Conclusiones

En esta práctica se trabajaron:

- Propietarios y grupos mediante `chown`.
- Permisos especiales SUID, SGID y Sticky Bit.
- Herencia de grupo en directorios mediante SGID.
- Funcionamiento de `umask`.
- Organización de directorios para una aplicación.
- Aplicación de permisos para facilitar el trabajo colaborativo.

La correcta configuración de propietarios y permisos permite organizar el acceso a los recursos y aplicar el principio de mínimo privilegio.

