@echo off
setlocal enabledelayedexpansion

echo ========================================
echo   LocalChatting Installer
echo ========================================
echo.

REM Configuration
set "GITHUB_BASE=https://raw.githubusercontent.com/oofioznis/LocalChatting/main"
set "INSTALL_DIR=LocalChatting"

echo [1/4] Checking for latest versions...

REM Get latest app version
for /f "delims=" %%i in ('curl -s %GITHUB_BASE%/latestversion.txt') do set "APP_VER=%%i"
if "!APP_VER!"=="" (
    echo ERROR: Could not retrieve latest app version.
    pause
    exit /b 1
)

REM Get latest updater version
for /f "delims=" %%i in ('curl -s %GITHUB_BASE%/latestupdaterversion.txt') do set "UPDATER_VER=%%i"
if "!UPDATER_VER!"=="" (
    echo ERROR: Could not retrieve latest updater version.
    pause
    exit /b 1
)

echo Latest App Version: !APP_VER!
echo Latest Updater Version: !UPDATER_VER!
echo.

echo [2/4] Creating installation directory...
if not exist "%INSTALL_DIR%" mkdir "%INSTALL_DIR%"

set "UPDATER_EXTRACTED=false"
set "APP_EXTRACTED=false"

echo [3/4] Downloading Updater Build (!UPDATER_VER!)...
REM Try ZIP first for auto-extraction
set "UPDATER_FILE=!UPDATER_VER!.zip"
curl -L -f -s -o "%INSTALL_DIR%\!UPDATER_FILE!" "%GITHUB_BASE%/builds/updater/!UPDATER_FILE!"

if %ERRORLEVEL% equ 0 (
    echo Extracting Updater...
    powershell -command "Expand-Archive -Path '%INSTALL_DIR%\!UPDATER_FILE!' -DestinationPath '%INSTALL_DIR%' -Force"
    del "%INSTALL_DIR%\!UPDATER_FILE!"
    set "UPDATER_EXTRACTED=true"
) else (
    REM Fallback to RAR
    set "UPDATER_FILE=!UPDATER_VER!.rar"
    echo ZIP not found, trying .rar...
    curl -L -# -o "%INSTALL_DIR%\!UPDATER_FILE!" "%GITHUB_BASE%/builds/updater/!UPDATER_FILE!"
    if !ERRORLEVEL! neq 0 (
        echo ERROR: Failed to download updater build.
        pause & exit /b 1
    )
)

echo [4/4] Downloading App Build (!APP_VER!)...
REM Try ZIP first
set "APP_FILE=!APP_VER!.zip"
curl -L -f -s -o "%INSTALL_DIR%\!APP_FILE!" "%GITHUB_BASE%/builds/app/!APP_FILE!"

if %ERRORLEVEL% equ 0 (
    echo Extracting App...
    powershell -command "Expand-Archive -Path '%INSTALL_DIR%\!APP_FILE!' -DestinationPath '%INSTALL_DIR%' -Force"
    del "%INSTALL_DIR%\!APP_FILE!"
    set "APP_EXTRACTED=true"
) else (
    REM Fallback to RAR
    set "APP_FILE=!APP_VER!.rar"
    echo ZIP not found, trying .rar...
    curl -L -# -o "%INSTALL_DIR%\!APP_FILE!" "%GITHUB_BASE%/builds/app/!APP_FILE!"
    if !ERRORLEVEL! neq 0 (
        echo ERROR: Failed to download app build.
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