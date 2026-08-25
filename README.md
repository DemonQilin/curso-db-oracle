# Entorno Oracle Database — Administración de Base de Datos

Entorno reproducible con **Oracle Database Free (23ai)** corriendo en Docker, listo para usar desde VS Code mediante **Dev Containers**. Incluye la extensión "Oracle Developer Tools for VS Code" preinstalada.

## Requisitos previos

Antes de abrir este proyecto, instala en tu máquina:

1. **[Docker Desktop](https://www.docker.com/products/docker-desktop/)** (Windows/Mac) o Docker Engine (Linux)
2. **[Visual Studio Code](https://code.visualstudio.com/)**
3. La extensión de VS Code **[Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)** (`ms-vscode-remote.remote-containers`)

## Estructura del proyecto

```
.
├── .devcontainer/
│   ├── devcontainer.json     # Configuración de VS Code para el Dev Container
│   ├── docker-compose.yml    # Definición de los contenedores (app + oracle)
│   ├── .env.example           # Plantilla de variables de entorno (sí se sube a Git)
│   └── .env                    # Tus contraseñas reales (NO se sube a Git)
└── .gitignore
```

## Puesta en marcha (primera vez)

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd <carpeta-del-proyecto>
```

### 2. Crear tu archivo de variables de entorno

```bash
cp .devcontainer/.env.example .devcontainer/.env
```

Abre `.devcontainer/.env` y define tus propias contraseñas:

```
ORACLE_PASSWORD=TuPasswordSegura123
APP_USER=estudiante
APP_USER_PASSWORD=OtraPassword456
```

> Este archivo es local a tu máquina y está en `.gitignore` — nunca se comparte ni se sube al repositorio.
>
> **Importante:** el `.env` debe estar dentro de `.devcontainer/`, junto al `docker-compose.yml`. Docker Compose busca el archivo `.env` en el mismo directorio que el `docker-compose.yml` que está usando; si lo pones en la raíz del proyecto, las variables quedan vacías y el contenedor de Oracle falla al arrancar.

### 3. Abrir el proyecto en VS Code

```bash
code .
```

### 4. Reabrir en el Dev Container

VS Code mostrará una notificación abajo a la derecha: **"Reopen in Container"**. Haz clic ahí.

(Si no aparece la notificación: `Ctrl+Shift+P` → busca **"Dev Containers: Reopen in Container"**)

### 5. Esperar la inicialización

La primera vez, Docker debe descargar las imágenes y Oracle debe inicializar la base de datos — puede tardar varios minutos. Puedes seguir el progreso en la pestaña **"Dev Containers"** del panel de salida (Output) de VS Code.

Cuando termine, ya tienes:

- Oracle Database corriendo en `localhost:1521`
- La extensión de Oracle instalada y lista dentro del contenedor

## Datos de conexión

Usa estos datos en la extensión de Oracle, DBeaver, SQL Developer, etc.:

| Campo             | Valor                                                           |
| ----------------- | --------------------------------------------------------------- |
| Host              | `localhost`                                                     |
| Puerto            | `1521`                                                          |
| Service name      | `FREEPDB1`                                                      |
| Usuario           | `estudiante` (o `system` para tareas administrativas)           |
| Contraseña        | La que definiste en tu `.devcontainer/.env`                     |
| Role / Connect as | `Default` (deja `SYSDBA` únicamente para conectarte como `sys`) |

## Comandos útiles

Desde una terminal en la raíz del proyecto (fuera o dentro del Dev Container):

```bash
# Detener los contenedores sin perder datos
docker compose -f .devcontainer/docker-compose.yml stop

# Volver a levantarlos
docker compose -f .devcontainer/docker-compose.yml start

# Ver logs de Oracle (útil si algo no arranca)
docker compose -f .devcontainer/docker-compose.yml logs -f oracle

# Ver el estado de los contenedores
docker compose -f .devcontainer/docker-compose.yml ps
```

## Persistencia de datos

- Tus datos (tablas, usuarios, cambios) se guardan en un volumen de Docker y **sobreviven** cada vez que cierras y vuelves a abrir el Dev Container en tu máquina.
- Los datos **no se comparten entre compañeros** — cada quien tiene su propia copia local, aislada. Lo único que se comparte vía Git es la configuración (este README, el `docker-compose.yml`, el `devcontainer.json`).

## Empezar de cero (borrar todo y reiniciar)

Si necesitas resetear completamente tu base de datos local:

```bash
docker compose -f .devcontainer/docker-compose.yml down -v
```

El flag `-v` elimina también el volumen de datos. La próxima vez que reabras el Dev Container, Oracle se inicializará desde cero.

## Solución de problemas

**"Credenciales inválidas" en la extensión de VS Code:**
Verifica que el campo _Role_ esté en `Default` y no en `SYSDBA` (ese modo es solo para conectarte como `sys`).

**El contenedor tarda mucho en arrancar la primera vez:**
Es normal — Oracle inicializa la base de datos completa en el primer arranque (varios minutos). Los arranques siguientes son mucho más rápidos.

**Necesito privilegios de administrador (DBA) con mi usuario:**
Conéctate como `system` y ejecuta:

```sql
GRANT DBA TO estudiante;
```

**El contenedor `oracle` sale con "Error" apenas arranca / el log dice "Oracle Database SYS and SYSTEM passwords have to be specified":**
El archivo `.env` no está en `.devcontainer/.env` (revisa el paso 2), o el contenedor ya se había creado antes con las variables vacías y no se recreó. Corrígelo así:

```bash
docker compose -f .devcontainer/docker-compose.yml down -v
```

y vuelve a reabrir el Dev Container para que se cree desde cero con las variables correctas.

## Limpieza total (desinstalar todo)

Al terminar el curso, para no dejar ningún rastro en tu máquina:

```bash
docker compose -f .devcontainer/docker-compose.yml down -v --rmi all
```

Esto elimina los contenedores, el volumen de datos y las imágenes descargadas.
