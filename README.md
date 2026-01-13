To install LocalChatting (Windows Only), create a .bat file in the folder you want to install the app in, and then copy the code below into said file, then run the file. 

```
@echo off
setlocal enabledelayedexpansion

echo ========================================
echo   LocalChatting Installer (Robust)
echo ========================================
echo.

REM Configuration
set "GITHUB_BASE=https://raw.githubusercontent.com/oofioznis/LocalChatting/main"
set "INSTALL_DIR=LocalChatting"

echo [1/4] Checking for latest versions...

REM Get latest app version
for /f "delims=" %%i in ('curl -s -f %GITHUB_BASE%/latestversion.txt') do set "APP_VER=%%i"
if "!APP_VER!"=="" (
    echo ERROR: Could not retrieve latest app version from GitHub.
    pause & exit /b 1
)

REM Get latest updater version
for /f "delims=" %%i in ('curl -s -f %GITHUB_BASE%/latestupdaterversion.txt') do set "UPDATER_VER=%%i"
if "!UPDATER_VER!"=="" (
    echo ERROR: Could not retrieve latest updater version from GitHub.
    pause & exit /b 1
)

echo Latest App Version: !APP_VER!
echo Latest Updater Version: !UPDATER_VER!
echo.

echo [2/4] Creating installation directory...
if not exist "%INSTALL_DIR%" mkdir "%INSTALL_DIR%"

set "UPDATER_EXTRACTED=false"
set "APP_EXTRACTED=false"

echo [3/4] Downloading Updater Build (!UPDATER_VER!)...
REM Try ZIP first
set "UPDATER_FILE=!UPDATER_VER!.zip"
set "UPDATER_PATH=%INSTALL_DIR%\!UPDATER_FILE!"

curl -L -f -# -o "!UPDATER_PATH!" "%GITHUB_BASE%/builds/updater/!UPDATER_FILE!"

if !ERRORLEVEL! equ 0 (
    REM Validate file size (if < 1KB, it's likely a broken download)
    for %%A in ("!UPDATER_PATH!") do if %%~zA LSS 1024 (
        echo WARNING: ZIP download seems corrupted (too small). Falling back...
        del "!UPDATER_PATH!"
        goto UPDATER_RAR
    )
    echo Extracting Updater...
    powershell -command "Expand-Archive -Path '!UPDATER_PATH!' -DestinationPath '%INSTALL_DIR%' -Force"
    if !ERRORLEVEL! equ 0 (
        del "!UPDATER_PATH!"
        set "UPDATER_EXTRACTED=true"
    ) else (
        echo WARNING: Auto-extraction failed. Please extract manually.
    )
) else (
    :UPDATER_RAR
    set "UPDATER_FILE=!UPDATER_VER!.rar"
    set "UPDATER_PATH=%INSTALL_DIR%\!UPDATER_FILE!"
    echo ZIP not found or failed, trying .rar...
    curl -L -f -# -o "!UPDATER_PATH!" "%GITHUB_BASE%/builds/updater/!UPDATER_FILE!"
    if !ERRORLEVEL! neq 0 (
        echo ERROR: Failed to download updater build (.zip or .rar).
        pause & exit /b 1
    )
)

echo.
echo [4/4] Downloading App Build (!APP_VER!)...
REM Try ZIP first
set "APP_FILE=!APP_VER!.zip"
set "APP_PATH=%INSTALL_DIR%\!APP_FILE!"

curl -L -f -# -o "!APP_PATH!" "%GITHUB_BASE%/builds/app/!APP_FILE!"

if !ERRORLEVEL! equ 0 (
    REM Validate file size
    for %%A in ("!APP_PATH!") do if %%~zA LSS 1024 (
        echo WARNING: ZIP download seems corrupted. Falling back...
        del "!APP_PATH!"
        goto APP_RAR
    )
    echo Extracting App...
    powershell -command "Expand-Archive -Path '!APP_PATH!' -DestinationPath '%INSTALL_DIR%' -Force"
    if !ERRORLEVEL! equ 0 (
        del "!APP_PATH!"
        set "APP_EXTRACTED=true"
    ) else (
        echo WARNING: Auto-extraction failed. Please extract manually.
    )
) else (
    :APP_RAR
    set "APP_FILE=!APP_VER!.rar"
    set "APP_PATH=%INSTALL_DIR%\!APP_FILE!"
    echo ZIP not found or failed, trying .rar...
    curl -L -f -# -o "!APP_PATH!" "%GITHUB_BASE%/builds/app/!APP_FILE!"
    if !ERRORLEVEL! neq 0 (
        echo ERROR: Failed to download app build (.zip or .rar).
        pause & exit /b 1
    )
)

echo.
echo ========================================
echo   Installation Complete!
echo ========================================
echo Files are located in: %CD%\%INSTALL_DIR%

REM Determine if extraction message is needed
if "!UPDATER_EXTRACTED!"=="false" (
    if "!APP_EXTRACTED!"=="false" (
        echo.
        echo NOTE: Please extract both !UPDATER_FILE! and !APP_FILE! 
        echo into the folder before running the application.
    ) else (
        echo.
        echo NOTE: Please extract !UPDATER_FILE! into the folder 
        echo before running the application.
    )
) else (
    if "!APP_EXTRACTED!"=="false" (
        echo.
        echo NOTE: Please extract !APP_FILE! into the folder 
        echo before running the application.
    )
)

echo ========================================
pause
```