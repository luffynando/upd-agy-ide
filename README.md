# Antigravity IDE Updater (`upd-agy-ide`)

Script automatizado en Bash para instalar, actualizar y configurar **Antigravity IDE** (fork de VS Code desarrollado por Google) en distribuciones basadas en Debian y Ubuntu.

---

## 🚀 Características

- **Instalación y actualización automática**: Consulta los endpoints oficiales de lanzamiento de Antigravity IDE, descarga la versión más reciente y la instala en `/opt/antigravity-ide`.
- **Detección inteligente de SO**:
  - **Ubuntu 22.04 LTS**, **Ubuntu 24.04 LTS**, **Ubuntu 26.04 LTS**.
  - **Debian 12 (Bookworm)**, **Debian 13 (Trixie)** y distribuciones derivadas (Linux Mint, Pop!_OS, PikaOS, etc.).
- **Soporte de librerías y transición time64 (`t64`)**:
  - Selecciona automáticamente paquetes `t64` (`libgtk-3-0t64`, etc.) en distros modernas (Ubuntu 24.04+, Debian 13+) o paquetes estándar (`libgtk-3-0`, etc.) en distros previas (Ubuntu 22.04, Debian 12).
- **Compatibilidad con AppArmor y User Namespaces**:
  - Configura el perfil de AppArmor necesario en Ubuntu 24.04+ y kernels con restricciones de namespaces de usuario para permitir el correcto funcionamiento del sandbox de Chromium/Electron.
- **Integración de Escritorio**:
  - Crea el acceso directo `.desktop` en `/usr/share/applications` y registra el icono del sistema.
- **Comando CLI y Alias `agy-ide` (idéntico a `code`)**:
  - Vincula el ejecutable CLI `/usr/local/bin/agy-ide` al lanzador interno (`bin/antigravity-ide`).
  - Detecta el shell del usuario (**Bash**, **Zsh** o **Fish**) e inyecta de manera limpia e idempotente el alias `agy-ide` en `~/.bashrc`, `~/.zshrc` o `~/.config/fish/config.fish`, preservando la propiedad y permisos del usuario real (no de root).
- **Configuración opcional del Marketplace de VS Code (`--vscode-marketplace`)**:
  - Al pasar el flag `--vscode-marketplace` (o `--marketplace`), inyecta en `~/.config/Antigravity IDE/User/settings.json` (si no existen ya) las URLs del Marketplace oficial de Visual Studio Code (`antigravity.marketplaceGalleryItemURL` y `antigravity.marketplaceExtensionGalleryServiceURL`) en lugar de Open VSX.
- **Migración opcional de extensiones compatibles desde VS Code (`--with-extensions`)**:
  - Al pasar el flag `--with-extensions`, si detecta Visual Studio Code instalado (vía APT, Snap o Flatpak) o el directorio `~/.vscode/extensions`, compara las extensiones instaladas con las de Antigravity IDE (`~/.antigravity-ide/extensions`).
  - Omite automáticamente todas las extensiones incompatibles o bloqueadas por Antigravity IDE (por ejemplo, `ms-vsliveshare.vsliveshare`, extensiones Remote propietarias de Microsoft, Copilot/Codeium/otros agentes en conflicto, o extensiones que requieran una versión superior del motor VS Code).
  - Migra únicamente las extensiones compatibles mediante el CLI de Antigravity IDE (con respaldo de copia local desde `~/.vscode/extensions`). Si no se pasa el flag, no realiza búsqueda ni instalación de extensiones.

---

## 📋 Requisitos

- Sistema operativo Linux compatible (**Ubuntu 22.04+**, **Debian 12+** o derivadas).
- Arquitecturas soportadas: `x86_64` (`amd64`) y `aarch64` (`arm64`).
- Privilegios de superusuario (`sudo`).

---

## 🛠️ Instalación y Uso

### 1. Instalar `upd-agy-ide` como comando global

Descarga el script directamente en `/usr/local/bin` y otórgale permisos de ejecución con un solo comando:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/luffynando/upd-agy-ide/main/upd-agy-ide -o /usr/local/bin/upd-agy-ide && sudo chmod +x /usr/local/bin/upd-agy-ide
```

<details>
<summary>Alternativa: Clonar el repositorio con <code>git</code></summary>

```bash
git clone https://github.com/luffynando/upd-agy-ide.git
cd upd-agy-ide
sudo install -m 0755 upd-agy-ide /usr/local/bin/upd-agy-ide
```
</details>

### 2. Ejecutar el instalador / actualizador

Una vez instalado el comando global, puedes ejecutarlo desde cualquier carpeta de tu terminal:

```bash
sudo upd-agy-ide
```

Si deseas configurar el Marketplace de VS Code en `settings.json` (en lugar de Open VSX) y/o migrar tus extensiones compatibles desde VS Code, puedes usar las siguientes flags:

```bash
# Configurar el Marketplace oficial de VS Code en settings.json (si no existe):
sudo upd-agy-ide --vscode-marketplace

# Buscar y migrar únicamente las extensiones compatibles desde VS Code:
sudo upd-agy-ide --with-extensions

# Ejecutar ambas opciones en un solo paso:
sudo upd-agy-ide --vscode-marketplace --with-extensions
```

> **Nota**: El script requiere privilegios `root` (`sudo`) para instalar los binarios en `/opt`, configurar enlaces en `/usr/local/bin` y generar los accesos directos del sistema.

---

## 💻 Uso del comando `agy-ide`

Una vez completada la instalación, el comando `agy-ide` funcionará exactamente como `code` en VS Code:

| Comando | Descripción |
| :--- | :--- |
| `agy-ide .` | Abre la carpeta actual en Antigravity IDE y devuelve el control a la terminal. |
| `agy-ide archivo.txt` | Abre un archivo específico en el editor. |
| `agy-ide -r .` | Reutiliza la ventana activa para abrir la carpeta o proyecto. |
| `agy-ide -n` | Abre una nueva ventana en blanco de Antigravity IDE. |
| `agy-ide -d arch1 arch2` | Abre la herramienta de comparación (diff) entre dos archivos. |
| `agy-ide -g archivo:42` | Abre el archivo directamente en la línea 42. |
| `agy-ide --wait archivo` | Abre el archivo y espera a que se cierre (ideal como `core.editor` en Git). |
| `agy-ide -v` | Muestra la versión instalada de Antigravity IDE. |

> **Tip**: Si acabas de ejecutar el instalador y la terminal activa no reconoce el alias de inmediato, recarga la configuración de tu shell (`source ~/.bashrc` o `source ~/.zshrc`), o abre una nueva pestaña de terminal.

---

## 🔄 Funcionamiento interno de las actualizaciones

- Al volver a ejecutar `sudo upd-agy-ide`:
  1. Si detecta el script antiguo en `/usr/local/bin/update-antigravity-ide`, migra hacia `/usr/local/bin/upd-agy-ide` (si aún no existe) y elimina automáticamente el binario obsoleto.
  2. Consulta la API de versiones oficiales de Antigravity IDE y detecta si ya existe una versión previa instalada en `/opt/antigravity-ide` (reportando la versión instalada actual y la nueva disponible).
  3. Si la versión instalada es idéntica a la última disponible, el script informará que ya está actualizado, limpiará cualquier respaldo `.previous` remanente y no descargará nada nuevamente.
  4. Si hay una nueva versión disponible, notificará la actualización (`versión_actual -> nueva_versión`), descargará el tarball correspondiente, reemplazará la versión previa y autoeliminará el respaldo temporal (`.previous`) una vez completada exitosamente la instalación.
