# escape=`
FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2022 

RUN powershell.exe -NoProfile -Command `
    New-Item -ItemType Directory -Force -Path 'C:\inetpub\logs'; `
    New-Item -ItemType Directory -Force -Path 'C:\logs'; `
    New-Item -ItemType Directory -Force -Path 'C:\temp'

RUN cmd /c icacls C:/inetpub/wwwroot /grant:r Everyone:F /t
RUN cmd /c icacls C:/logs /grant:r Everyone:F /t

COPY /logmon .

RUN powershell.exe -NoProfile -Command `
    Invoke-WebRequest -UseBasicParsing `
        -Uri "https://github.com/microsoft/windows-container-tools/releases/download/v2.1.1/LogMonitor.exe" `
        -OutFile 'C:\LogMonitor.exe'

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
    Invoke-WebRequest `
        -UseBasicParsing `
        -Uri https://dotnetbinaries.blob.core.windows.net/dockerassets/microsoft-windows-netfx3-ltsc2022.zip `
        -OutFile netfx3.zip; `
    Expand-Archive -Path netfx3.zip -DestinationPath .; `
    $cabFile = Get-ChildItem -Path . -Filter "*.cab" -Recurse | Select-Object -First 1; `
    Enable-WindowsOptionalFeature -Online -FeatureName "NetFx3" -Source $cabFile.FullName


RUN powershell.exe -NoProfile -Command `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-StaticContent; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-DefaultDocument; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpErrors; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpRedirect; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpLogging; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-LoggingLibraries; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpTracing; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpCompressionStatic; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-Security; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestFiltering; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationDevelopment; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET45; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-NetFxExtensibility45; `
    Enable-WindowsOptionalFeature -Online -FeatureName NetFx4Extended-ASPNET45; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerManagementTools; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationInit; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIExtensions; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIFilter; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebSockets; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-HealthAndDiagnostics; `
    Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestMonitor

RUN cmd /c C:\Windows\system32\inetsrv\appcmd.exe unlock config -section:system.webServer/handlers
RUN cmd /c C:\Windows\system32\inetsrv\appcmd.exe set apppool /apppool.name:DefaultAppPool /enable32BitAppOnWin64:true

WORKDIR /inetpub/wwwroot

RUN powershell.exe -NoProfile -Command Remove-Item -Recurse 'C:\temp'

# RUN powershell.exe -NoProfile -Command Remove-Item -Recurse 'C:\inetpup\wwwroot\*'

# COPY /app .

ENTRYPOINT [ `
    "powershell.exe", `
    "-NoProfile", `
    "-ExecutionPolicy", "Bypass", `
    "-Command", `
    " `
    Start-Process -FilePath 'C:\ServiceMonitor.exe'; `
    Start-Process -FilePath 'C:\LogMonitor.exe' -NoNewWindow -Wait `
    " `
]