# 🤖 AI-Ready Master Workflow: Full Auto Release (v4.0)

**Instructions for AI Agent:**
When this file is provided, you must execute the release in the following **STRICT ORDER**.

---

## 📝 Phase 1: Identity & Central Registry (Share First)
Before setting keys or building, extract app metadata and "Register" it in the Central Hub (`MyApps-Releases-Repo`). This shares the app info with the world first.

```powershell
# 1. Metadata Extraction (AI: Extract these from app/build.gradle.kts)
$projectName = "$(gh repo view --json name -q .name)"
$version = "1.0"
$category = "Utility" # AI: Ask user if unsure
$localRepoPath = "C:\Users\S S SORATHIYA\Desktop\gobinda\app\MyApps-Releases-Repo"
$readmePath = Join-Path $localRepoPath "README.md"
$appsJsonPath = Join-Path $localRepoPath "apps.json"

# 2. Update Central README Catalog
if (Test-Path $readmePath) {
    $readme = [System.IO.File]::ReadAllText($readmePath)
    if ($readme -notmatch $projectName) {
        $row = "| **$projectName** | $category | ![v](https://img.shields.io/badge/version-$version-blue) | Stable | [Download]($projectName-Latest.apk) |"
        $readme = $readme.Replace("| Download APK |", "| Download APK |`r`n$row")
        [System.IO.File]::WriteAllText($readmePath, $readme)
    }
}

# 3. Update Central apps.json (Website Registry)
if (Test-Path $appsJsonPath) {
    $apps = Get-Content $appsJsonPath | ConvertFrom-Json
    if ($apps.id -notcontains $projectName) {
        $newApp = @{
            id = $projectName
            name = $projectName
            category = $category
            version = "v$version"
            description = "High-performance automated release from Gobinda101 Labs."
            icon = "fas fa-cube"
            color = "blue"
            downloadUrl = "https://github.com/gobinda101/MyApps-Releases/raw/main/$projectName-Latest.apk"
            features = @("Automated Build", "Secure Signing")
        }
        $apps += $newApp
        $apps | ConvertTo-Json -Depth 10 | Out-File $appsJsonPath -Encoding utf8
    }

    # 3.1 Info System Registry (Powers the website Info button)
    $subAppDir = Join-Path $localRepoPath "apps\$projectName"
    $subReadme = Join-Path $subAppDir "README.md"
    if (-not (Test-Path $subAppDir)) { New-Item -ItemType Directory -Path $subAppDir }
    if (-not (Test-Path $subReadme)) {
        $template = "# 📦 $projectName`r`n`r`n## Features`r`n- Automated Release from Gobinda101 Labs`r`n- Securely signed binary`r`n`r`n## Download`r`n[Download APK](https://github.com/gobinda101/MyApps-Releases/raw/main/$projectName-Latest.apk)"
        [System.IO.File]::WriteAllText($subReadme, $template)
    }
}

# 4. Sync Registry to GitHub
Set-Location $localRepoPath
git add README.md apps.json apps/$projectName/README.md
git commit -m "Registry: Added $projectName v$version metadata and Info (README)"
git push origin main
Set-Location $PSScriptRoot
Write-Host "✅ Phase 1: App registered in README, apps.json, and Info system."
```

---

## 🔐 Phase 2: Security & Identity (Set Secrets)
Now that the app is registered, configure the secrets required to build it.

```powershell
# Configuration
$keystorePath = "release.keystore"
$keystorePass = "05052026"
$keyAlias     = "brainreminder"

# Execution
if (Test-Path $keystorePath) {
    $base64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes($keystorePath))
    gh secret set ANDROID_KEYSTORE_BASE64 --body "$base64"
    gh secret set ANDROID_KEYSTORE_PASSWORD --body "$keystorePass"
    gh secret set ANDROID_KEY_ALIAS --body "$keyAlias"
    gh secret set ANDROID_KEY_PASSWORD --body "$keystorePass"
    gh secret set RELEASE_REPO_TOKEN --body "$(gh auth token)"
    Write-Host "✅ Phase 2: GitHub Secrets configured."
}
```

---

## 🚀 Phase 3: Project Configuration
Configure the project files for the automation.

**1. Create `.github/workflows/release.yml`**:
Must include:
- `paths-ignore`: Ignore `.md` and `.gitignore` files to save build minutes.
- `chmod +x gradlew`
- Keystore decoding from secrets.
- **Sync APK to Central Hub**: Add a step to clone `MyApps-Releases`, copy the new APK, and push it.

**2. Update `app/build.gradle.kts`**:
Ensure `signingConfigs` use environment variables with fallback to `../release.keystore`.

---

## 🏁 Phase 4: Final Push (Deploy)
Trigger the actual build and cloud release.

1. `git add .`
2. `git commit -m "Release System Integrated: Phases 1-3 Complete"`
3. `git push origin main`

---

## 🔍 Phase 5: Global Integrity Check (Final Stage)
Run this check once the release is complete to ensure all links and registry entries are synchronized across the entire Hub.

```powershell
# 1. Load Registry & Local State
$localRepoPath = "C:\Users\S S SORATHIYA\Desktop\gobinda\app\MyApps-Releases-Repo"
$appsJsonPath = Join-Path $localRepoPath "apps.json"
$readmePath = Join-Path $localRepoPath "README.md"
$apps = Get-Content $appsJsonPath | ConvertFrom-Json
$readme = Get-Content $readmePath

Write-Host "🚀 Starting Global Integrity Check..." -ForegroundColor Cyan

# 2. Validation Loop
foreach ($app in $apps) {
    Write-Host "-------------------------------------------"
    Write-Host "Checking: $($app.name) ($($app.id))" -ForegroundColor Yellow

    # Check 1: APK Physical Existence
    $apkName = "$($app.id)-Latest.apk"
    $apkPath = Join-Path $localRepoPath $apkName
    if (Test-Path $apkPath) {
        Write-Host "  ✅ APK Source: Found ($apkName)" -ForegroundColor Green
    } else {
        Write-Host "  ❌ APK Source: MISSING ($apkName)" -ForegroundColor Red
    }

    # Check 2: apps.json vs README.md Link Consistency
    $expectedUrl = "https://github.com/gobinda101/MyApps-Releases/raw/main/$apkName"
    if ($app.downloadUrl -eq $expectedUrl) {
        Write-Host "  ✅ apps.json URL: Correct" -ForegroundColor Green
    } else {
        Write-Host "  ⚠️ apps.json URL: Mismatch (Found: $($app.downloadUrl))" -ForegroundColor Yellow
    }

    if ($readme -match [regex]::Escape($expectedUrl)) {
        Write-Host "  ✅ README.md Link: Synchronized" -ForegroundColor Green
    } else {
        Write-Host "  ❌ README.md Link: Broken or Outdated" -ForegroundColor Red
    }

    # Check 3: Sub-app README Sync
    $subReadmePath = Join-Path $localRepoPath "apps\$($app.id)\README.md"
    if (Test-Path $subReadmePath) {
        $subReadme = Get-Content $subReadmePath
        if ($subReadme -match [regex]::Escape($expectedUrl)) {
            Write-Host "  ✅ App README: Verified" -ForegroundColor Green
        } else {
            Write-Host "  ⚠️ App README: Needs Update" -ForegroundColor Yellow
        }
    }
}

Write-Host "-------------------------------------------"
Write-Host "🏁 Check Complete. Ensure all [❌] and [⚠️] are resolved before publicizing." -ForegroundColor Cyan
```

---
*Metadata-First Sequence-Locked for Gobinda Workflow.*
