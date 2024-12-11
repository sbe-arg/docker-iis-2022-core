# escape=`
FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022 

RUN powershell.exe -NoProfile -Command `
    New-Item -ItemType Directory -Force -Path 'C:\inetpub\logs'; `
    New-Item -ItemType Directory -Force -Path 'C:\logs'; `
    New-Item -ItemType Directory -Force -Path 'C:\temp'; `
    New-Item -ItemType Directory -Force -Path 'C:\pwsh'; `
    New-Item -ItemType Directory -Force -Path 'C:\LogMonitor'

RUN cmd /c icacls C:/inetpub/wwwroot /grant:r Everyone:F /t
RUN cmd /c icacls C:/logs /grant:r Everyone:F /t

WORKDIR /LogMonitor

COPY /logmon .

RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest -UseBasicParsing `
        -Uri "https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe" `
        -OutFile 'C:\LogMonitor\LogMonitor.exe'

RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest `
        -UseBasicParsing `
        -Uri https://github.com/PowerShell/PowerShell/releases/download/v7.4.6/PowerShell-7.4.6-win-x64.zip `
        -OutFile c:\temp\pwsh7.zip; `
    Expand-Archive -Path c:\temp\pwsh7.zip -DestinationPath c:\pwsh

WORKDIR /temp

RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest -UseBasicParsing `
        -Uri https://truststore.pki.rds.amazonaws.com/global/global-bundle.p7b `
        -OutFile global-bundle.p7b; `
    certutil -addstore 'Root' global-bundle.p7b

RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest -UseBasicParsing `
        -Uri http://download.microsoft.com/download/D/D/E/DDE57C26-C62C-4C59-A1BB-31D58B36ADA2/rewrite_amd64_en-US.msi `
        -OutFile rewrite_amd64_en-US.msi; `
    Start-Process rewrite_amd64_en-US.msi `
        -ArgumentList "/qn" -Wait
    
RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest `
        -UseBasicParsing `
        -Uri https://dot.net/v1/dotnet-install.ps1 `
        -OutFile dotnet-install.ps1; `
    ./dotnet-install.ps1 `
        -InstallDir '/Program Files/dotnet' `
        -Channel 9.0 `
        -Runtime dotnet; `
    && setx /M PATH "%PATH%;C:\Program Files\dotnet"

ENV DOTNET_RUNNING_IN_CONTAINER=true
ENV COMPLUS_NGenProtectedProcess_FeatureEnabled=0

RUN powershell.exe -NoProfile -Command `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET45 -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationDevelopment -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationInit -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-DefaultDocument -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HealthAndDiagnostics -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpCompressionStatic -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpErrors -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpLogging -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpRedirect -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpTracing -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIExtensions -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIFilter -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-LoggingLibraries -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-NetFxExtensibility45 -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestFiltering -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestMonitor -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-Security -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerManagementTools -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebSockets -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName NetFx3 -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName NetFx4Extended-ASPNET45 -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-StaticContent -All; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestMonitor -All

RUN cmd /c C:\Windows\system32\inetsrv\appcmd.exe unlock config -section:system.webServer/handlers
RUN cmd /c C:\Windows\system32\inetsrv\appcmd.exe set apppool /apppool.name:DefaultAppPool /enable32BitAppOnWin64:true

WORKDIR /inetpub/wwwroot

RUN powershell.exe -NoProfile -Command `
    Remove-Item -Recurse C:\temp\*; `
    Remove-Item -Recurse -Force C:\Windows\Temp\* ; `
    Remove-Item -Recurse -Force C:\Users\ContainerAdministrator\AppData\Local\Temp\* ; `
    && dism /online /cleanup-image /startcomponentcleanup

# RUN powershell.exe -NoProfile -Command Remove-Item -Recurse 'C:\inetpup\wwwroot\*'

# COPY /app .

ENTRYPOINT [ `
    "powershell.exe", `
    "-NoProfile", `
    "-ExecutionPolicy", "Bypass", `
    "-Command", `
    " Start-Process -FilePath 'C:\ServiceMonitor.exe'; Start-Process -FilePath 'C:\LogMonitor\LogMonitor.exe' -NoNewWindow -Wait" ]