# Kiosco Windows 10 (sin Acceso Asignado) -- Configuración completa

Configuración de tótem en Windows 10 usando auto-login +
`shell:startup` + script `.bat` + Microsoft Edge en modo kiosco.

------------------------------------------------------------------------

# 🧭 Objetivo

-   Inicio automático de usuario `totem1`
-   Apertura automática de Microsoft Edge en modo kiosco
-   Sin acceso al escritorio funcional
-   Impresión directa sin diálogo
-   Bloqueo de gestos laterales y atajos del sistema

------------------------------------------------------------------------

# 1. Crear usuario kiosco

-   Usuario: `totem1`
-   Tipo: usuario estándar (no administrador)

------------------------------------------------------------------------

# 2. Configurar auto-login

Ejecutar:

Win + R → netplwiz

Pasos: - Seleccionar usuario `totem1` - Desactivar opción:\
"Los usuarios deben escribir su nombre y contraseña"

------------------------------------------------------------------------

# 3. Script de inicio (.bat)

Crear carpeta:

C:`\ProgramData\kiosk`

Crear archivo:

C:`\ProgramData\kiosk\kiosk.bat`

Contenido:

@echo off taskkill /IM msedge.exe /F \>nul 2\>&1 timeout /t 1 \>nul
start "" "C:\Program Files
(x86)\Microsoft\Edge\Application\msedge.exe"
--kiosk https://totem-boletos-la.netlify.app
--edge-kiosk-type=fullscreen --no-first-run --disable-infobars
--kiosk-printing

------------------------------------------------------------------------

# 4. Inicio automático (shell:startup)

Iniciar sesión como `totem1`:

Win + R → shell:startup

Agregar acceso directo a:

C:`\ProgramData\kiosk\kiosk.bat`

------------------------------------------------------------------------

# 5. Políticas de bloqueo (GPO)

Aplicar a usuario `totem1`.

## 5.1 Desactivar teclas de Windows

Ruta:

Configuración de usuario → Plantillas administrativas → Explorador de
archivos

Política: - Desactivar teclas de método abreviado de Windows

------------------------------------------------------------------------

## 5.2 Desactivar Administrador de tareas

Configuración de usuario → Sistema → Ctrl+Alt+Supr

Política: - Quitar Administrador de tareas

------------------------------------------------------------------------

## 5.3 Quitar Ejecutar

Menú Inicio y barra de tareas

Política: - Quitar menú Ejecutar

------------------------------------------------------------------------

# 6. Bloqueo de deslizamiento lateral

Ruta:

Configuración del equipo → Plantillas administrativas → Componentes de
Windows → Interfaz de usuario perimetral

Política: - Permitir deslizamientos desde el borde → Deshabilitada

------------------------------------------------------------------------

# 7. Barra de tareas

En sesión `totem1`:

-   Activar:
    -   Ocultar automáticamente la barra de tareas

------------------------------------------------------------------------

# ⚙️ Flujo de inicio

1.  Windows inicia\
2.  Auto-login en `totem1`\
3.  Explorer carga brevemente\
4.  `shell:startup` ejecuta `kiosk.bat`\
5.  Edge se cierra/reinicia si es necesario\
6.  Edge abre en modo kiosco\
7.  Sistema queda bloqueado en la aplicación web

------------------------------------------------------------------------

# ⚠️ Limitaciones

-   Puede existir un flash inicial del escritorio\
-   No es un kiosco embebido real tipo sistema dedicado\
-   Depende de estabilidad de Microsoft Edge\
-   Si Edge falla, el sistema queda sin interfaz funcional

------------------------------------------------------------------------

# 🧠 Conclusión

Este enfoque:

-   No usa Acceso Asignado\
-   Permite control total del navegador\
-   Habilita impresión directa (`--kiosk-printing`)\
-   Reduce interacción del sistema operativo\
-   Es adecuado para tótems funcionales de producción básica
