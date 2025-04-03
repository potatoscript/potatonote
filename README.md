# 🚀 **Work Daily Report**   

📌 [View Tabbed Menu in Wiki](https://github.com/heartlanguage2024/heartlanguage2024/wiki/TabbedMenu)

##### menu
## 📌 Table of Contents  
| Date       | Title |
|------------|------------------------------------------------|
| 2025-04-03 | [Setup and run test on action-runner](#setup-and-run-test-on-action-runner) |
| 2025-04-02 | [Solving the workflow setting for the C++ project](#solving-the-workflow-setting-for-the-c-project) |
| 2025-04-01 | [GitHub Actions Workflow Automation and Deployment](#github-actions-workflow-automation-and-deployment) |
| 2025-03-30 | [CI/CD with GitHub Actions: Setting Up Self-Hosted Runner for C# WPF Project](#ci-cd-with-github-actions-setting-up-self-hosted-runner-for-c-wpf-project) |
| 2025-03-28 | [Create the bat file to auto start the runner](#bat-auto-runner) |

---

###### setup-and-run-test-on-action-runner
###### [back](#menu) 
<details>
<summary><strong>🍓 **2025-04-03: Setup and run test on action-runner** 🔧🚀</strong></summary>

# Fixing VCTargetsPath Error in Visual Studio

## Issue Description
If you encounter the following error when building your project in Visual Studio:

```
The imported project "C:\Microsoft\VC\v170\Microsoft.Cpp.Default.props" was not found.
Confirm that the expression in the Import declaration "$(VCTargetsPath)\Microsoft.Cpp.Default.props",
which evaluated to "C:\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v170\Microsoft.Cpp.Default.props",
is correct, and that the file exists on disk.
```

This typically happens because the `VCTargetsPath` environment variable is incorrectly set or missing a trailing backslash (`\`).

### **Why is `VCTargetsPath` Important in a WPF Project?**  
Even though WPF primarily uses C# and XAML, you might have **C++/CLI (Common Language Infrastructure) components** in your WPF project. This happens if:  
1. **Your WPF project references a C++/CLI project** (e.g., for performance reasons or legacy code).  
2. **You are using a native C++ library with P/Invoke or interop** (e.g., Open CASCADE in your case).  
3. **Your solution includes both C# and C++ projects**, such as a mixed-language setup.

When you build your project, MSBuild relies on `VCTargetsPath` to locate these C++ build definitions. If the path is incorrect or missing, Visual Studio **cannot find the necessary build scripts**, leading to errors like:  
```
The imported project "C:\Microsoft\VC\v170\Microsoft.Cpp.Default.props" was not found.
```

## Solution: Set `VCTargetsPath` Correctly
To fix this issue, ensure that `VCTargetsPath` is set correctly in the system environment variables.

### Steps to Fix:
1. Open **Run** (`Win + R`), type `sysdm.cpl`, and press **Enter**.
2. Go to the **Advanced** tab and click on **Environment Variables**.
3. Under **System Variables**, look for `VCTargetsPath`:
   - If it exists, **edit it** and ensure it is set to:
     ```
     C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v170\
     ```
     (Note the trailing `\` at the end.)
   - If it doesn’t exist, **create a new system variable** with the above value.
4. Click **OK** and restart your computer.

### Additional Steps if the Issue Persists:
- Delete the `.vs`, `bin`, and `obj` folders in your project directory.
- Open Visual Studio and **Rebuild Solution**.
- If the error still occurs, manually edit `JwwControl.vcxproj` and ensure that all references to `VCTargetsPath` are correct.

## Summary
- Ensure `VCTargetsPath` ends with `\`.
- Update the environment variable if necessary.
- Clean and rebuild the project in Visual Studio.



</details>
---

###### solving-the-workflow-setting-for-the-c-project
###### [back](#menu) 
<details>
<summary><strong>🍓 **2025-04-02: Solving the workflow setting for the C++ project** 🔧🚀</strong></summary>

### **Issue**  
The following workflow still encounters an error:  

```
The error is caused by missing Visual Studio C++ build components, specifically the Microsoft.Cpp.Default.props file. Your workflow attempts to install Visual Studio Build Tools using Chocolatey but does not correctly configure the environment to use the installed tools.
```

### **Workflow Configuration**  
```yaml
name: Potato Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    # ✅ # Use Windows since you're working with Visual Studio projects
    runs-on: windows-latest  

    strategy:
      matrix:
        dotnet-version: [8.0]

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      # ✅ Set up .NET SDK (No need for VS installation)
      - name: Set up .NET SDK
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ matrix.dotnet-version }}

      # Install Visual Studio Build Tools
      - name: Install Visual Studio Build Tools
        run: |
          choco install visualstudio2022buildtools --package-parameters "--add Microsoft.VisualStudio.Workload.VC --includeRecommended"
          choco install visualstudio2022buildtools --package-parameters "--add Microsoft.VisualStudio.Workload.MSBuildTools --includeRecommended"

      # Initialize Visual Studio Environment Variables
      - name: Initialize Visual Studio Environment Variables
        run: |
          & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\Common7\Tools\VsDevCmd.bat"


      # Clean the solution (optional)
      - name: Clean MSBuild Cache
        run: |
          & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /t:Clean Fw.sln
        shell: pwsh

      # ✅ Restore dependencies
      - name: Restore .NET dependencies
        run: dotnet restore

      # Build solution using MSBuild (required for C++ projects) ✖
      #- name: Build solution using MSBuild
      #  run: |
      #    msbuild D:\a\FwCAD\FwCAD\Fw.sln /p:Configuration=Release

      # ✅ Restore dependencies using MSBuild (No installation needed)
      - name: Restore dependencies (MSBuild)
        run: |
          & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /t:restore /p:Configuration=Release Fw.sln
        shell: pwsh

      # ✅ Build the project using preinstalled MSBuild
      - name: Build the project
        run: |
          & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /p:Configuration=Release Fw.sln
        shell: pwsh
      
      # Build the solution ✖
      #- name: Build solution
      #  run: dotnet build

      # ✅ Run tests (if applicable)
      - name: Run tests
        run: dotnet test


      # Set up Git credentials
      - name: Set up Git credentials
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "actions@github.com"

      # Fetch and checkout production branch
      - name: Fetch and checkout production branch
        run: |
          git fetch origin
          git checkout production

      # Merge main into production
      - name: Merge main into production
        run: |
          git pull origin production

      # Push changes to production
      - name: Push changes to production
        run: |
          git push origin production
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

## **🔍 Step 1: Verify Your Installed Components**
1. **Open Visual Studio Installer**  
   - Press `Win + S`, search for **Visual Studio Installer**, and open it.  
   - Click **Modify** on **Visual Studio 2022 Community**.  

2. **Ensure the Following Are Installed:**  
   In the **Workloads** tab:  
   ✅ `Desktop development with C++` (if not checked, enable it).  

   In the **Individual Components** tab, **check the following** under “Compilers, build tools, and runtimes”:  
   - ✅ **MSVC v143 - VS 2022 C++ x64/x86 build tools**  
   - ✅ **Windows 10 SDK (latest installed)**  
   - ✅ **Windows 11 SDK (latest installed)**  
   - ✅ **C++ CMake tools for Windows**  
   - ✅ **C++ ATL for v143**  
   - ✅ **C++ MFC for v143**  
   - ✅ **C++ CMake tools for Windows**  

   📌 **Make sure these are installed before proceeding.** Click **Modify** to install missing components.

---

## **🔍 Step 2: Manually Check Installed Files**
1. Open **File Explorer** and go to:
   ```
   C:\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\
   ```
   You should see `v150`, `v160`, and `v170`, but **not `v170Platforms`**.

2. Check inside the `v170` folder:
   ```
   C:\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v170\
   ```
   - Do you see `Microsoft.Cpp.Default.props`?  
   - Do you see `BuildCustomizations`?  

   📌 **Let me know what files exist there!**

---

## **🔧 Step 3: Manually Copy Missing Files**
If **`v170Platforms`** is still missing after reinstalling the components:  

### **1️⃣ Copy from an Existing Version**
Check if `v160Platforms` or `v150Platforms` exist:  
```
C:\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v160Platforms\
C:\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v150Platforms\
```
If **either exists**, copy the entire folder and rename it to `v170Platforms`.

### **2️⃣ Reinstall VS Build Tools (Last Resort)**
If **you do not have `v150Platforms` or `v160Platforms`**, reinstall VS Build Tools:  
1. **Download Visual Studio Build Tools** from:  
   👉 [https://visualstudio.microsoft.com/visual-cpp-build-tools/](https://visualstudio.microsoft.com/visual-cpp-build-tools/)  
2. **Run the installer**, select **C++ Desktop Development**, and install all required components.  
3. **Restart your computer** and check if `v170Platforms` appears.  

---

## **📢 Final Steps**
After ensuring `v170Platforms` exists, run:
```sh
msbuild /t:clean
msbuild JwwControl.vcxproj
```
</details>

---

###### ci-cd-with-github-actions-setting-up-self-hosted-runner-for-c-wpf-project
###### [back](#menu) 
<details>
<summary><strong>🍓 2025-03-30: CI/CD with GitHub Actions: Setting Up Self-Hosted Runner for C# WPF Project 🔧🚀</strong></summary>

### **Overview**  
This guide covers implementing **CI/CD** using **GitHub Actions** for a **C# WPF** project. We will:  
✅ Set up a **self-hosted runner**  
✅ Modify **GitHub Actions workflows**  
✅ Create **unit tests**  
✅ Implement **deployment strategies**  

---

### **1. Introduction to CI/CD and GitHub Actions** 🎉  

**CI/CD** is an essential practice that enhances software development efficiency.  
💡 **Continuous Integration (CI)**: Automatically tests each code change to prevent bugs.  
💡 **Continuous Deployment (CD)**: Deploys changes to production or a testing environment once tests pass.  

📖 [Read More](#ci-cd-with-github-actions-setting-up-self-hosted-runner-for-c-wpf-project)  

---

### **2. Setting Up a Self-Hosted Runner** 🖥️  

A **self-hosted runner** allows GitHub Actions workflows to execute on your own machine.  

#### **📌 Steps:**  
1️⃣ Navigate to your GitHub repository → **Settings** → **Actions** → **Runners**  
2️⃣ Click **New self-hosted runner** and follow the installation steps  
3️⃣ Run the setup script and start the runner  

```bash
./config.sh --url https://github.com/your-repo --token YOUR_ACCESS_TOKEN
./run.sh
```

🚀 Your self-hosted runner is now ready to execute workflows!

---

### **3. GitHub Actions Workflow Configuration** ⚙️  

Create a **workflow YAML** file for building and testing your WPF project.  

📄 **`.github/workflows/ci.yml`**  

```yaml
name: Build and Test WPF App

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: self-hosted

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Restore Dependencies
        run: dotnet restore

      - name: Build Solution
        run: dotnet build --configuration Release --no-restore

      - name: Run Tests
        run: dotnet test --no-restore --verbosity normal
```
</details>

---

###### github-actions-workflow-automation-and-deployment
###### [back](#menu)
<details>
<summary><strong>🍓 2025-04-01: GitHub Actions Workflow Automation and Deployment 🚀  </strong></summary>

### **Automating the CI/CD Pipeline**  

On day two, we improved the **GitHub Actions workflow** for our **C# WPF project**.  

✅ **Setup .NET 8.0 SDK**  
✅ **Handle dependencies**  
✅ **Build with MSBuild**  
✅ **Run unit tests**  
✅ **Deploy to production**  

---

### **Deploying the Application** 🚀  

Modify the workflow to **automatically publish** the application.  

📄 **`.github/workflows/deploy.yml`**  

```yaml
name: Deploy WPF App

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Publish Application
        run: dotnet publish -c Release -o ./publish

      - name: Deploy to Server
        run: |
          scp -r ./publish user@your-server:/var/www/wpf-app
```

🚀 Your application will **automatically deploy** when changes are pushed to `main`!  

📖 [Read More](#github-actions-workflow-automation-and-deployment)  

---

## 🎯 **Conclusion**  

By following these steps, we've successfully:  
✅ Configured a **self-hosted runner**  
✅ Set up **GitHub Actions workflows**  
✅ Implemented **unit testing**  
✅ Automated **deployment**  

🔗 **Next Steps:** Optimize CI/CD pipelines, add error logging, and refine deployment strategies.  

📢 **Need help?** Open an issue or contribute to this repository! 🚀  

</details>

---

###### bat-auto-runner
###### [back](#menu)
<details>
<summary><strong>🍓 2025-03-28: Automating Service Startup on Windows Boot: Creating `start_services.bat` 🚀 </strong></summary>

## 📌 Introduction
Managing multiple services manually every time you start your Windows system can be tedious. This tutorial will guide you through creating a batch script (`start_services.bat`) that automates the startup of **Jenkins**, **GitHub Actions Runner**, **ngrok**, and **Git operations**.

By the end of this tutorial, your system will automatically:
- Start Jenkins in the background.
- Ensure GitHub Actions Runner is running.
- Start ngrok and update the workflow with the new public URL.
- Pull the latest code from your Git repository and push any necessary changes.

---

## 🛠️ Prerequisites
Before proceeding, ensure you have the following installed and configured:
1. **Java** (for Jenkins)
2. **Jenkins** (`jenkins.war` should be placed in a known directory)
3. **GitHub Actions Runner** (already configured in `C:\Projects\actions-runner`)
4. **ngrok** (installed and accessible via command line)
5. **Git** (configured with SSH or HTTPS authentication)

---

## 📝 Creating the `start_services.bat` Script
Create a new file named `start_services.bat` and paste the following script:

```bat
@echo off
echo Starting Jenkins...

:: Start Jenkins in the background
start /B java -jar jenkins.war --httpPort=8080 --httpListenAddress=0.0.0.0
echo Jenkins started.

:: Wait a few seconds for Jenkins to start
timeout /t 5

:: Check if Actions Runner is running
tasklist | find /i "Runner.Listener.exe" > nul
if %errorlevel% neq 0 (
    echo GitHub Actions Runner is not running. Starting now...
    cd /d "C:\Projects\actions-runner"
    start /B run.cmd
) else (
    echo GitHub Actions Runner is already running.
)

:: Start ngrok and capture the public URL
echo Starting ngrok...
start /B ngrok http 8080 > nul 2>&1

:: Wait for ngrok to initialize
timeout /t 5

:: Fetch the ngrok URL using PowerShell
for /f "delims=" %%A in ('powershell -Command "(Invoke-RestMethod -Uri 'http://127.0.0.1:4040/api/tunnels').tunnels[0].public_url"') do set NGROK_URL=%%A

:: Trim the quotation marks from the URL
set NGROK_URL=%NGROK_URL:"=%

echo ngrok URL: %NGROK_URL%

:: Update the ngrok URL in `potato.yml`
powershell -Command "(Get-Content C:\Projects\ConvertImageToBase64\.github\workflows\potato.yml) -replace 'https://.*?\.ngrok-free\.app', '%NGROK_URL%' | Set-Content C:\Projects\ConvertImageToBase64\.github\workflows\potato.yml"

:: Pull the latest changes from origin and merge with local changes
echo Pulling latest changes from origin main...
cd /d C:\Projects\ConvertImageToBase64
git fetch origin
git pull origin main

:: Check if there was an error while pulling (e.g., merge conflicts)
if %errorlevel% neq 0 (
    echo There was an error while pulling from the remote repository. Please resolve any conflicts and try again.
    pause
    exit /b
)

:: Check if a merge is in progress (MERGE_HEAD exists)
if exist .git\MERGE_HEAD (
    echo Merge is in progress. Committing the merge...
    git commit --no-edit
)

:: Check for uncommitted changes (before merge or after conflict resolution)
git diff --exit-code > nul
if %errorlevel% neq 0 (
    echo Uncommitted changes detected. Committing changes...
    git add .
    git commit -m "Auto-commit changes"
)

:: Push the merged changes to GitHub
git push origin main

echo All services are running. The GitHub Actions workflow has been updated.
pause
```

---

## 🔥 Understanding the Script
Here’s a breakdown of what this script does:

### **1. Start Jenkins**
- Runs Jenkins in the background on port 8080.
- Uses `start /B` to keep it running without opening a separate window.

### **2. Ensure GitHub Actions Runner is Running**
- Checks if `Runner.Listener.exe` is running.
- If not, navigates to `C:\Projects\actions-runner` and starts it.

### **3. Start ngrok & Capture Public URL**
- Runs ngrok in the background.
- Waits for initialization and fetches the ngrok public URL.
- Uses PowerShell to parse the JSON response and extract the URL.

### **4. Update GitHub Actions Workflow with the New ngrok URL**
- Updates `potato.yml` in the GitHub repository.
- Uses PowerShell to replace the old ngrok URL with the new one.

### **5. Synchronize with GitHub Repository**
- Pulls the latest changes from `origin/main`.
- Handles merge conflicts if any.
- Commits and pushes changes automatically.

---

## 🚀 Automating the Script Execution on Windows Startup
To make this script run every time Windows starts, follow these steps:

### **Step 1: Create a Shortcut**
1. Right-click on `start_services.bat` and select **Create Shortcut**.
2. Rename the shortcut to something like **Start Services**.

### **Step 2: Add to Startup Folder**
1. Press `Win + R`, type `shell:startup`, and hit **Enter**.
2. Copy the shortcut into the opened folder.

Now, every time Windows starts, your script will execute automatically! 🎉

</details>

---
