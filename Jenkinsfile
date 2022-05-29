pipeline {
  agent { label "windows" }
  environment {
    GITHUB_TOKEN = credentials("github-publish-token")
  }
  stages {
    // Configure git
    stage("Configure git") {
      steps {
        bat "git config user.email burt-macklin@jenkins"
        bat 'git config user.name "Agent Burt Macklin"'
      }
    }
    // Determine build & publish flags for branch
    stage("Setup bleeding edge environment") {
      when {
        anyOf {
          branch "main"
          tag "*bleeding-edge*"
        }
      }
      steps {
        script {
          env.ARTIFACT_CACHES = "bleeding-edge"
          env.AUTO_PUBLISH = "true"
          env.BUILD_CONFIG = "debug"
          env.IS_PRERELEASE = "true"
          env.JOB_CACHE = "bleeding-edge"
          env.TAG_PREFIX = "Unstable Release"
        }
      }
    }
    stage("Setup experimental environment") {
      when { branch "experimental" }
      steps {
        script {
          env.ARTIFACT_CACHES = "experimental"
          env.AUTO_PUBLISH = "false"
          env.BUILD_CONFIG = "debug"
          env.IS_PRERELEASE = "true"
          env.JOB_CACHE = "experimental"
          env.TAG_PREFIX = "Experimental Release"
        }
      }
    }
    stage("Setup pre-release environment") {
      when { branch "prerelease" }
      steps {
        script {
          env.ARTIFACT_CACHES = "bleeding-edge,prerelease"
          env.AUTO_PUBLISH = "true"
          env.BUILD_CONFIG = "release"
          env.IS_PRERELEASE = "true"
          env.JOB_CACHE = "prerelease"
          env.TAG_PREFIX = "Pre-Release"
        }
      }
    }
    stage("Setup release environment") {
      when { branch "release" }
      steps {
        script {
          env.ARTIFACT_CACHES = "bleeding-edge,prerelease,release"
          env.AUTO_PUBLISH = "true"
          env.BUILD_CONFIG = "release"
          env.IS_PRERELEASE = "false"
          env.JOB_CACHE = "release"
          env.TAG_PREFIX = "Stable Release"
        }
      }
    }
    // Determine the version number
    stage("Calculate semver") {
      steps {
        bat "gitversion /output buildserver"
        script {
          def props = readProperties file: "gitversion.properties"
          env.GITVERSION_SEMVER = props.GitVersion_SemVer
          env.PUBLISH_TAG = "v${props.GitVersion_SemVer}"
        }
      }
    }
    // Packaging
    stage("Package artifacts") {
      steps {
        powershell '''
          Write-Output "Gathering artifacts to package..."
          if (Test-Path -Path "./artifacts") {
            Remove-Item -Path "./artifacts/*" -Recurse -Force
          }
          New-Item -Path . -Name "artifacts" -ItemType Directory -Force
          New-Item -Path ./artifacts -Name "GameData" -ItemType Directory -Force
          New-Item -Path ./artifacts/GameData -Name "UmbraSpaceIndustries" -ItemType Directory -Force
          Copy-Item -Path ./*.txt -Destination ./artifacts

          Write-Output "Pulling in cached dependencies..."
          $DriveRoot = Split-Path -Path $env:WORKSPACE -Qualifier
          $ReleasePath = Join-Path -Path $DriveRoot -ChildPath "usi-releases"
          $ModuleManagerPath = Join-Path -Path $ReleasePath -ChildPath "ModuleManager"
          $CachePath = Join-Path -Path $ReleasePath -ChildPath $env:JOB_CACHE
          $UsiCachePath = Join-Path -Path $CachePath -ChildPath "UmbraSpaceIndustries"
          Copy-Item -Path $ModuleManagerPath/* -Destination ./artifacts/GameData
          Copy-Item -Path (Join-Path -Path $CachePath -ChildPath "000_USITools") -Destination ./artifacts/GameData -Recurse
          Copy-Item -Path (Join-Path -Path $CachePath -ChildPath "CommunityCategoryKit") -Destination ./artifacts/GameData -Recurse
          Copy-Item -Path (Join-Path -Path $CachePath -ChildPath "CommunityResourcePack") -Destination ./artifacts/GameData -Recurse
          Copy-Item -Path (Join-Path -Path $CachePath -ChildPath "Firespitter") -Destination ./artifacts/GameData -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Akita") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "ART") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "ExpPack") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "FTT") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "FX") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Karbonite") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "KarbonitePlus") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Karibou") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Konstruction") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Kontainers") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "LifeSupport") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Malemute") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "MKS") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "NuclearRockets") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "Orion") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "ReactorPack") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "SoundingRockets") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "SrvPack") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "SubPack") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "USICore") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "WarpDrive") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
          Copy-Item -Path (Join-Path -Path $UsiCachePath -ChildPath "WOLF") -Destination ./artifacts/GameData/UmbraSpaceIndustries -Recurse
        '''
        script {
          env.ARCHIVE_FILENAME = "UmbraSpaceIndustries_${env.GITVERSION_SEMVER}.zip"
          zip dir: "artifacts", zipFile: "${env.ARCHIVE_FILENAME}", archive: true
        }
      }
    }
    // Tag commit, if necessary
    stage("Tag commit") {
      steps {
        powershell '''
          Write-Output "Looking for tag $env:PUBLISH_TAG..."
          $tagFound = git tag -l "$env:PUBLISH_TAG"
          if ( $tagFound -ne $env:PUBLISH_TAG )
          {
            Write-Output "Tag not found. Creating tag..."
            git tag -a $env:PUBLISH_TAG -m "$env:TAG_PREFIX $env:GITVERSION_SEMVER"
            Write-Output "Pushing tag to GitHub..."
            git push --tags
          }
        '''
      }
    }
    // Push artifacts to GitHub
    stage("Push release artifacts to GitHub") {
      when {
        environment name: "AUTO_PUBLISH", value: "true"
      }
      steps {
        powershell '''
          echo "Creating release on GitHub..."
          $RepoUrl = [uri]$env:GIT_URL
          $RepoPath = $RepoUrl.LocalPath.Replace(".git", "")
          $Url = "https://api.github.com/repos$RepoPath/releases"
          $Headers = @{
            "Accept" = "application/vnd.github.v3+json"
            "Authorization" = "token $env:GITHUB_TOKEN"
          }
          $Body = @{
            tag_name = "$env:PUBLISH_TAG"
            name = "$env:TAG_PREFIX $env:GITVERSION_SEMVER"
            prerelease = ($env:IS_PRERELEASE -eq "true")
          }
          $Json = ConvertTo-Json $Body
          $Response = Invoke-WebRequest -Method Post -Uri $Url -Headers $Headers `
              -ContentType "application/json" -Body $Json -UseBasicParsing
          if ( $Response.StatusCode -ge 300 ) {
            Write-Output "Could not create GitHub Release"
            Write-Output "Status Code: $Response.StatusCode"
            Write-Output $Response.Content
            throw $Response.StatusCode
          } else {
            Write-Output "GitHub API replied with status code: $($Response.StatusCode)"
            Write-Output ($Response.Content | ConvertFrom-Json | ConvertTo-Json -Depth 100)
          }

          echo "Uploading artifacts to GitHub..."
          $ReleaseMetadata = ConvertFrom-Json $Response.Content
          $UploadUrl = $ReleaseMetadata | Select -ExpandProperty "upload_url"
          $UploadUrl = $UploadUrl.Replace("{?name,label}", "?name=$env:ARCHIVE_FILENAME")
          $Response = Invoke-WebRequest -Method Post -Uri $UploadUrl -Headers $Headers `
              -ContentType "application/zip" -InFile $env:ARCHIVE_FILENAME -UseBasicParsing
          if ( $Response.StatusCode -ge 300 ) {
            Write-Output "Could not upload artifacts to GitHub"
            Write-Output "Status Code: $Response.StatusCode"
            Write-Output $Response.Content
            throw $Response.StatusCode
          } else {
            Write-Output "GitHub API replied with status code: $($Response.StatusCode)"
            Write-Output ($Response.Content | ConvertFrom-Json | ConvertTo-Json -Depth 100)
          }
        '''
      }
    }
  }
}
