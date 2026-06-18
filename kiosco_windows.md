# 🖥️ Windows 10 Kiosco con Google Chrome (sin Assigned Access)

Configuración de kiosco básico usando:

- Auto-login
- Reemplazo de shell
- Google Chrome en modo kiosco
- Restricciones básicas
- Reinicio automático del navegador

---

# 🧭 Objetivo

El objetivo de esta configuración es:

- realizar login automático con el usuario `totem1`
- abrir Chrome directamente en modo kiosco
- ocultar el escritorio tradicional
- reiniciar Chrome automáticamente si se cierra
- aplicar restricciones básicas del sistema

---

# 1. Crear usuario kiosco

Ir a:

Configuración → Cuentas → Familia y otros usuarios → Agregar otra persona a este equipo

Crear:

- Usuario: `totem1`
- Tipo: Usuario estándar (NO administrador)

---

# 2. Configurar auto-login

Abrir:

Win + R → `netplwiz`

Pasos:

1. Seleccionar usuario `totem1`
2. Desmarcar:

```text
Los usuarios deben escribir su nombre y contraseña para usar el equipo
```

3. Aplicar
4. Ingresar contraseña del usuario

---

# ⚠️ Si la opción no aparece en netplwiz

Algunas versiones de Windows ocultan la opción.

## Solución

Abrir:

Win + R → `regedit`

Ir a:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device
```

Modificar:

```text
DevicePasswordLessBuildVersion = 0
```

Luego:

- reiniciar
o
- cerrar sesión

---

# 3. Crear carpeta de kiosco

Iniciar sesión como `totem1`

Crear:

```text
C:\ProgramData\kiosk
```

---

# 4. Crear script principal del kiosco

Crear archivo:

```text
C:\ProgramData\kiosk\shell.bat
```

Contenido:

```bat
@echo off
REM =========================================
REM ASEGURAR ANYDESK
REM =========================================
sc start AnyDesk >nul 2>&1

REM =========================================
REM INICIAR MONITOR DE HARDWARE LOCAL
REM =========================================
REM Ejecuta el monitor en segundo plano sin abrir ventanas negras
start /b "" "C:\ProgramData\kiosk\totem-hardware.exe"

REM =========================================
REM INICIAR TECLADO VISUAL (COMUNICACIÓN CON FRONTEND)
REM =========================================
REM Arranca la aplicación principal del teclado numérico
start "" "C:\ProgramData\kiosk\TecladoVisual.exe"

REM =========================================
REM LOOP KIOSKO
REM =========================================
:loop
taskkill /IM chrome.exe /F >nul 2>&1
timeout /t 1 >nul

start /wait "" "C:\Program Files\Google\Chrome\Application\chrome.exe" ^
--kiosk https://totem-boletos-la.netlify.app ^
--no-first-run ^
--disable-session-crashed-bubble ^
--overscroll-history-navigation=0 ^
--disable-pinch ^
--kiosk-printing

timeout /t 2 >nul
goto loop
```

---

# 🧠 Función del script

El script:

- elimina instancias previas de Chrome
- abre Chrome en modo kiosco
- espera hasta que Chrome se cierre
- reinicia Chrome automáticamente

Esto ayuda a evitar:

- pantalla negra
- cierre accidental del kiosco
- sesiones inactivas tras cerrar Chrome

---

# 5. Configurar reemplazo de shell

## ⚠️ Importante

NO modificar:

sin comprender completamente el impacto.

Modificar `HKLM` afecta TODOS los usuarios del sistema.

---

## Método recomendado (por usuario)

Iniciar sesión como:

```text
totem1
```

Abrir:

Win + R → `regedit`

Ir a:

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

---

## Si la clave `Winlogon` no existe

Crear:

- clic derecho sobre `CurrentVersion`
- Nuevo
- Clave
- Nombre:

```text
Winlogon
```

---

## Crear valor Shell

Dentro de `Winlogon`:

- clic derecho en panel derecho
- Nuevo
- Valor de cadena

Nombre exacto:

```text
Shell
```

Valor:

```text
C:\ProgramData\kiosk\shell.bat
```

---

# 6. Crear script de restricciones

Crear archivo:

```text
C:\ProgramData\kiosk\setup_kiosk.bat
```

Contenido:

```bat
@echo off

REM =========================
REM CREAR CLAVES NECESARIAS
REM =========================

reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\EdgeUI" /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /f

REM =========================
REM RESTRICCIONES BASICAS
REM =========================

reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" ^
/v NoWinKeys /t REG_DWORD /d 1 /f

reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" ^
/v NoRun /t REG_DWORD /d 1 /f

reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" ^
/v DisableTaskMgr /t REG_DWORD /d 1 /f

reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\EdgeUI" ^
/v AllowEdgeSwipe /t REG_DWORD /d 0 /f

reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" ^
/v DisableWebSearch /t REG_DWORD /d 1 /f

REM =========================
REM ANYDESK
REM =========================

sc config AnyDesk start= auto
sc start AnyDesk

REM =========================
REM FIREWALL ANYDESK
REM =========================

netsh advfirewall firewall add rule ^
name="AnyDesk TCP" ^
dir=in action=allow ^
protocol=TCP localport=7070

REM =========================
REM EVITAR SUSPENSION
REM =========================

powercfg /change standby-timeout-ac 0
powercfg /change monitor-timeout-ac 0

REM =========================
REM FINAL
REM =========================

echo.
echo Kiosko configurado correctamente.
echo AnyDesk deberia permanecer accesible remotamente.
pause
```

Ejecutar:

- clic derecho
- Ejecutar como administrador

---

# 7. Reiniciar sistema

Ejecutar:

```cmd
shutdown /r /t 0
```

---

# 🔄 Flujo de inicio

```text
Windows → Auto-login → shell.bat → Chrome kiosco
```

---

# 🔧 Recuperación de emergencia

Si el kiosco falla y aparece:

- pantalla negra
- login roto
- Chrome no inicia

Abrir administrador de tareas:

```text
Ctrl + Alt + Supr
```

Luego:

Archivo → Ejecutar nueva tarea

Ejecutar:

```text
regedit
```

Ir a:

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

Modificar:

```text
Shell = explorer.exe
```

Cerrar sesión o reiniciar.

---

# ⚠️ Limitaciones reales

Esta configuración NO bloquea completamente:

- Ctrl + Alt + Supr
- Alt + Tab
- algunas teclas Windows
- accesibilidad
- diálogos del sistema
- herramientas administrativas
- posibles escapes desde Chrome

---

# ✅ Conclusión

Esta configuración resulta adecuada para:

- tótems simples
- kioscos informativos
- pantallas interactivas básicas
- entornos controlados
