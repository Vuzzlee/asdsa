Add-Type -AssemblyName System.IO.Compression.FileSystem
Add-Type -AssemblyName System.Windows.Forms

[console]::WindowWidth = 110
[console]::WindowHeight = 35
[console]::BufferWidth = 110

$colorTitle = "Magenta"
$colorSubtitle = "DarkMagenta"
$colorPrompt = "Cyan"
$colorWarning = "Yellow"
$colorError = "Red"
$colorSuccess = "Green"
$colorInfo = "White"
$colorHeader = "Blue"
$colorProgress = "DarkCyan"

function Show-Banner {
    Clear-Host
    $banner = @"
    
    ╔═══════════════════════════════════════════════════════════════════════════════╗
    ║                                                                               ║
    ║  ███╗   ███╗███████╗██╗  ██╗███╗   ███╗███████╗████████╗  ██╗   ██╗██╗        ║
    ║  ████╗ ████║██╔════╝██║  ██║████╗ ████║██╔════╝╚══██╔══╝  ╚██╗ ██╔╝██║        ║
    ║  ██╔████╔██║█████╗  ███████║██╔████╔██║█████╗     ██║      ╚████╔╝ ██║        ║
    ║  ██║╚██╔╝██║██╔══╝  ██╔══██║██║╚██╔╝██║██╔══╝     ██║       ╚██╔╝  ██║        ║
    ║  ██║ ╚═╝ ██║███████╗██║  ██║██║ ╚═╝ ██║███████╗   ██║███████╗██║   ███████╗   ║
    ║  ╚═╝     ╚═╝╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝╚══════╝   ╚═╝╚══════╝╚═╝   ╚══════╝   ║
    ║                                                                               ║
    ║                           MOD TARAYICI PRO                                    ║
    ╚═══════════════════════════════════════════════════════════════════════════════╝
"@
    
    Write-Host $banner -ForegroundColor $colorTitle
    Write-Host " Sürüm: " -ForegroundColor $colorSubtitle -NoNewline
    Write-Host "v1.2.0 - Premium Mod Tarayıcı" -ForegroundColor $colorInfo
    Write-Host " Yapımcı: " -ForegroundColor $colorSubtitle -NoNewline
    Write-Host "Mehmet_yl" -ForegroundColor $colorInfo
    Write-Host " Tarih: " -ForegroundColor $colorSubtitle -NoNewline
    Write-Host "$(Get-Date -Format "dd.MM.yyyy HH:mm")" -ForegroundColor $colorInfo
    Write-Host ""
}

function Show-ModInfo {
    param (
        [string]$Title,
        [string]$FileName,
        [string]$Location,
        [string]$Url = $null,
        [string]$ModType = "Bilinmeyen",
        [string]$Hash = $null
    )
    
    if ($ModType -eq "Yasaklı" -or $ModType -eq "Bilinmeyen") {
        $statusColor = switch ($ModType) {
            "Yasaklı" { $colorError }
            "Bilinmeyen" { $colorWarning }
            default { $colorInfo }
        }
        
        Write-Host ""
        Write-Host "Mod Bilgisi: " -NoNewline -ForegroundColor $statusColor
        Write-Host $Title -ForegroundColor White
        Write-Host "Dosya Adı: " -NoNewline -ForegroundColor $statusColor
        Write-Host $FileName -ForegroundColor White
        Write-Host "Konum: " -NoNewline -ForegroundColor $statusColor
        
        $folderPath = Split-Path -Path $Location
        Write-Host $folderPath -ForegroundColor White
        
        if ($Url) {
            Write-Host "İndirme Linki: " -NoNewline -ForegroundColor $statusColor
            Write-Host $Url -ForegroundColor DarkGray
        }
        
        Write-Host ""
    }
}

function Show-SummaryTable {
    param (
        [array]$UnknownMods,
        [array]$BannedMods,
        [array]$CleanMods
    )
    
    $totalMods = $UnknownMods.Count + $BannedMods.Count + $CleanMods.Count
    
    Write-Host ""
    Write-Host " ╔════════════════════════════════════════════════════════════════════╗" -ForegroundColor $colorHeader
    Write-Host " ║                        TARAMA SONUÇLARI                            ║" -ForegroundColor $colorHeader
    Write-Host " ╠════════════════════════════════════╦═══════════════════════════════╣" -ForegroundColor $colorHeader
    Write-Host " ║ Toplam Bulunan Mod:                ║" -NoNewline -ForegroundColor $colorHeader
    Write-Host ("{0,30}" -f $totalMods) -NoNewline -ForegroundColor $colorInfo
    Write-Host " ║" -ForegroundColor $colorHeader
    Write-Host " ╠════════════════════════════════════╬═══════════════════════════════╣" -ForegroundColor $colorHeader
    Write-Host " ║ Şüpheli/Bilinmeyen Modlar:         ║" -NoNewline -ForegroundColor $colorHeader
    Write-Host ("{0,30}" -f $UnknownMods.Count) -NoNewline -ForegroundColor $colorWarning
    Write-Host " ║" -ForegroundColor $colorHeader
    Write-Host " ║ Yasaklı Modlar:                    ║" -NoNewline -ForegroundColor $colorHeader
    Write-Host ("{0,30}" -f $BannedMods.Count) -NoNewline -ForegroundColor $colorError
    Write-Host " ║" -ForegroundColor $colorHeader
    Write-Host " ║ Temiz Modlar:                      ║" -NoNewline -ForegroundColor $colorHeader
    Write-Host ("{0,30}" -f $CleanMods.Count) -NoNewline -ForegroundColor $colorSuccess
    Write-Host " ║" -ForegroundColor $colorHeader
    Write-Host " ╚════════════════════════════════════╩═══════════════════════════════╝" -ForegroundColor $colorHeader
    Write-Host ""
}

function Start-ModScanner {
    Show-Banner
    
    Write-Host "Başlamak için Enter'a basın..." -ForegroundColor $colorPrompt
    Read-Host
    Write-Host ""

    $process = Get-Process javaw -ErrorAction SilentlyContinue

    if ($process) {
        $startTime = $process.StartTime
        $elapsedTime = (Get-Date) - $startTime
        $elapsedHours = $elapsedTime.Hours
        $elapsedMinutes = $elapsedTime.Minutes
        $elapsedSeconds = $elapsedTime.Seconds

        Write-Host ""
        Write-Host " ┌──────────────────────────── MINECRAFT DURUMU ────────────────────────────┐" -ForegroundColor Cyan
        Write-Host "   DURUM                : " -NoNewline -ForegroundColor Cyan
        Write-Host "ÇALIŞIYOR" -NoNewline -ForegroundColor $colorSuccess
        Write-Host "                                                " -ForegroundColor Cyan
        Write-Host "  Process ID           : " -NoNewline -ForegroundColor Cyan
        Write-Host ("{0,-60}" -f $process.Id) -NoNewline -ForegroundColor White
        Write-Host "" -ForegroundColor Cyan
        Write-Host "  Başlatılma Zamanı    : " -NoNewline -ForegroundColor Cyan
        Write-Host ("{0,-60}" -f $startTime) -NoNewline -ForegroundColor White
        Write-Host "" -ForegroundColor Cyan
        Write-Host "  Çalışma Süresi       : " -NoNewline -ForegroundColor Cyan
        Write-Host ("{0,-60}" -f "$elapsedHours saat $elapsedMinutes dakika $elapsedSeconds saniye") -NoNewline -ForegroundColor White
        Write-Host "" -ForegroundColor Cyan
        Write-Host " └──────────────────────────────────────────────────────────────────────────┘" -ForegroundColor Cyan
        Write-Host ""
    }
    else {
        Write-Host " ┌──────────────────────────── MINECRAFT DURUMU ────────────────────────────┐" -ForegroundColor Cyan
        Write-Host "   DURUM                : " -NoNewline -ForegroundColor Cyan
        Write-Host "ÇALIŞMIYOR" -NoNewline -ForegroundColor $colorError
        Write-Host "                                             " -ForegroundColor Cyan
        Write-Host " └──────────────────────────────────────────────────────────────────────────┘" -ForegroundColor Cyan
        Write-Host ""
    }
}

$bannedMods = @(
    @{ name = "Wurst Client"; value = "wurst" },
    @{ name = "Meteor Client"; value = "meteor" },
    @{ name = "Aristois"; value = "aristois" },
    @{ name = "Inertia Client"; value = "inertia" },
    @{ name = "Polonium"; value = "polonium" },
    @{ name = "Prestige Client"; value = "prestige" },
    @{ name = "LiquidBounce"; value = "liquidbounce" },
    @{ name = "Vape Client"; value = "vape" },
    @{ name = "Impact Client"; value = "impact" },
    @{ name = "MinecraftHax"; value = "minecrafthax" },
    @{ name = "Wolfram Client"; value = "wolfram" },
    @{ name = "Ares Client"; value = "ares" },
    @{ name = "Expensive Client"; value = "expensive" },
    @{ name = "Rise Client"; value = "rise" },
    @{ name = "Sigma Client"; value = "sigma" },
    @{ name = "Phantom"; value = "phantom" },
    @{ name = "Zenora Client"; value = "zenora" },
    @{ name = "Kwish Client"; value = "kwish" },
    @{ name = "Thor Client"; value = "thor" },
    @{ name = "Grim Client"; value = "grim" },
    @{ name = "Baritone"; value = "baritone" },
    @{ name = "Valiant Client"; value = "valiant" },
    @{ name = "Biggie Client"; value = "biggie" },
    @{ name = "Litka Client"; value = "litka" },
    @{ name = "VegaLine Client"; value = "vegaline" },
    @{ name = "Ghost Client"; value = "ghostclient" },
    @{ name = "Xenon Client"; value = "xenon" },
    @{ name = "Zerodium Client"; value = "zerodium" },
    @{ name = "Ragnolik Client"; value = "ragnolik" },
    @{ name = "ThunderHack ReCode"; value = "thunderhack" },
    @{ name = "Dream Client"; value = "dream" },
    @{ name = "SalHack"; value = "salhack" },
    @{ name = "Argon"; value = "argon" },
    @{ name = "Pandora Client"; value = "pandora" },
    @{ name = "Future Client"; value = "future" },
    @{ name = "BleachHack"; value = "bleach" },
    @{ name = "Shield Statuses"; value = "shieldstatus" },
    @{ name = "Auto Totem"; value = "autototem" },
    @{ name = "Mouse Tweaks"; value = "mousetweaks" },
    @{ name = "Inventory Profiles Next"; value = "inventoryprofilesnext" },
    @{ name = "Item Scroller"; value = "itemscroller" },
    @{ name = "Cut Through"; value = "cutthrough" },
    @{ name = "Debugify"; value = "Debugify" }
)

function Get-FileHashSHA1 {
    param (
        [string]$filePath
    )
    try {
        $hashAlgorithm = [System.Security.Cryptography.SHA1]::Create()
        $fileStream = [System.IO.File]::OpenRead($filePath)
        try {
            $hashBytes = $hashAlgorithm.ComputeHash($fileStream)
        }
        finally {
            $fileStream.Close()
            $hashAlgorithm.Dispose()
        }
        $hash = [BitConverter]::ToString($hashBytes).Replace("-", "")
        return $hash
    }
    catch {
        Write-Host "Hash hesaplanırken hata: $($_.Exception.Message)" -ForegroundColor $colorError
        return ""
    }
}

function Get-AdsUrl {
    param (
        [string]$filePath
    )
    try {
        $ads = Get-Content -Stream Zone.Identifier $filePath -ErrorAction SilentlyContinue -Raw
        if ($ads -match "HostUrl=(.+)") {
            return $matches[1]
        }
    }
    catch {
    }
    
    return $null
}

function Get-ModrinthData {
    param (
        [string]$hash
    )
    
    if ([string]::IsNullOrEmpty($hash)) {
        return @{ Name = ""; Slug = "" }
    }
    
    $modrinthApiUrl = "https://api.modrinth.com/v2/version_file/$hash"
    try {
        $response = Invoke-RestMethod -Uri $modrinthApiUrl -Method Get -TimeoutSec 5 -ErrorAction Stop
        if ($response.project_id) {
            $projectResponse = "https://api.modrinth.com/v2/project/$($response.project_id)"
            $projectData = Invoke-RestMethod -Uri $projectResponse -Method Get -TimeoutSec 5 -ErrorAction Stop
            return @{ Name = $projectData.title; Slug = $projectData.slug }
        }
    }
    catch {
    }
    
    return @{ Name = ""; Slug = "" }
}

function Start-MainScan {
    $unknownMods = @()
    $foundBannedMods = @()
    $cleanMods = @()

    $allDrives = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Root -match '^[A-Z]:\\$' }

    $recursivePaths = @(
        "$env:USERPROFILE\AppData\Roaming\.minecraft\mods",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\mods.orig",
        "$env:USERPROFILE\Desktop",
        "$env:USERPROFILE\Downloads",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\shaderpacks",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\texturepacks",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\feather-mods",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\shaderpacks",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\texturepacks",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\screenshots",
        "$env:USERPROFILE\AppData\Roaming\.minecraft\screenshots",
        "$env:USERPROFILE\APPDATA\Roaming\Microsoft\Windows\Recent"
        "$env:USERPROFILE\AppData\Roaming\.minecraft\versions",
        "$env:USERPROFILE\AppData\Roaming\.feather\user-mods"
        "$env:USERPROFILE\Documents",
        "$env:USERPROFILE\Pictures",
        "$env:USERPROFILE\Music",
        "$env:USERPROFILE\Videos"
    )

    $nonRecursivePaths = @(
        "$env:USERPROFILE\AppData\Roaming\.minecraft\libraries\java"
    )

    $allPaths = $recursivePaths + $nonRecursivePaths

    $validRecursivePaths = @()
    $validNonRecursivePaths = @()
    
    foreach ($path in $recursivePaths) {
        if (Test-Path $path) {
            $validRecursivePaths += $path
        }
    }
    
    foreach ($path in $nonRecursivePaths) {
        if (Test-Path $path) {
            $validNonRecursivePaths += $path
        }
    }

    if ($validRecursivePaths.Count -eq 0 -and $validNonRecursivePaths.Count -eq 0) {
        Write-Host "Hiçbir geçerli klasör bulunamadı!" -ForegroundColor $colorError
        Write-Host "Minecraft kurulumunuzu kontrol edin." -ForegroundColor $colorWarning
        return
    }

    Write-Host ""
    Write-Host "Modlar taranıyor..." -ForegroundColor $colorWarning
    Write-Host ""

    foreach ($path in $validRecursivePaths) {
        
        try {
            $files = Get-ChildItem -Path $path -Include *.jar, *.zip -Recurse -ErrorAction SilentlyContinue
            
            if ($files.Count -eq 0) {
                continue
            }
            
            foreach ($file in $files) {
                try {
                    if ($file.Length -eq 0) {
                        continue
                    }
                    
                    $hash = Get-FileHashSHA1 -filePath $file.FullName
                    
                    if ([string]::IsNullOrEmpty($hash)) {
                        continue
                    }
                    
                    $modData = Get-ModrinthData -hash $hash
                    $url = Get-AdsUrl $file.FullName
                    
                    $isBanned = $false
                    $bannedModName = ""
                    
                    foreach ($mod in $bannedMods) {
                        if ($file.Name -like "*$($mod.value)*") {
                            $isBanned = $true
                            $bannedModName = $mod.name
                            break
                        }
                    }
                    
                    if ($isBanned) {
                        Show-ModInfo -FileName $file.Name -Title $bannedModName -Location $file.FullName -Url $url -ModType "Yasaklı" -Hash $hash
                        
                        $foundBannedMods += @{
                            Name    = $file.Name
                            Path    = $file.FullName
                            ModName = $bannedModName
                            Hash    = $hash
                            Url     = $url
                        }
                    }
                    elseif ($modData.Slug -eq "") {
                        Show-ModInfo -FileName $file.Name -Title "Bilinmeyen Mod" -Location $file.FullName -Url $url -ModType "Bilinmeyen" -Hash $hash
                        
                        $unknownMods += @{
                            Name = $file.Name
                            Path = $file.FullName
                            Hash = $hash
                            Url  = $url
                        }
                    }
                    else {
                        $cleanMods += @{
                            Name    = $file.Name
                            Path    = $file.FullName
                            ModName = $modData.Name
                            Slug    = $modData.Slug
                            Hash    = $hash
                            Url     = $url
                        }
                    }
                }
                catch {
                    Write-Host "    Dosya işlenirken hata: $($_.Exception.Message)" -ForegroundColor $colorError
                }
            }
        }
        catch {
            Write-Host "  Klasör taranırken hata: $($_.Exception.Message)" -ForegroundColor $colorError
        }
    }

    Write-Host ""
    foreach ($path in $validNonRecursivePaths) {
        
        try {
            if ($path -like "*Recent*") {
                $files = Get-ChildItem -Path $path -Include *.lnk, *.jar, *.zip -ErrorAction SilentlyContinue
            } else {
                $files = Get-ChildItem -Path $path -Include *.jar, *.zip -ErrorAction SilentlyContinue
            }
            
            if ($files.Count -eq 0) {
                continue
            }
            
            foreach ($file in $files) {
                try {
                    if ($file.Length -eq 0) {
                        continue
                    }
                    
                    $hash = Get-FileHashSHA1 -filePath $file.FullName
                    
                    if ([string]::IsNullOrEmpty($hash)) {
                        continue
                    }
                    
                    $modData = Get-ModrinthData -hash $hash
                    $url = Get-AdsUrl $file.FullName
                    
                    $isBanned = $false
                    $bannedModName = ""
                    
                    foreach ($mod in $bannedMods) {
                        if ($file.Name -like "*$($mod.value)*") {
                            $isBanned = $true
                            $bannedModName = $mod.name
                            break
                        }
                    }
                    
                    if ($isBanned) {
                        Write-Host "    YASAKLI MOD BULUNDU: $bannedModName" -ForegroundColor $colorError
                        Show-ModInfo -Title $bannedModName -FileName $file.Name -Location $file.FullName -Url $url -ModType "Yasaklı" -Hash $hash
                        
                        $foundBannedMods += @{
                            Name    = $file.Name
                            Path    = $file.FullName
                            ModName = $bannedModName
                            Hash    = $hash
                            Url     = $url
                        }
                    }
                    elseif ($modData.Slug -eq "") {
                        Show-ModInfo -FileName $file.Name -Title "Bilinmeyen Mod" -Location $file.FullName -Url $url -ModType "Bilinmeyen" -Hash $hash
                        
                        $unknownMods += @{
                            Name = $file.Name
                            Path = $file.FullName
                            Hash = $hash
                            Url  = $url
                        }
                    }
                    else {
                        $cleanMods += @{
                            Name    = $file.Name
                            Path    = $file.FullName
                            ModName = $modData.Name
                            Slug    = $modData.Slug
                            Hash    = $hash
                            Url     = $url
                        }
                    }
                }
                catch {
                    Write-Host "    Dosya işlenirken hata: $($_.Exception.Message)" -ForegroundColor $colorError
                }
            }
        }
        catch {
            Write-Host "  Klasör taranırken hata: $($_.Exception.Message)" -ForegroundColor $colorError
        }
    }

    Write-Host ""
    Write-Host "Diskler taranıyor..." -ForegroundColor $colorWarning
    
    $filteredDrives = $allDrives | Where-Object { $_.Root -ne "C:\" }
    
    foreach ($drive in $filteredDrives) {
        try {
            Write-Host "Disk taranıyor: $($drive.Root)" -ForegroundColor $colorInfo
            
            $files = Get-ChildItem -Path $drive.Root -Include *.jar, *.zip -Recurse -File -ErrorAction SilentlyContinue
            
            if ($files.Count -gt 0) {
                foreach ($file in $files) {
                    try {
                        $hash = Get-FileHashSHA1 -filePath $file.FullName
                        
                        if ([string]::IsNullOrEmpty($hash)) {
                            continue
                        }
                        
                        $modData = Get-ModrinthData -hash $hash
                        $url = Get-AdsUrl $file.FullName
                        
                        $isBanned = $false
                        $bannedModName = ""
                        
                        foreach ($mod in $bannedMods) {
                            if ($file.Name -like "*$($mod.value)*") {
                                $isBanned = $true
                                $bannedModName = $mod.name
                                break
                            }
                        }
                        
                        if ($isBanned) {
                            Write-Host "    YASAKLI MOD BULUNDU: $bannedModName" -ForegroundColor $colorError
                            Show-ModInfo -Title $bannedModName -FileName $file.Name -Location $file.FullName -Url $url -ModType "Yasaklı" -Hash $hash
                            
                            $foundBannedMods += @{
                                Name    = $file.Name
                                Path    = $file.FullName
                                ModName = $bannedModName
                                Hash    = $hash
                                Url     = $url
                            }
                        }
                        elseif ($modData.Slug -eq "") {
                            Show-ModInfo -Title "Bilinmeyen Mod" -FileName $file.Name -Location $file.FullName -Url $url -ModType "Bilinmeyen" -Hash $hash
                            
                            $unknownMods += @{
                                Name = $file.Name
                                Path = $file.FullName
                                Hash = $hash
                                Url  = $url
                            }
                        }
                        else {
                            $cleanMods += @{
                                Name    = $file.Name
                                Path    = $file.FullName
                                ModName = $modData.Name
                                Slug    = $modData.Slug
                                Hash    = $hash
                                Url     = $url
                            }
                        }
                    }
                    catch {
                        Write-Host "    Dosya işlenirken hata: $($_.Exception.Message)" -ForegroundColor $colorError
                    }
                }
            }
        }
        catch {
            Write-Host "  Disk taranırken hata: $($_.Exception.Message)" -ForegroundColor $colorError
        }
    }

    Show-SummaryTable -UnknownMods $unknownMods -BannedMods $foundBannedMods -CleanMods $cleanMods

    if ($unknownMods.Count -gt 0) {
        Write-Host "ŞÜPHELİ MODLAR:" -ForegroundColor $colorWarning
        $unknownMods | ForEach-Object { 
            Write-Host "- $($_.Name)" -ForegroundColor $colorWarning
        }
        Write-Host ""
    }

    if ($foundBannedMods.Count -gt 0) {
        Write-Host "YASAKLI MODLAR:" -ForegroundColor $colorError
        $foundBannedMods | ForEach-Object { 
            Write-Host "- $($_.ModName): $($_.Name)" -ForegroundColor $colorError
        }
        Write-Host ""
    }

    if ($unknownMods.Count -eq 0 -and $foundBannedMods.Count -eq 0) {
        Write-Host "Hiçbir şüpheli veya yasaklı mod bulunamadı." -ForegroundColor $colorSuccess
    }
    
    Write-Host ""
    Write-Host "Tarama Tamamlandı." -ForegroundColor $colorSuccess
}

Start-ModScanner
Start-MainScan
