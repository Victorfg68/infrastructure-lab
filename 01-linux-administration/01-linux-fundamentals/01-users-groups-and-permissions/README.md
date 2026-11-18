
# Users, Groups and Permissions

## 1. Objetivo

Practicar la administración básica de usuarios, grupos y permisos en Linux, comprendiendo cómo se controla el acceso a archivos y directorios.

## 2. Entorno de laboratorio

- Sistema operativo: Rocky Linux 10.2.
- Plataforma: VMware.
- Usuario administrador: `Admin`.
- Usuario de pruebas: `dev1`.
- Grupo de trabajo: `developers`.

## 3. Gestión de usuarios

Linux permite crear diferentes usuarios para separar las tareas y los permisos de acceso al sistema.

### Comandos utilizados

**whoami**

Muestra el nombre del usuario que está ejecutando la sesión actual.

```bash
whoami
```

**id**

Muestra el UID, GID y los grupos a los que pertenece un usuario.

```bash
id
id dev1
```

**su - usuario**

Permite cambiar a otro usuario iniciando una sesión de login.

```bash
su - dev1
```

El guion permite cargar el entorno de inicio del usuario al que cambiamos.

## 4. Usuarios, grupos y sudo

Los grupos permiten gestionar permisos compartidos entre diferentes usuarios.

En este laboratorio se trabajó con el grupo `developers` y con el grupo `wheel`.

En Rocky Linux, `wheel` se utiliza habitualmente para conceder permisos de administración mediante `sudo`, siempre que la configuración de sudo lo permita.

Comprobación de los grupos de un usuario:

```bash
id dev1
```

## 5. Permisos básicos de Linux

Linux utiliza tres permisos principales:

| Permiso | Valor | Significado |
|---|---:|---|
| Read (r) | 4 | Lectura |
| Write (w) | 2 | Escritura |
| Execute (x) | 1 | Ejecución |

Los permisos se aplican a tres categorías:

- Propietario (user).
- Grupo (group).
- Otros usuarios (others).

### Ejemplo: chmod 770

```bash
chmod 770 directorio
```

El resultado es:

```text
rwxrwx---
```

Esto significa:

- Propietario: lectura, escritura y ejecución.
- Grupo: lectura, escritura y ejecución.
- Otros: sin permisos.

### Interpretación de permisos

Ejemplo:

```text
drwxrwx---
```

- `d`: indica que es un directorio.
- Primer bloque `rwx`: permisos del propietario.
- Segundo bloque `rwx`: permisos del grupo.
- Tercer bloque `---`: permisos de otros usuarios.

## 6. Comprobación de permisos

Para consultar los permisos de archivos y directorios:

```bash
ls -l
ls -ld directorio
```

## 7. Conclusiones

En esta práctica se trabajaron los fundamentos de:

- Identificación de usuarios mediante `whoami` e `id`.
- Cambio de usuario mediante `su`.
- Organización de usuarios mediante grupos.
- Grupo `wheel` y permisos de administración con sudo.
- Interpretación de permisos Linux.
- Asignación de permisos mediante `chmod`.

La correcta gestión de usuarios, grupos y permisos es fundamental para mantener la seguridad y separar responsabilidades en un sistema Linux.

