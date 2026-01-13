To install LocalChatting (Windows Only), create a .bat file in the folder you want to install the app in, and then copy the code below into said file, then run the file. 

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

REM Get and trim latest app version
for /f "delims=" %%i in ('curl -s -f %GITHUB_BASE%/latestversion.txt') do (
    set "APP_VER=%%i"
    set "APP_VER=!APP_VER: =!"
)
REM Get and trim latest updater version
for /f "delims=" %%i in ('curl -s -f %GITHUB_BASE%/latestupdaterversion.txt') do (
    set "UPDATER_VER=%%i"
    set "UPDATER_VER=!UPDATER_VER: =!"
)

if "!APP_VER!"=="" ( echo ERROR: Could not retrieve App Version. & pause & exit /b 1 )
if "!UPDATER_VER!"=="" ( echo ERROR: Could not retrieve Updater Version. & pause & exit /b 1 )

echo Latest App Version: "!APP_VER!"
echo Latest Updater Version: "!UPDATER_VER!"
echo.

if not exist "%INSTALL_DIR%" mkdir "%INSTALL_DIR%"

set "UPDATER_EXTRACTED=false"
set "APP_EXTRACTED=false"

echo [2/4] Downloading and Extracting Updater...
set "U_ZIP_URL=%GITHUB_BASE%/builds/updater/!UPDATER_VER!.zip"
set "U_ZIP_PATH=%CD%\%INSTALL_DIR%\!UPDATER_VER!.zip"

REM Use PowerShell for both download and extraction to be safe
powershell -command "& { try { Invoke-WebRequest -Uri '%U_ZIP_URL%' -OutFile '%U_ZIP_PATH%' -ErrorAction Stop; Expand-Archive -Path '%U_ZIP_PATH%' -DestinationPath '%INSTALL_DIR%' -Force; Remove-Item '%U_ZIP_PATH%'; exit 0 } catch { exit 1 } }"

if !ERRORLEVEL! equ 0 (
    echo Updater extracted successfully.
    set "UPDATER_EXTRACTED=true"
) else (
    echo ZIP not found or extraction failed, trying .rar...
    set "UPDATER_FILE=!UPDATER_VER!.rar"
    curl -L -f -# -o "%INSTALL_DIR%\!UPDATER_FILE!" "%GITHUB_BASE%/builds/updater/!UPDATER_FILE!"
    if !ERRORLEVEL! neq 0 (
        echo ERROR: Failed to download updater build.
        pause & exit /b 1
    )
)

echo.
echo [3/4] Downloading and Extracting App...
set "A_ZIP_URL=%GITHUB_BASE%/builds/app/!APP_VER!.zip"
set "A_ZIP_PATH=%CD%\%INSTALL_DIR%\!APP_VER!.zip"

powershell -command "& { try { Invoke-WebRequest -Uri '%A_ZIP_URL%' -OutFile '%A_ZIP_PATH%' -ErrorAction Stop; Expand-Archive -Path '%A_ZIP_PATH%' -DestinationPath '%INSTALL_DIR%' -Force; Remove-Item '%A_ZIP_PATH%'; exit 0 } catch { exit 1 } }"

if !ERRORLEVEL! equ 0 (
    echo App extracted successfully.
    set "APP_EXTRACTED=true"
) else (
    echo ZIP not found or extraction failed, trying .rar...
    set "APP_FILE=!APP_VER!.rar"
    curl -L -f -# -o "%INSTALL_DIR%\!APP_FILE!" "%GITHUB_BASE%/builds/app/!APP_FILE!"
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

if "!UPDATER_EXTRACTED!"=="false" (
    if "!APP_EXTRACTED!"=="false" (
        echo.
        echo NOTE: Please extract both versions (!UPDATER_VER!.rar and !APP_VER!.rar) 
        echo into the folder before running the application.
    ) else (
        echo.
        echo NOTE: Please extract !UPDATER_VER!.rar into the folder.
    )
) else (
    if "!APP_EXTRACTED!"=="false" (
        echo.
        echo NOTE: Please extract !APP_VER!.rar into the folder.
    )
)

echo ========================================
pause
```