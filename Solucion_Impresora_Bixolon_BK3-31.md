# Documentación Técnica: Solución de Error "No se puede abrir el puerto" en Impresora BIXOLON BK3-31 (Samsung Kiosk)

Esta guía documenta el procedimiento paso a paso realizado para solucionar el error crítico "No se puede abrir el puerto" en terminales Samsung Kiosk equipadas con la ticketera térmica interna BIXOLON BK3-31.

---

## 1. Diagnóstico del Problema Original
Al intentar interactuar con la impresora o abrir utilidades de diagnóstico, el sistema operativo Windows arrojaba el error de bloqueo de puerto. 

### Causas identificadas de forma remota:
1. **Secuestro de Puerto (Software Conflict):** El software complementario *BIXOLON Virtual Port Driver* emulaba un puerto virtual artificial (COM6) que creaba un bucle lógico e interceptaba el canal físico real.
2. **Desalineación de Parámetros Seriales:** La paridad interna de la impresora operaba en None (Ninguna), mientras que las utilidades o aplicaciones intentaban forzar la apertura del bus bajo una paridad Even (Par), bloqueando inmediatamente el buffer de hardware.
3. **Desplazamiento del Puerto Físico:** Windows reasignó dinámicamente la interfaz real al puerto COM4, dejando inoperantes las configuraciones heredadas orientadas a otros puertos.

---

## 2. Fase 1: Purga Absoluta del Entorno de Impresión (Empezar desde 0)

### Paso 1.1: Detención del Spooler y Purga de Archivos Temporales

```powershell
# Detener el servicio de cola de impresión
Start-Service -Name "Spooler" -ErrorAction SilentlyContinue

# Buscar y remover la impresora lógica antigua vinculada a Bixolon
$Printer = Get-Printer | Where-Object {$_.Name -like "*BIXOLON*"}
if ($Printer) {
    Remove-Printer -Name $Printer.Name
    Write-Host "-> Impresora '$($Printer.Name)' eliminada con éxito." -ForegroundColor Green
}

# Remover el paquete de drivers residentes de la base de datos de Windows
$Driver = Get-PrinterDriver | Where-Object {$_.Name -like "*BIXOLON*"}
if ($Driver) {
    Remove-PrinterDriver -Name $Driver.Name
    Write-Host "-> Driver '$($Driver.Name)' removido del almacén." -ForegroundColor Green
}

# Detener el Spooler para limpiar el almacenamiento físico en disco
Stop-Service -Name "Spooler" -Force
$SpoolPath = "$env:SystemRoot\System32\spool\PRINTERS\*"
if (Test-Path $SpoolPath) {
    Remove-Item -Path $SpoolPath -Recurse -Force -ErrorAction SilentlyContinue
}

# Reactivar el servicio limpio
Start-Service -Name "Spooler"
```

### Paso 1.2: Purga de Controladores de Terceros (Archivos INF) mediante PnPUtil

```powershell
$AllDrivers = pnputil /enum-drivers
$InfFiles = $AllDrivers | Select-String -Pattern "oem\d+\.inf" | ForEach-Object { $_.Matches.Value } | Unique

if ($InfFiles) {
    foreach ($inf in $InfFiles) {
        $DriverCheck = pnputil /enum-drivers /driver $inf
        if ($DriverCheck -match "BIXOLON") {
            Write-Host "-> Eliminando paquete Bixolon: $inf" -ForegroundColor Cyan
            pnputil /delete-driver $inf /force /uninstall | Out-Null
        }
    }
    Write-Host "Purga de drivers INF finalizada con éxito." -ForegroundColor Green
}
```

### Paso 1.3: Limpieza del Mapa de Registros de Hardware y Restablecimiento del Bus USB

```powershell
Remove-ItemProperty -Path "HKLM:\HARDWARE\DEVICEMAP\SERIALCOMM" -Name "\Device\BixolonVSerial0" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKLM:\HARDWARE\DEVICEMAP\SERIALCOMM" -Name "\Device\BSerial0" -ErrorAction SilentlyContinue

$CypressDevices = Get-CimInstance Win32_PnPEntity | Where-Object {$_.DeviceID -like "*VID_04B4&PID_00FB*"}
foreach ($Device in $CypressDevices) {
    pnputil /disable-device $Device.DeviceID /force | Out-Null
    Start-Sleep -Seconds 2
    pnputil /enable-device $Device.DeviceID /force | Out-Null
}
```

## 3. Fase 2: Descubrimiento y Mapeo del Puerto Físico Real

```powershell
mode COM4: BAUD=9600 PARITY=N DATA=8 STOP=1
```

## 4. Fase 3: Prueba de Impresión en Crudo (Sin Drivers)

```powershell
$port = New-Object System.IO.Ports.SerialPort COM4, 9600, None, 8, one
$port.Open()

$port.Write("====================================`r`n")
$port.Write("   TEST DE IMPRESION DIRECTA COM4   `r`n")
$port.Write("====================================`r`n")
$port.Write("Kiosco Desbloqueado Exitosamente`r`n")

$port.Close()
```

## 5. Fase 4: Reinstalación Limpia del Driver de Windows

- Descargar Driver: Software POS Windows Driver V5.2.2
- Evitar: BXLVCOM4USB
- Interface: Serial (COM Port)
- Puerto: COM4
- Baud Rate: 9600
- Parity: None
- Data Bits: 8
- Stop Bits: 1
