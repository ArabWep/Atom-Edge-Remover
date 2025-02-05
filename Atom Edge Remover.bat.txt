@echo off
title Atom Edge Remover - Full Microsoft Edge Removal
cls
echo ======================================================
echo                ATOM EDGE REMOVER
echo     Completely Removing Microsoft Edge & Updates
echo ======================================================
echo.

echo [*] Stopping Edge-related processes...
taskkill /F /IM msedge.exe >nul 2>&1
taskkill /F /IM edgeupdate.exe >nul 2>&1
taskkill /F /IM edgeupdatem.exe >nul 2>&1

echo [*] Uninstalling Microsoft Edge...
:: Uninstall Edge Stable
if exist "C:\Program Files (x86)\Microsoft\Edge\Application\*" (
    cd /d "C:\Program Files (x86)\Microsoft\Edge\Application\*\Installer"
    setup.exe --uninstall --system-level --verbose-logging --force-uninstall
)
:: Uninstall Edge Beta
if exist "C:\Program Files (x86)\Microsoft\Edge Beta\Application\*" (
    cd /d "C:\Program Files (x86)\Microsoft\Edge Beta\Application\*\Installer"
    setup.exe --uninstall --system-level --verbose-logging --force-uninstall
)

echo [*] Removing leftover Edge files and folders...
rd /s /q "C:\Program Files (x86)\Microsoft\Edge" >nul 2>&1
rd /s /q "C:\Program Files (x86)\Microsoft\Edge Beta" >nul 2>&1
rd /s /q "C:\ProgramData\Microsoft\Edge" >nul 2>&1
rd /s /q "%LOCALAPPDATA%\Microsoft\Edge" >nul 2>&1
rd /s /q "%LOCALAPPDATA%\Microsoft\EdgeUpdate" >nul 2>&1

echo [*] Disabling Edge-related scheduled tasks...
schtasks /delete /tn "\Microsoft\Edge\*" /f >nul 2>&1
schtasks /delete /tn "\Microsoft\EdgeUpdate\*" /f >nul 2>&1

echo [*] Disabling Edge services...
sc config edgeupdate start= disabled >nul 2>&1
sc config edgeupdatem start= disabled >nul 2>&1
sc stop edgeupdate >nul 2>&1
sc stop edgeupdatem >nul 2>&1

echo [*] Blocking Microsoft Edge reinstallation via Registry...
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\EdgeUpdate" /v "DoNotUpdateToEdgeWithChromium" /t REG_DWORD /d 1 /f >nul 2>&1
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Edge" /v "UpdateDefault" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\EdgeUpdate" /v "UpdateDefault" /t REG_DWORD /d 0 /f >nul 2>&1
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Microsoft Edge" /v "NoRemove" /t REG_DWORD /d 1 /f >nul 2>&1

echo [*] Blocking Microsoft Edge from Windows Updates...
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate" /v "DoNotAllowEdgeUpdate" /t REG_DWORD /d 1 /f >nul 2>&1
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\EdgeUpdate" /v "NoRemove" /t REG_DWORD /d 1 /f >nul 2>&1

echo [*] Checking if Edge is still installed...
where msedge >nul 2>&1
if %errorlevel%==0 (
    echo [!] Edge removal failed. Please restart and run this script again.
) else (
    echo [✓] Microsoft Edge has been successfully removed!
)

echo.
echo [*] Microsoft Edge removal process complete.
echo ======================================================
echo.
pause
exit
