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
}

# 4. Sync Registry to GitHub
Set-Location $localRepoPath
git add README.md apps.json
git commit -m "Registry: Added $projectName v$version metadata"
git push origin main
Set-Location $PSScriptRoot
Write-Host "✅ Phase 1: App registered in README and apps.json."

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
*Metadata-First Sequence-Locked for Gobinda Workflow.*
