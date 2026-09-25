<#
.SYNOPSIS
    Enterprise Post-Installation & App Lifecycle Engine for Windows 10/11 & LTSC.
.DESCRIPTION
    Detects installed apps, missing apps, and apps needing update.
    Shows [OK], [INSTALL], and [UPDATE], and waits for approval before making changes.
#>

[CmdletBinding()]
param(
    [switch]$AutoApprove,
    [switch]$SelectPackages
)

# Project by: Abu Kasim
# GitHub: https://github.com/abukasim0
# Telegram: https://t.me/programs_edditing

$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls13

if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Host "[!] Elevating privileges to Administrator..." -ForegroundColor Yellow
    Start-Process powershell.exe -ArgumentList @('-NoProfile', '-ExecutionPolicy', 'Bypass', '-File', $PSCommandPath) -Verb RunAs
    exit 0
}

Clear-Host
Write-Host "=====================================================" -ForegroundColor Cyan
Write-Host "      Windows Post-Installation Automated Setup      " -ForegroundColor Cyan
Write-Host "        Project by: Abu Kasim                       " -ForegroundColor Magenta
Write-Host "        GitHub: https://github.com/abukasim0          " -ForegroundColor Magenta
Write-Host "        Telegram: https://t.me/programs_edditing     " -ForegroundColor Magenta
Write-Host "=====================================================" -ForegroundColor Cyan

Write-Host "`n[+] Verifying Package Manager engine..." -ForegroundColor Cyan

function Install-WingetBootstrap {
    Write-Host "[*] WinGet not found. Bootstrapping it..." -ForegroundColor Yellow
    $ProgressPreference = 'SilentlyContinue'
    $TempDir = "$env:TEMP\WinGetBootstrap"
    New-Item -ItemType Directory -Path $TempDir -Force | Out-Null

    try {
        $BundlePath = Join-Path $TempDir 'Microsoft.DesktopAppInstaller.msixbundle'
        $DownloadUrl = 'https://aka.ms/getwinget'

        Invoke-WebRequest -Uri $DownloadUrl -OutFile $BundlePath -UseBasicParsing
        Add-AppxPackage -Path $BundlePath -ForceApplicationShutdown

        if (Get-Command winget -ErrorAction SilentlyContinue) {
            Write-Host "[OK] WinGet installed successfully." -ForegroundColor Green
            return $true
        }

        throw 'WinGet is still unavailable after installation.'
    }
    catch {
        Write-Host "[X] Automatic WinGet bootstrap failed: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "    Please install App Installer manually from Microsoft Store or visit: https://aka.ms/getwinget" -ForegroundColor Yellow
        return $false
    }
    finally {
        Remove-Item -Path $TempDir -Recurse -Force -ErrorAction SilentlyContinue
    }
}

if (-not (Get-Command winget -ErrorAction SilentlyContinue)) {
    if (-not (Install-WingetBootstrap)) {
        exit 1
    }
}
else {
    Write-Host "[OK] WinGet is operational." -ForegroundColor Green
}

try {
    winget source update --disable-interactivity | Out-Null
    if ($LASTEXITCODE -ne 0) {
        throw "winget source update exited with code $LASTEXITCODE"
    }
}
catch {
    Write-Host "[!] WinGet source update failed: $($_.Exception.Message)" -ForegroundColor Yellow
    Write-Host "    Continuing with the current cached package metadata..." -ForegroundColor Yellow
}

$SystemRuntimes = @(
    'Microsoft.AppInstaller',
    'Microsoft.DotNet.DesktopRuntime.8',
    'Microsoft.DotNet.DesktopRuntime.9',
    'Microsoft.DotNet.Native.Runtime',
    'Microsoft.DirectX',
    'Microsoft.WindowsTerminal',
    'Microsoft.EdgeWebView2Runtime',
    'Microsoft.UI.Xaml.2.8',
    'Microsoft.WindowsAppRuntime.1.5',
    'Microsoft.WindowsAppRuntime.1.6',
    'Microsoft.WindowsAppRuntime.1.8',
    'Microsoft.WindowsAppRuntime.2',
    'Microsoft.VCLibs.14',
    'Microsoft.VCLibs.Desktop.14',
    'Microsoft.VCRedist.2005.x86',
    'Microsoft.VCRedist.2008.x64',
    'Microsoft.VCRedist.2008.x86',
    'Microsoft.VCRedist.2010.x86',
    'Microsoft.VCRedist.2012.x64',
    'Microsoft.VCRedist.2012.x86',
    'Microsoft.VCRedist.2013.x64',
    'Microsoft.VCRedist.2013.x86',
    'Microsoft.VCRedist.2015+.x64',
    'Microsoft.VCRedist.2015+.x86',
    'Microsoft.VSTOR',
    'Microsoft.XNARedist',
    'Python.Python.3.13',
    'Python.Launcher'
) | Sort-Object -Unique

$EssentialApps = @(
    '7zip.7zip',
    'Brave.Brave',
    'CodecGuide.K-LiteCodecPack.Mega',
    'Flow-Launcher.Flow-Launcher',
    'Google.Chrome',
    'GuoJikun.QuickLook',
    'JanDeDobbeleer.OhMyPosh',
    'Microsoft.Edge',
    'Microsoft.PowerShell',
    'Microsoft.VisualStudioCode',
    'Microsoft.WindowsTerminal',
    'Notepad++.Notepad++',
    'OpenWhisperSystems.Signal',
    'qBittorrent.qBittorrent',
    'Python.Python.3.13',
    'RARLab.WinRAR',
    'Telegram.TelegramDesktop',
    'VideoLAN.VLC'
) | Sort-Object -Unique

$OptionalApps = @(
    'Apple.iTunes',
    'Apple.QuickTime',
    'CreativeTechnology.OpenAL',
    'Discord.Discord',
    'erez-c137.NetSpeedTray',
    'WhatsApp.WhatsApp'
) | Sort-Object -Unique

$MicrosoftApps = @(
    'Microsoft.AppInstaller',
    'Microsoft.Edge',
    'Microsoft.EdgeWebView2Runtime',
    'Microsoft.PowerShell',
    'Microsoft.VisualStudioCode',
    'Microsoft.WindowsAppRuntime.1.5',
    'Microsoft.WindowsAppRuntime.1.6',
    'Microsoft.WindowsAppRuntime.1.8',
    'Microsoft.WindowsAppRuntime.2',
    'Microsoft.WindowsTerminal'
) | Sort-Object -Unique

function Test-AppxPackageInstalled {
    param(
        [string[]]$NamePatterns
    )

    if (-not $NamePatterns -or $NamePatterns.Count -eq 0) {
        return $false
    }

    $AppxPackages = @(Get-AppxPackage -ErrorAction SilentlyContinue)
    if (-not $AppxPackages) {
        $AppxPackages = @(Get-AppxPackage -AllUsers -ErrorAction SilentlyContinue)
    }

    foreach ($Pattern in $NamePatterns) {
        foreach ($Package in $AppxPackages) {
            $Candidates = @(
                $Package.Name,
                $Package.DisplayName,
                $Package.PackageFamilyName,
                $Package.FullName,
                $Package.Id
            ) | Where-Object { $_ }

            foreach ($Candidate in $Candidates) {
                if ($Candidate -like $Pattern) {
                    return $true
                }
            }
        }
    }

    return $false
}

function Test-MicrosoftStoreReady {
    $StorePatterns = @(
        '*Microsoft.WindowsStore*',
        '*WindowsStore*',
        '*Store*'
    )

    $StoreDetected = Test-AppxPackageInstalled -NamePatterns $StorePatterns
    if (-not $StoreDetected) {
        return $false
    }

    $StorePackage = Get-AppxPackage -Name Microsoft.WindowsStore -AllUsers -ErrorAction SilentlyContinue | Select-Object -First 1
    if (-not $StorePackage) {
        return $false
    }

    if ($StorePackage.InstallLocation) {
        $StoreExe = Join-Path $StorePackage.InstallLocation 'WinStore.App.exe'
        if (Test-Path $StoreExe) {
            return $true
        }
    }

    return $true
}

function ConvertTo-NormalizedPackageList {
    param(
        [string[]]$PackageList
    )

    if (-not $PackageList) {
        return @()
    }

    return @($PackageList |
        Where-Object { -not [string]::IsNullOrWhiteSpace($_) } |
        ForEach-Object { $_.Trim() } |
        Sort-Object -Unique)
}

function Get-WingetTableIds {
    param(
        [string]$Result,
        [int]$IdColumnIndex = 1
    )

    $Ids = [System.Collections.Generic.HashSet[string]]::new([System.StringComparer]::OrdinalIgnoreCase)
    if ([string]::IsNullOrWhiteSpace($Result)) {
        return $Ids
    }

    foreach ($Line in ($Result -split "`r?`n")) {
        $Trimmed = $Line.Trim()
        if ([string]::IsNullOrWhiteSpace($Trimmed)) {
            continue
        }

        if ($Trimmed -match '^(Name|No packages found|No results found|No upgrades available|No package matches|No package updates available|Package\s+Id\s+Version)') {
            continue
        }

        if ($Trimmed -match '^\d+\s+(package|packages?)\(s\)?\s+have') {
            continue
        }

        $Columns = [regex]::Split($Trimmed, '\s{2,}') | Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
        if ($Columns.Count -le $IdColumnIndex) {
            continue
        }

        $CandidateColumn = $Columns[$IdColumnIndex].Trim()
        if ([string]::IsNullOrWhiteSpace($CandidateColumn)) {
            continue
        }

        $Candidate = (($CandidateColumn -split '\s+')[0]).Trim()
        if ([string]::IsNullOrWhiteSpace($Candidate)) {
            continue
        }

        if ($Candidate.Contains('\\') -or $Candidate.Contains('/')) {
            continue
        }

        if ($Candidate -match '^(?:\d+\.){1,}\d+$') {
            continue
        }

        $IsLikelyPackageId = $Candidate -match '^[A-Za-z0-9][A-Za-z0-9.+-]*[A-Za-z0-9]$' -or $Candidate -match '^[A-Za-z0-9]$'
        if (-not $IsLikelyPackageId) {
            continue
        }

        if ($Candidate -match '^(?:[A-Za-z]+|\d+)$') {
            continue
        }

        $Ids.Add($Candidate) | Out-Null
    }

    return $Ids
}

function Get-WingetInstalledSnapshot {
    param(
        [string[]]$PackageIds
    )

    $Installed = @{}
    foreach ($Pkg in @($PackageIds | Sort-Object -Unique)) {
        if ([string]::IsNullOrWhiteSpace($Pkg)) {
            continue
        }

        $null = & winget list --id $Pkg --exact --accept-source-agreements --disable-interactivity 2>$null | Out-String
        if ($LASTEXITCODE -eq 0) {
            $Installed[$Pkg] = $true
        }
    }

    return $Installed
}

function Get-WingetUpgradableSnapshot {
    param(
        [string[]]$PackageIds
    )

    $Upgradable = @{}
    foreach ($Pkg in @($PackageIds | Sort-Object -Unique)) {
        if ([string]::IsNullOrWhiteSpace($Pkg)) {
            continue
        }

        $null = & winget upgrade --id $Pkg --exact --accept-source-agreements --disable-interactivity 2>$null | Out-String
        if ($LASTEXITCODE -eq 0) {
            $Upgradable[$Pkg] = $true
        }
    }

    return $Upgradable
}

$AllPackageIdsForSnapshot = @($SystemRuntimes + $EssentialApps + $MicrosoftApps + $OptionalApps)
$script:WingetInstalledSnapshot = Get-WingetInstalledSnapshot -PackageIds $AllPackageIdsForSnapshot
$script:WingetUpgradableSnapshot = Get-WingetUpgradableSnapshot -PackageIds $AllPackageIdsForSnapshot

function Test-WingetPackageInstalled {
    param(
        [string]$PackageId
    )

    if ([string]::IsNullOrWhiteSpace($PackageId)) {
        return $false
    }

    return $script:WingetInstalledSnapshot.ContainsKey($PackageId)
}

function Test-WingetPackageUpgradable {
    param(
        [string]$PackageId
    )

    if ([string]::IsNullOrWhiteSpace($PackageId)) {
        return $false
    }

    return $script:WingetUpgradableSnapshot.ContainsKey($PackageId)
}

function Write-ConsoleSection {
    param(
        [string]$Title,
        [ConsoleColor]$Color = [ConsoleColor]::Cyan
    )

    Write-Host ""
    Write-Host "=====================================================" -ForegroundColor $Color
    Write-Host "  $Title" -ForegroundColor $Color
    Write-Host "=====================================================" -ForegroundColor $Color
}

function ConvertTo-SelectionList {
    param(
        [string]$InputValue,
        [string[]]$Options
    )

    $Selected = [System.Collections.Generic.List[string]]::new()
    if ([string]::IsNullOrWhiteSpace($InputValue)) {
        return $Selected
    }

    foreach ($Part in ($InputValue -split ',')) {
        $Value = $Part.Trim()
        if ([string]::IsNullOrWhiteSpace($Value)) {
            continue
        }

        $Index = 0
        if ([int]::TryParse($Value, [ref]$Index)) {
            if ($Index -ge 1 -and $Index -le $Options.Count) {
                $Selected.Add($Options[$Index - 1])
            }
        }
    }

    return $Selected
}

function Select-PackageIdsInteractively {
    param(
        [string[]]$MissingPackages,
        [string[]]$UpgradePackages
    )

    $SelectedInstall = @()
    $SelectedUpgrade = @()

    if ($MissingPackages.Count -gt 0) {
        Write-ConsoleSection -Title "INSTALL OPTIONS"
        Write-Host "Missing packages: [A]ll [N]one [C]ustom" -ForegroundColor Cyan
        $InstallChoice = Read-Host "Choose an option"
        switch ($InstallChoice.Trim().ToLower()) {
            { $_ -in @('a', 'all') } { $SelectedInstall = $MissingPackages }
            { $_ -in @('n', 'none', '0') } { $SelectedInstall = @() }
            default {
                Write-Host "Custom selection: enter numbers like 1,3,5" -ForegroundColor Yellow
                for ($i = 0; $i -lt $MissingPackages.Count; $i++) {
                    Write-Host ("  [{0}] {1}" -f ($i + 1), $MissingPackages[$i]) -ForegroundColor Red
                }
                $InstallInput = Read-Host "Select missing packages"
                $SelectedInstall = @(ConvertTo-SelectionList -InputValue $InstallInput -Options $MissingPackages)
            }
        }
    }

    if ($UpgradePackages.Count -gt 0) {
        Write-ConsoleSection -Title "UPDATE OPTIONS"
        Write-Host "Upgradable packages: [A]ll [N]one [C]ustom" -ForegroundColor Cyan
        $UpgradeChoice = Read-Host "Choose an option"
        switch ($UpgradeChoice.Trim().ToLower()) {
            { $_ -in @('a', 'all') } { $SelectedUpgrade = $UpgradePackages }
            { $_ -in @('n', 'none', '0') } { $SelectedUpgrade = @() }
            default {
                Write-Host "Custom selection: enter numbers like 1,3,5" -ForegroundColor Yellow
                for ($i = 0; $i -lt $UpgradePackages.Count; $i++) {
                    Write-Host ("  [{0}] {1}" -f ($i + 1), $UpgradePackages[$i]) -ForegroundColor Yellow
                }
                $UpgradeInput = Read-Host "Select packages to update"
                $SelectedUpgrade = @(ConvertTo-SelectionList -InputValue $UpgradeInput -Options $UpgradePackages)
            }
        }
    }

    return [pscustomobject]@{
        Install = @($SelectedInstall | Sort-Object -Unique)
        Upgrade = @($SelectedUpgrade | Sort-Object -Unique)
    }
}

function Get-PackageStatus {
    param(
        [string[]]$PackageList,
        [string]$SectionName
    )

    $PackageList = ConvertTo-NormalizedPackageList -PackageList $PackageList

    Write-Host "`n===== $SectionName =====" -ForegroundColor Cyan
    $InstalledList = [System.Collections.Generic.List[string]]::new()
    $UpgradeList   = [System.Collections.Generic.List[string]]::new()
    $MissingList   = [System.Collections.Generic.List[string]]::new()

    foreach ($Pkg in $PackageList) {
        $IsInstalled = Test-WingetPackageInstalled -PackageId $Pkg
        $NeedsUpgrade = $false

        if ($IsInstalled) {
            $NeedsUpgrade = Test-WingetPackageUpgradable -PackageId $Pkg
        }

        if (-not $IsInstalled) {
            $AppxPatterns = @()
            switch ($Pkg) {
                'Discord.Discord' { $AppxPatterns = @('*Discord*', '*Discord*') }
                'WhatsApp.WhatsApp' { $AppxPatterns = @('*WhatsApp*', '*5319275A.WhatsAppDesktop*', '*WhatsAppDesktop*') }
                'Microsoft.WindowsTerminal' { $AppxPatterns = @('*WindowsTerminal*', '*Terminal*') }
                'Microsoft.PowerShell' { $AppxPatterns = @('*PowerShell*', 'Microsoft.PowerShell*') }
                'Microsoft.Edge' { $AppxPatterns = @('*MicrosoftEdge*', '*Edge*') }
                default { $AppxPatterns = @() }
            }

            if ($AppxPatterns.Count -gt 0) {
                $IsInstalled = Test-AppxPackageInstalled -NamePatterns $AppxPatterns
            }
        }

        if ($NeedsUpgrade) {
            Write-Host "[UPDATE] $Pkg" -ForegroundColor Yellow
            $UpgradeList.Add($Pkg)
        }
        elseif ($IsInstalled) {
            $StoreHint = $false
            if ($Pkg -eq 'Discord.Discord' -or $Pkg -eq 'WhatsApp.WhatsApp' -or $Pkg -eq 'Microsoft.Edge' -or $Pkg -eq 'Microsoft.WindowsTerminal') {
                $StoreHint = $true
            }

            Write-Host "[OK] $Pkg" -ForegroundColor Green
            if ($StoreHint) {
                Write-Host "     [!] Microsoft Store/AppX package detected. Check the Store for newer versions if needed." -ForegroundColor Yellow
            }
            $InstalledList.Add($Pkg)
        }
        else {
            Write-Host "[INSTALL] $Pkg" -ForegroundColor Red
            $MissingList.Add($Pkg)
        }
    }

    return [pscustomobject]@{
        Installed = @($InstalledList | Sort-Object -Unique)
        Upgrade = @($UpgradeList | Sort-Object -Unique)
        Missing = @($MissingList | Sort-Object -Unique)
    }
}

function Invoke-AppInstallation {
    param(
        [string[]]$PackageIds,
        [string]$Mode
    )

    foreach ($Id in @($PackageIds | Sort-Object -Unique)) {
        Write-Host "`n[*] Executing $Mode for: $Id..." -ForegroundColor Cyan
        try {
            if ($Mode -eq 'Install') {
                winget install --id $Id --exact --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
            }
            else {
                winget upgrade --id $Id --exact --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
            }

            if ($LASTEXITCODE -ne 0) {
                throw "winget $Mode exited with code $LASTEXITCODE for $Id"
            }
        }
        catch {
            Write-Host "[!] ${Mode} failed for ${Id}: $($_.Exception.Message)" -ForegroundColor Red
        }
    }
}

Write-Host "`n[*] Querying installed packages and pending upgrades..." -ForegroundColor Cyan

$RuntimeStatus = Get-PackageStatus -PackageList $SystemRuntimes -SectionName 'SYSTEM RUNTIMES'
$EssentialStatus = Get-PackageStatus -PackageList $EssentialApps -SectionName 'ESSENTIAL APPLICATIONS'
$MicrosoftStatus = Get-PackageStatus -PackageList $MicrosoftApps -SectionName 'MICROSOFT APPLICATIONS'
$OptionalStatus = Get-PackageStatus -PackageList $OptionalApps -SectionName 'OPTIONAL APPLICATIONS'

$AllMissing = @(( $RuntimeStatus.Missing + $EssentialStatus.Missing + $MicrosoftStatus.Missing + $OptionalStatus.Missing ) | Sort-Object -Unique)
$AllUpgrade = @(( $RuntimeStatus.Upgrade + $EssentialStatus.Upgrade + $MicrosoftStatus.Upgrade + $OptionalStatus.Upgrade ) | Sort-Object -Unique)
$AllOk = @(( $RuntimeStatus.Installed + $EssentialStatus.Installed + $MicrosoftStatus.Installed + $OptionalStatus.Installed ) | Sort-Object -Unique)

Write-ConsoleSection -Title "SUMMARY"
Write-Host "[OK]     $($AllOk.Count)" -ForegroundColor Green
Write-Host "[INSTALL] $($AllMissing.Count)" -ForegroundColor Red
Write-Host "[UPDATE]  $($AllUpgrade.Count)" -ForegroundColor Yellow

$SelectedPackages = [pscustomobject]@{
    Install = @()
    Upgrade = @()
}

if ($AutoApprove) {
    $SelectedPackages.Install = $AllMissing
    $SelectedPackages.Upgrade = $AllUpgrade
}
elseif ($SelectPackages) {
    $SelectedPackages = Select-PackageIdsInteractively -MissingPackages $AllMissing -UpgradePackages $AllUpgrade
}
else {
    if ($AllMissing.Count -gt 0) {
        $InstallPrompt = Read-Host 'Install missing packages? (y/n)'
        if ($InstallPrompt.Trim().ToLower() -eq 'y') {
            $SelectedPackages.Install = $AllMissing
        }
    }

    if ($AllUpgrade.Count -gt 0) {
        $UpgradePrompt = Read-Host 'Update available packages? (y/n)'
        if ($UpgradePrompt.Trim().ToLower() -eq 'y') {
            $SelectedPackages.Upgrade = $AllUpgrade
        }
    }
}

if ($SelectedPackages.Install.Count -gt 0) {
    Write-ConsoleSection -Title "INSTALLING MISSING PACKAGES"
    Invoke-AppInstallation -PackageIds $SelectedPackages.Install -Mode 'Install'
}

if ($SelectedPackages.Upgrade.Count -gt 0) {
    Write-ConsoleSection -Title "UPDATING AVAILABLE PACKAGES"
    Invoke-AppInstallation -PackageIds $SelectedPackages.Upgrade -Mode 'Upgrade'
}

if ($AllMissing.Count -eq 0 -and $AllUpgrade.Count -eq 0) {
    Write-Host "`n[OK] All essential and optional packages are up to date." -ForegroundColor Green
}

$env:Path = [System.Environment]::GetEnvironmentVariable('Path', 'Machine') + ';' + [System.Environment]::GetEnvironmentVariable('Path', 'User')

Write-Host "`n[+] Finished. Press Enter to exit..." -ForegroundColor Gray
Read-Host | Out-Null
