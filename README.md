# GNS3 NetForge MCP

Servidor **Model Context Protocol (MCP)** para automatizar laboratorios de red en GNS3. Permite administrar proyectos, routers, switches, VPCS, máquinas virtuales, enlaces y consolas desde Claude Code, Claude Desktop, Codex u otro cliente compatible con MCP.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![MCP](https://img.shields.io/badge/Protocolo-Model%20Context%20Protocol-6E56CF)](https://modelcontextprotocol.io/)
[![GNS3](https://img.shields.io/badge/GNS3-Automatización-00AEEF)](https://www.gns3.com/)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-green.svg)](LICENSE)

<p align="center">
  <img src="docs/assets/gns3-netforge-banner.svg" alt="GNS3 NetForge MCP conectado a una topología GNS3" width="760">
</p>

## Funciones principales

- Crear, abrir, cerrar y consultar proyectos de GNS3.
- Crear, iniciar, detener y reiniciar nodos.
- Crear y eliminar enlaces entre dispositivos.
- Ejecutar comandos en consolas Telnet y SSH.
- Automatizar routers, switches, VPCS y máquinas virtuales.
- Consultar topologías, enlaces, sesiones y reportes.
- Ejecutar operaciones por lotes y trabajar con Docker.
- Usar autenticación por API Key en modo HTTP.

## Arquitectura

~~~text
Cliente MCP → GNS3 NetForge MCP → API de GNS3 → Routers, switches, VPCS y máquinas virtuales
~~~

<p align="center">
  <img src="mcp-server/docs/diagrams/topology_discovery_workflow.svg" alt="Flujo de descubrimiento de topologías GNS3" width="760">
</p>

Para consultar el flujo completo de creación de laboratorios, revisa [lab_setup_workflow.svg](mcp-server/docs/diagrams/lab_setup_workflow.svg).

## Requisitos

- Windows 10/11, Linux o macOS.
- Python 3.10 o superior.
- GNS3 Server instalado, encendido y accesible.
- Claude Code, Claude Desktop, Codex u otro cliente MCP.
- uv o uvx para la instalación recomendada.

## Instalación rápida con uvx

Instala uv:

~~~powershell
pip install uv
~~~

Registra el servidor en Claude Code. Cambia los valores por los datos de tu servidor GNS3:

~~~powershell
claude mcp add --transport stdio gns3-mcp --env GNS3_HOST=192.168.1.20 --env GNS3_PORT=80 --env GNS3_USER=admin --env GNS3_PASSWORD=tu-contrasena --scope user -- uvx gns3-mcp@latest
~~~

Comprueba la conexión:

~~~powershell
claude mcp get gns3-mcp
~~~

## Instalación desde el código fuente

~~~powershell
git clone https://github.com/Arminfx7/gns3-netforge-mcp.git
cd gns3-netforge-mcp
python -m venv .venv
.\\.venv\\Scripts\\Activate.ps1
python -m pip install --upgrade pip
pip install -e .
~~~

Después registra el ejecutable local:

~~~powershell
claude mcp add --transport stdio gns3-mcp --env GNS3_HOST=192.168.1.20 --env GNS3_PORT=80 --env GNS3_USER=admin --env GNS3_PASSWORD=tu-contrasena --scope user -- gns3-mcp
~~~

## Variables de configuración

Puedes usar un archivo .env basado en la plantilla:

~~~powershell
Copy-Item .env.example .env
notepad .env
~~~

| Variable | Obligatoria | Ejemplo | Descripción |
|---|---:|---|---|
| GNS3_HOST | Sí | 192.168.1.20 | IP o nombre del servidor GNS3 |
| GNS3_PORT | No | 80 | Puerto de la API de GNS3 |
| GNS3_USER | Sí | admin | Usuario de GNS3 |
| GNS3_PASSWORD | Sí | tu-contrasena | Contraseña de GNS3 |
| GNS3_USE_HTTPS | No | false | Activa HTTPS |
| GNS3_VERIFY_SSL | No | true | Verifica el certificado SSL |
| LOG_LEVEL | No | INFO | Nivel de registro |

Nunca publiques un archivo .env ni contraseñas reales.

## Configuración en Claude Desktop

Agrega el servidor en la configuración MCP:

~~~json
{
  "mcpServers": {
    "gns3-mcp": {
      "command": "uvx",
      "args": ["gns3-mcp@latest"],
      "env": {
        "GNS3_HOST": "192.168.1.20",
        "GNS3_PORT": "80",
        "GNS3_USER": "admin",
        "GNS3_PASSWORD": "tu-contrasena"
      }
    }
  }
}
~~~

Reinicia Claude Desktop después de guardar el archivo.

## Modo HTTP

~~~powershell
$env:GNS3_HOST="192.168.1.20"
$env:GNS3_PORT="80"
$env:GNS3_USER="admin"
$env:GNS3_PASSWORD="tu-contrasena"
gns3-mcp --transport http --host 0.0.0.0 --port 8000
~~~

Comprueba el servicio:

~~~powershell
curl http://localhost:8000/health
~~~

## Docker

Requisitos: Docker Desktop y acceso de red al servidor GNS3.

~~~powershell
Copy-Item .env.example .env
docker compose up -d
docker compose logs -f
~~~

Para detener los contenedores:

~~~powershell
docker compose down
~~~

## Desarrollo y pruebas

~~~powershell
pip install -e ".[dev]"
python -m pytest
ruff check .
~~~

## Solución de problemas

Si no conecta con GNS3:

1. Confirma que GNS3 Server esté encendido.
2. Comprueba la IP, el puerto y las credenciales.
3. Verifica el firewall.
4. Prueba la API:

~~~powershell
curl http://192.168.1.20:80/v3/version
~~~

Si Claude no muestra el servidor:

~~~powershell
claude mcp get gns3-mcp
claude mcp list
~~~

Para problemas con Docker:

~~~powershell
docker compose ps
docker compose logs gns3-mcp
docker compose restart
~~~

## Documentación adicional

- [DEPLOYMENT.md](DEPLOYMENT.md): despliegue y proxy SSH.
- [PORTABLE_SETUP.md](PORTABLE_SETUP.md): instalación portable en Windows.
- [PUBLISHING.md](PUBLISHING.md): publicación y versiones.
- [docs/architecture](docs/architecture/): diagramas y arquitectura.
- [skill/SKILL.md](skill/SKILL.md): instrucciones para agentes compatibles.

## Créditos y procedencia

Este repositorio es una copia/adaptación de **GNS3 MCP Server**. El código base fue extraído de:

- [ChistokhinSV/gns3-mcp](https://github.com/ChistokhinSV/gns3-mcp)
- Autor original: Sergei Chistokhin.
- Protocolo utilizado: [Model Context Protocol](https://modelcontextprotocol.io/).

Se conserva la licencia MIT y los avisos de copyright del proyecto original. Las modificaciones y la publicación de esta copia corresponden a [Arminfx7/gns3-netforge-mcp](https://github.com/Arminfx7/gns3-netforge-mcp).

## Licencia

Este proyecto se distribuye bajo la licencia MIT. Consulta [LICENSE](LICENSE).
