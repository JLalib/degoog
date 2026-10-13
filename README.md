# 🔍 Degoog | Agregador de motores de búsqueda autohospedado con sistema de plugins

[![GitHub Repo stars](https://img.shields.io/github/stars/degoog-org/degoog?style=for-the-badge)](https://github.com/degoog-org/degoog)
[![GitHub forks](https://img.shields.io/github/forks/degoog-org/degoog?style=for-the-badge)](https://github.com/degoog-org/degoog)
[![GitHub issues](https://img.shields.io/github/issues/degoog-org/degoog?style=for-the-badge)](https://github.com/degoog-org/degoog/issues)
[![GitHub license](https://img.shields.io/github/license/degoog-org/degoog?style=for-the-badge)](https://github.com/degoog-org/degoog/blob/main/LICENSE)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/degoog-org/degoog?style=for-the-badge)](https://ghcr.io/degoog-org/degoog)
[![Docker Image Version](https://img.shields.io/docker/v/ghcr.io/degoog-org/degoog?style=for-the-badge)](https://ghcr.io/degoog-org/degoog)

## 📋 Descripción general

**Degoog** es un agregador de motores de búsqueda de código abierto escrito en **TypeScript** y ejecutado sobre **Bun**. Permite consultar varios motores a la vez y presentar los resultados en una única interfaz. Su arquitectura modular soporta motores de búsqueda personalizados, plugins de comandos bang, plugins de ranura (paneles activados por consultas) y transportes (estrategias HTTP personalizadas como curl o FlareSolverr). El proyecto está en beta estable y se distribuye bajo licencia **AGPL-3.0**.

> 🎥 **Vídeo tutorial**: [Mira el vídeo completo en YouTube](https://www.youtube.com/@GENBYTE) | 📖 **Documentación oficial**: [degoog-org.github.io/docs](https://degoog-org.github.io/docs/)

## ✨ Características principales

- 🔄 **Agregación multi-motor**: Consulta simultánea a múltiples motores de búsqueda y fusión de resultados en una sola vista
- 🧩 **Sistema de plugins completo**: Cuatro tipos de extensiones: motores de búsqueda, comandos bang, paneles de ranura (slots) y transportes HTTP personalizados
- 🏪 **Tienda de extensiones comunitaria**: Repositorio oficial y repositorios comunitarios para descubrir e instalar plugins con un clic desde la interfaz
- 🎨 **Temas y personalización**: Soporte para temas visuales propios y alias de búsqueda para atajos personalizados
- 🔒 **Seguridad integrada**: Protección de ajustes mediante contraseña (`DEGOOG_SETTINGS_PASSWORDS`) para evitar instalación de código no autorizado en instancias públicas
- 📈 **Escalabilidad opcional**: Integración nativa con Valkey/Redis para caché compartido y PostgreSQL para índice de texto completo en instancias de alto tráfico
- 🐳 **Despliegue flexible**: Imagen Docker oficial, ejemplos de Docker Compose para distintos perfiles, módulo NixOS y ejecución nativa con Bun
- 🤖 **Compatibilidad MCP**: Compose dedicado (`mcp.yml`) para exponer Degoog a clientes MCP como Claude, Cursor o llama.cpp junto a la UI web

## 📋 Requisitos del sistema

- ✅ **Docker** y **Docker Compose** (recomendado)
- ✅ Para ejecución nativa: **Bun 1.3.14**, **Git**, **curl** y **ca-certificates**
- ✅ Directorio de datos con permisos para **UID/GID 1000:1000**
- ✅ Variable `DEGOOG_SETTINGS_PASSWORDS` definida antes de exponer a internet

## 🐳 Instalación

### Docker Compose simple (uso personal)

```bash
mkdir -p ./data
sudo chown -R 1000:1000 ./data
```

```yaml
# docker-compose.yml
services:
  degoog:
    image: ghcr.io/degoog-org/degoog:latest
    volumes:
      - ./data:/app/data
    ports:
      - "4444:4444"
    environment:
      - DEGOOG_SETTINGS_PASSWORDS=tu_password_seguro
    restart: unless-stopped
```

```bash
docker compose up -d
```

### Docker Compose con Valkey (caché compartido)

```yaml
# docker-compose.yml
services:
  degoog:
    image: ghcr.io/degoog-org/degoog:latest
    volumes:
      - ./data:/app/data
    ports:
      - "4444:4444"
    environment:
      - DEGOOG_SETTINGS_PASSWORDS=tu_password_seguro
    restart: unless-stopped

  valkey:
    image: valkey/valkey:latest
    volumes:
      - ./valkey-data:/data
    restart: unless-stopped
```

```bash
docker compose up -d
```

### Docker Compose con PostgreSQL (índice FTS escalable)

```yaml
# docker-compose.yml
services:
  degoog:
    image: ghcr.io/degoog-org/degoog:latest
    volumes:
      - ./data:/app/data
    ports:
      - "4444:4444"
    environment:
      - DEGOOG_SETTINGS_PASSWORDS=tu_password_seguro
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=degoog
      - POSTGRES_USER=degoog
      - POSTGRES_PASSWORD=password_seguro
    restart: unless-stopped
```

```bash
docker compose up -d
```

### Ejecución nativa con Bun

```bash
git clone https://github.com/degoog-org/degoog.git
cd degoog
bun install
bun run build
bun run start
```

## ⚙️ Configuración

1. **Protección de ajustes**: Define `DEGOOG_SETTINGS_PASSWORDS` con una contraseña segura antes de exponer la instancia a internet
2. **Directorio de datos**: Asegúrate de que `./data` tenga permisos para UID/GID 1000:1000
3. **Puertos**: Por defecto usa el puerto 4444 (configurable via `ports` en docker-compose)
4. **Variables de entorno principales**:
   - `DEGOOG_ENGINES_DIR=./data/engines`
   - `DEGOOG_PLUGINS_DIR=./data/plugins`
   - `DEGOOG_THEMES_DIR=./data/themes`
   - `DEGOOG_TRANSPORTS_DIR=./data/transports`
   - `DEGOOG_ALIASES_FILE=./data/aliases.json`
   - `DEGOOG_PLUGIN_SETTINGS_FILE=./data/plugin-settings.json`
   - `DEGOOG_DEFAULT_ENGINES_FILE=./data/default-engines.json`
   - `DEGOOG_SETTINGS_TOKENS_FILE=./data/settings-tokens.json`
   - `DEGOOG_AUTOCOMPLETE_DIR=./data/autocomplete`
   - `DEGOOG_BLOCKLIST_FILE=./data/blocklist.json`
5. **Valkey/Redis**: Configura la conexión para caché compartido entre réplicas
6. **PostgreSQL**: Habilita para índice de texto completo (FTS) en instancias de alto tráfico
7. **MCP**: Usa `mcp.yml` para exponer a clientes MCP (Claude, Cursor, llama.cpp)

## 🚀 Primeros pasos

1. Clona o crea el directorio del proyecto y asigna permisos al directorio de datos
2. Crea el archivo `docker-compose.yml` con la configuración deseada (simple, Valkey o PostgreSQL)
3. Define la variable `DEGOOG_SETTINGS_PASSWORDS` con una contraseña segura
4. Levanta los contenedores con `docker compose up -d`
5. Accede a la interfaz web en `http://localhost:4444`
6. Configura motores, plugins, temas y ajustes desde la UI
7. Verifica el healthcheck en `http://localhost:4444/readyz`

## 💡 Casos de uso

- 🔐 **Búsqueda privada** sin seguimiento ni perfiles publicitarios
- 💻 **Agregación de resultados técnicos** (GitHub, Stack Overflow, docs) junto a web general
- 🕵️ **Investigación OSINT** combinando motores especializados y bangs personalizados
- 🤖 **Exposición a LLMs vía MCP** para búsquedas en tiempo real desde asistentes de código
- 🌐 **Instancia pública comunitaria** con caché compartido (Valkey) y índice PostgreSQL
- 🛠️ **Desarrollo de plugins propios**: motores, bangs, paneles UI o transportes HTTP

## 🔒 Acceso remoto seguro

Para exponer Degoog de forma segura a internet:

- Usa un **reverse proxy** (Nginx, Traefik, Caddy) con **TLS/SSL**
- Configura **autenticación básica** o **OAuth** en el proxy
- Mantén `DEGOOG_SETTINGS_PASSWORDS` fuerte y único
- Limita el acceso por IP si es posible
- Considera usar **Tailscale** o **WireGuard** para acceso VPN

## 🛠️ Gestión y mantenimiento

### Actualización de imagen

```bash
docker compose pull
docker compose up -d
```

> Los datos persisten en el volumen `./data`

### Backup de datos

```bash
tar -czf degoog-backup-$(date +%F).tar.gz ./data
```

> Incluye motores, plugins, temas, ajustes, bloqueos y autocompletado

### Logs y diagnóstico

```bash
docker compose logs -f degoog
curl -fsS http://localhost:4444/readyz
```

### Limpieza de caché Valkey/Redis

```bash
docker exec -it valkey_container valkey-cli FLUSHALL
```

## 📝 Licencia

Este proyecto está licenciado bajo **AGPL-3.0** - ver el archivo [LICENSE](https://github.com/degoog-org/degoog/blob/main/LICENSE) para más detalles.

---

> 📖 **Artículo original**: [Cómo instalar Degoog en Docker | Agregador de motores de búsqueda autohospeada con sistema de Plugins](https://genbyte.blogspot.com/2026/10/como-instalar-degoog-en-docker.html)  
> 🎥 **Canal YouTube**: [GENBYTE](https://www.youtube.com/@GENBYTE)  
> ☕ **Apoya el proyecto**: [Ko-fi](https://ko-fi.com/genbyte)