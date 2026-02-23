@echo off
setlocal
title WUBlock - Windows Update Control Utility
mode con: cols=85 lines=35

:: ADMIN CHECK
net session >nul 2>&1
if %errorLevel% neq 0 (
    color 0C
    echo.
    echo   ############################################################
    echo   #                                                          #
    echo   #    [!!] ERROR: ADMINISTRATIVE PRIVILEGES REQUIRED [!!]   #
    echo   #                                                          #
    echo   #    Please right-click and "Run as administrator".        #
    echo   #                                                          #
    echo   ############################################################
    echo.
    pause
    exit /b 1
)

:MENU
cls
color 0A
echo.
echo   #     # #     # ######  #       #######  #####  #    #
echo   #  #  # #     # #     # #       #     # #     # #   # 
echo   #  #  # #     # #     # #       #     # #       #  #  
echo   #  #  # #     # ######  #       #     # #       ###   
echo   #  #  # #     # #     # #       #     # #       #  #  
echo   #  #  # #     # #     # #       #     # #     # #   # 
echo    ## ##   #####  ######  ####### #######  #####  #    #
echo   ============================================================
echo               Windows Update Control Utility v1
echo   ============================================================
echo.

:: Detect Current Status
set "U_STATUS=DISABLED"
sc query wuauserv | findstr /I "RUNNING" >nul 2>&1
if %errorLevel% == 0 set "U_STATUS=ENABLED"

echo     Current System Status: [%U_STATUS%]
echo.
echo   ------------------------------------------------------------
echo     [1]  DISABLE ALL UPDATES  (Hard Block)
echo     [2]  ENABLE ALL UPDATES   (Restore Default)
echo     [3]  RUN SAFETY CHECK     (Full Diagnostics)
echo     [4]  EXIT
echo   ------------------------------------------------------------
echo.
set /p CHOICE="  Selection > "

if "%CHOICE%"=="1" goto DISABLE
if "%CHOICE%"=="2" goto ENABLE
if "%CHOICE%"=="3" goto SAFETY_CHECK
if "%CHOICE%"=="4" exit /b 0
goto MENU

:DISABLE
cls
echo.
echo   [!] INITIATING HARD BLOCK...
echo   ------------------------------------------------------------
echo.
echo   [1/5] Stopping and Disabling Services...
sc stop wuauserv >nul 2>&1
sc config wuauserv start= disabled >nul 2>&1
sc stop UsoSvc >nul 2>&1
sc config UsoSvc start= disabled >nul 2>&1
sc stop WaaSMedicSvc >nul 2>&1
sc stop bits >nul 2>&1
sc config bits start= disabled >nul 2>&1
echo         Done.

echo   [2/5] Locking WaaSMedicSvc (Registry)...
reg add "HKLM\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc" /v "Start" /t REG_DWORD /d 4 /f >nul 2>&1
echo         Done.

echo   [3/5] Applying GPO Policy Blocks...
set "WUPOLICY=HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"
reg add "%WUPOLICY%" /v "DisableWindowsUpdateAccess" /t REG_DWORD /d 1 /f >nul 2>&1
reg add "%WUPOLICY%\AU" /v "NoAutoUpdate" /t REG_DWORD /d 1 /f >nul 2>&1
echo         Done.

echo   [4/5] Updating Local Hosts File...
set "H_PATH=%SystemRoot%\System32\drivers\etc\hosts"
attrib -r "%H_PATH%" >nul 2>&1
findstr /C:"WUBlock" "%H_PATH%" >nul 2>&1
if %errorLevel% neq 0 (
    echo. >> "%H_PATH%"
    echo # --- WUBlock Start --- >> "%H_PATH%"
    echo 0.0.0.0 windowsupdate.microsoft.com >> "%H_PATH%"
    echo 0.0.0.0 update.microsoft.com >> "%H_PATH%"
    echo # --- WUBlock End --- >> "%H_PATH%"
)
echo         Done.

echo   [5/5] Disabling Orchestrator Tasks...
schtasks /Change /TN "Microsoft\Windows\UpdateOrchestrator\Schedule Scan" /DISABLE >nul 2>&1
echo         Done.

echo.
echo   [SUCCESS] System updates are now blocked.
pause
goto SAFETY_CHECK_SILENT

:ENABLE
cls
echo.
echo   [!] RESTORING SYSTEM DEFAULTS...
echo   ------------------------------------------------------------
echo.
echo   [1/4] Setting Services to Auto...
sc config wuauserv start= auto >nul 2>&1
sc config UsoSvc start= auto >nul 2>&1
sc config bits start= delayed-auto >nul 2>&1
echo         Done.

echo   [2/4] Restoring Registry Policies...
reg add "HKLM\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc" /v "Start" /t REG_DWORD /d 3 /f >nul 2>&1
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /f >nul 2>&1
echo         Done.

echo   [3/4] Cleaning Hosts File...
findstr /V /C:"windowsupdate" /C:"update.microsoft" /C:"WUBlock" "%SystemRoot%\System32\drivers\etc\hosts" > "%SystemRoot%\System32\drivers\etc\hosts.tmp"
move /Y "%SystemRoot%\System32\drivers\etc\hosts.tmp" "%SystemRoot%\System32\drivers\etc\hosts" >nul 2>&1
echo         Done.

echo   [4/4] Re-enabling Tasks ^& Engine...
schtasks /Change /TN "Microsoft\Windows\UpdateOrchestrator\Schedule Scan" /ENABLE >nul 2>&1
sc start wuauserv >nul 2>&1
echo         Done.

echo.
echo   [SUCCESS] System updates are now restored.
pause
goto SAFETY_CHECK_SILENT

:SAFETY_CHECK
cls
:SAFETY_CHECK_SILENT
echo.
echo   ============================================================
echo                  DIAGNOSTIC SAFETY REPORT
echo   ============================================================
echo.

:: Service Status
sc query wuauserv | findstr /I "RUNNING" >nul 2>&1
if %errorLevel% == 0 (echo     [SERVICE]  Windows Update       : ACTIVE) else (echo     [SERVICE]  Windows Update       : BLOCKED)

:: Policy Status
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v "NoAutoUpdate" >nul 2>&1
if %errorLevel% == 0 (echo     [REGISTRY] Group Policy         : APPLIED) else (echo     [REGISTRY] Group Policy         : NONE)

:: Hosts Status
findstr /C:"windowsupdate.microsoft.com" "%SystemRoot%\System32\drivers\etc\hosts" >nul 2>&1
if %errorLevel% == 0 (echo     [NETWORK]  Update Servers       : NULL-ROUTED) else (echo     [NETWORK]  Update Servers       : OPEN)

:: Medic Status
reg query "HKLM\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc" /v "Start" | findstr "0x4" >nul 2>&1
if %errorLevel% == 0 (echo     [MEDIC]    WaaSMedicSvc         : LOCKED) else (echo     [MEDIC]    WaaSMedicSvc         : UNLOCKED)

echo.
echo   ============================================================
echo.
pause
goto MENU
