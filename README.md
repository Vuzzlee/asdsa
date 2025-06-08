# Kont-function Invoke-ModSecurityCheck {
    param (
        [array]$UnknownMods,
        [array]$BannedMods,
        [array]$CleanMods
    )

    # Güvenlik riski analizi
    $securityRisk = $false
    $recommendations = @()

    # 1. Yasaklı mod kontrolü
    if ($BannedMods.Count -gt 0) {
        $securityRisk = $true
        $recommendations += "`n[!] ACİL EYLEM GEREKLİ: Sisteminizde yasaklı modlar bulundu!"
        $recommendations += "   - Bulunan yasaklı mod sayısı: $($BannedMods.Count)"
        $recommendations += "   - Önerilen aksiyon: Bu modları hemen silin veya kaldırın"
        
        $BannedMods | ForEach-Object {
            $recommendations += "     * $($_.ModName) (Dosya: $($_.Name))"
            
            # Ek bilgi için Modrinth kontrolü
            if (-not [string]::IsNullOrEmpty($_.Hash)) {
                $modInfo = Get-ModrinthData -hash $_.Hash
                if ($modInfo.Slug -ne "") {
                    $recommendations += "       Modrinth Link: https://modrinth.com/mod/$($modInfo.Slug)"
                }
            }
        }
    }

    # 2. Bilinmeyen mod kontrolü
    if ($UnknownMods.Count -gt 5) {  # Belirli bir eşik değeri
        $securityRisk = $true
        $recommendations += "`n[!] UYARI: Çok fazla bilinmeyen mod bulundu ($($UnknownMods.Count) adet"
        $recommendations += "   - Bu modlar resmi kaynaklardan doğrulanamadı"
        $recommendations += "   - Önerilen aksiyon: Her modu tek tek inceleyin veya güvenilir kaynaklardan edinin"
        
        # İlk 5 bilinmeyen modu listele
        $UnknownMods[0..4] | ForEach-Object {
            $recommendations += "     * $($_.Name)"
            if (-not [string]::IsNullOrEmpty($_.Url)) {
                $recommendations += "       İndirme Kaynağı: $($_.Url)"
            }
        }
        
        if ($UnknownMods.Count -gt 5) {
            $recommendations += "     * ...ve $(($UnknownMods.Count)-5) daha fazla"
        }
    }

    # 3. Hash doğrulama kontrolü
    $noHashMods = $UnknownMods | Where-Object { [string]::IsNullOrEmpty($_.Hash) }
    if ($noHashMods.Count -gt 0) {
        $securityRisk = $true
        $recommendations += "`n[!] UYARI: Hash değeri alınamayan modlar bulundu ($($noHashMods.Count) adet"
        $recommendations += "   - Bu modların bütünlüğü doğrulanamıyor"
        $recommendations += "   - Önerilen aksiyon: Bu modları yeniden indirin veya kaldırın"
    }

    # 4. Şüpheli indirme kaynakları kontrolü
    $suspiciousSources = $UnknownMods | Where-Object { 
        $_.Url -match "mediafire|dropbox|mega\.nz|anonfiles|zippyshare" -and 
        $_.Url -notmatch "modrinth|curseforge"
    }
    
    if ($suspiciousSources.Count -gt 0) {
        $securityRisk = $true
        $recommendations += "`n[!] UYARI: Şüpheli kaynaklardan indirilmiş modlar bulundu"
        $recommendations += "   - Resmi olmayan kaynaklardan mod indirmek risklidir"
        $recommendations += "   - Önerilen aksiyon: Bu modları resmi kaynaklardan (Modrinth/CurseForge) yeniden indirin"
        
        $suspiciousSources | Select-Object -First 3 | ForEach-Object {
            $recommendations += "     * $($_.Name) (Kaynak: $($_.Url))"
        }
    }

    # 5. Temiz modlar için öneriler
    if ($CleanMods.Count -gt 0 -and -not $securityRisk) {
        $recommendations += "`n[i] BİLGİ: Sisteminizde doğrulanmış temiz modlar bulundu ($($CleanMods.Count) adet)"
        $recommendations += "   - Bu modlar resmi kaynaklardan doğrulandı"
        $recommendations += "   - Modlarınız güncel mi? En son sürümleri kontrol edin"
        
        # Popüler modları vurgula
        $popularMods = $CleanMods | Where-Object { 
            $_.ModName -match "Sodium|Iris|OptiFine|Xaero|JourneyMap" 
        }
        
        if ($popularMods.Count -gt 0) {
            $recommendations += "`n   Popüler Modlarınız:"
            $popularMods | ForEach-Object {
                $recommendations += "     * $($_.ModName) (https://modrinth.com/mod/$($_.Slug))"
            }
        }
    }

    # Sonuç özeti
    Write-Host ""
    Write-Host "╔════════════════════════════════════════════════════════════════════╗" -ForegroundColor $colorHeader
    Write-Host "║                     GÜVENLİK DEĞERLENDİRMESİ                      ║" -ForegroundColor $colorHeader
    Write-Host "╠════════════════════════════════════════════════════════════════════╣" -ForegroundColor $colorHeader
    
    if ($securityRisk) {
        Write-Host "║  DURUM: " -NoNewline -ForegroundColor $colorHeader
        Write-Host "YÜKSEK RİSK                                                        " -ForegroundColor $colorError
        Write-Host "║                                                                  " -ForegroundColor $colorHeader
        Write-Host "║  Sisteminizde güvenlik riski oluşturabilecek modlar bulundu.     " -ForegroundColor $colorHeader
        Write-Host "║  Aşağıdaki önerileri dikkatle inceleyin.                         " -ForegroundColor $colorHeader
    }
    else {
        Write-Host "║  DURUM: " -NoNewline -ForegroundColor $colorHeader
        Write-Host "DÜŞÜK RİSK                                                         " -ForegroundColor $colorSuccess
        Write-Host "║                                                                  " -ForegroundColor $colorHeader
        Write-Host "║  Sisteminizde kritik risk oluşturan bir mod bulunamadı.          " -ForegroundColor $colorHeader
        Write-Host "║  Yine de bilinmeyen modları kontrol etmeniz önerilir.           " -ForegroundColor $colorHeader
    }
    
    Write-Host "╚════════════════════════════════════════════════════════════════════╝" -ForegroundColor $colorHeader

    # Önerileri göster
    if ($recommendations.Count -gt 0) {
        Write-Host ""
        Write-Host "ÖNERİLER VE SONRAKİ ADIMLAR:" -ForegroundColor $colorHeader
        $recommendations | ForEach-Object {
            if ($_ -match "\[!\]") {
                Write-Host $_ -ForegroundColor $colorError
            }
            elseif ($_ -match "\[i\]") {
                Write-Host $_ -ForegroundColor $colorInfo
            }
            else {
                Write-Host $_ -ForegroundColor $colorInfo
            }
        }
    }

    # Son kullanıcı mesajı
    Write-Host ""
    if ($securityRisk) {
        Write-Host "GÜVENLİK UYARISI: Bulunan riskler nedeniyle Minecraft sunucularına giriş yapmadan " -ForegroundColor $colorError -NoNewline
        Write-Host "önce bu sorunları çözmeniz şiddetle tavsiye edilir." -ForegroundColor $colorError
    }
    else {
        Write-Host "GÜVENLİK DURUMU: Mod koleksiyonunuz temiz görünüyor. " -ForegroundColor $colorSuccess -NoNewline
        Write-Host "Yine de düzenli kontroller yapmayı unutmayın." -ForegroundColor $colorInfo
    }
}

# Tarama sonunda bu fonksiyonu çağırın
# Start-MainScan fonksiyonunun en sonuna ekleyin:
Invoke-ModSecurityCheck -UnknownMods $unknownMods -BannedMods $foundBannedMods -CleanMods $cleanMods
