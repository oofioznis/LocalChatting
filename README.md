Run this script for installation on Windows.

```
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

echo [3/4] Downloading Updater...
curl -L -# -o "%INSTALL_DIR%\updater.exe" "%GITHUB_BASE%/builds/updater/updater.exe"
if %ERRORLEVEL% neq 0 (
    echo ERROR: Failed to download updater.
        pause
            exit /b 1
            )

echo [4/4] Downloading App Build (!APP_VER!)...
set "APP_FILE=!APP_VER!.rar"
curl -L -# -o "%INSTALL_DIR%\!APP_FILE!" "%GITHUB_BASE%/builds/app/!APP_FILE!"

if %ERRORLEVEL% neq 0 (
    echo ERROR: Failed to download app build.
    pause
    exit /b 1
)

echo.
echo ========================================
echo   Installation Complete!
echo ========================================
echo Files are located in: %CD%\%INSTALL_DIR%
echo.
echo NOTE: Please extract both .rar files into the folder 
echo before running the application.
echo ========================================
pause
```