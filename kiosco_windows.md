# 🖥️ Kiosco en Windows 10 con Google Chrome (sin Assigned Access)

Configuración completa de un tótem usando:
- Auto-login
- Reemplazo de shell
- Google Chrome en modo kiosco
- Script .bat con restricciones de sistema

---

# 🧭 Objetivo

- Inicio automático con usuario `totem1`
- Lanzamiento directo de Chrome en modo kiosco
- Sin acceso funcional al escritorio
- Restricciones básicas del sistema
- Experiencia controlada tipo kiosco

---

# 1. Crear usuario kiosco

- Usuario: `totem1`
- Tipo: usuario estándar (NO administrador)

---

# 2. Configurar auto-login

Ejecutar:

Win + R → netplwiz

Pasos:

- Seleccionar usuario `totem1`
- Desmarcar:
  Los usuarios deben escribir su nombre y contraseña

---

# 3. Crear carpeta de kiosco

C:\ProgramData\kiosk

---

# 4. Script principal (Chrome en modo kiosco)

Crear archivo:

C:\ProgramData\kiosk\shell.bat

Contenido:

@echo off
taskkill /IM chrome.exe /F >nul 2>&1
timeout /t 1 >nul
start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --kiosk https://totem-boletos-la.netlify.app --no-first-run --disable-infobars --disable-session-crashed-bubble --overscroll-history-navigation=0 --disable-pinch --kiosk-printing

---

# 5. Reemplazar shell por el script

Abrir regedit:

HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Winlogon

Crear/modificar:

Shell = C:\ProgramData\kiosk\shell.bat

---

# 6. Script de restricciones del sistema

Crear archivo:

C:\ProgramData\kiosk\setup_kiosk.bat

Ejecutar como administrador

Contenido:

@echo off
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Explorer" /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\EdgeUI" /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoWinKeys /t REG_DWORD /d 1 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v NoRun /t REG_DWORD /d 1 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /v DisableTaskMgr /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\EdgeUI" /v AllowEdgeSwipe /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v DisableWebSearch /t REG_DWORD /d 1 /f

---

# 7. Aplicar restricciones al usuario correcto

- Iniciar sesión como `totem1`
- Ejecutar nuevamente el script si es necesario

---

# 8. Reiniciar sistema

shutdown /r /t 0

---

# ⚙️ Flujo de inicio

Inicio Windows → Auto-login → shell.bat → Chrome kiosco

---

# ⚠️ Limitaciones

- Puede haber un flash inicial
- Ctrl + Alt + Supr no se puede eliminar
- Algunas restricciones dependen del hardware

---

# 🧠 Conclusión

Configuración flexible de kiosco sin Assigned Access, con control de navegador e impresión silenciosa.
