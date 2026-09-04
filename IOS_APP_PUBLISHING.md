# iOS App Publishing Workflow

This repo is the official place for the no-Mac Expo iOS build workflow.

## What This Does

- Build and test the Expo app locally with Expo Go.
- Push the app to GitHub.
- Let GitHub Actions use a macOS runner to build an unsigned iOS IPA.
- Download the IPA artifact.
- Sign and sideload it with Sideloadly or AltStore.

## Local Test

```sh
npm install
npm run start
```

Open the app in Expo Go and test it before building the IPA.

## Required GitHub Secrets

Expo public environment values are embedded at build time. Add these in GitHub:

```text
Settings -> Secrets and variables -> Actions -> New repository secret
```

Required for this app:

```text
EXPO_PUBLIC_SUPABASE_URL
EXPO_PUBLIC_SUPABASE_ANON_KEY
EXPO_PUBLIC_VOLUNTEER_NAME
```

Do not commit `.env` to this public repo. Keep `.env.example` only.

## Build IPA Automatically

Push to `master` or `main`:

```sh
git add -A
git commit -m "Update app"
git push
```

The workflow runs automatically:

```text
.github/workflows/build-ios-unsigned.yml
```

It creates this artifact:

```text
unsigned-ipa
```

Inside the artifact:

```text
<AppName>-unsigned.ipa
```

## Trigger Build Without Code Changes

```sh
git commit --allow-empty -m "Trigger iOS IPA build"
git push
```

## Trigger And Download With GitHub CLI

```sh
gh auth login
gh workflow run build-ios-unsigned.yml --repo saaheerpurav/ios-app --ref master
gh run watch --repo saaheerpurav/ios-app
mkdir -p dist
gh run download --repo saaheerpurav/ios-app --name unsigned-ipa --dir dist
```

PowerShell:

```powershell
$Owner = "saaheerpurav"
$Repo = "ios-app"
$Workflow = "build-ios-unsigned.yml"
$Branch = "master"

gh workflow run $Workflow --repo "$Owner/$Repo" --ref $Branch
$RunId = (gh run list --repo "$Owner/$Repo" --workflow $Workflow --limit 1 --json databaseId | ConvertFrom-Json)[0].databaseId
gh run watch $RunId --repo "$Owner/$Repo"
New-Item -ItemType Directory -Force -Path dist | Out-Null
gh run download $RunId --repo "$Owner/$Repo" --name unsigned-ipa --dir dist
```

## Sideload

The IPA from GitHub Actions is unsigned. Sign and install it on a non-jailbroken iPhone with Sideloadly or AltStore.

Free Apple ID installs usually expire after 7 days.
