## **2025-03-30: CI/CD with GitHub Actions: Setting Up Self-Hosted Runner for C# WPF Project** 🔧🚀

### **Overview**  
This report covers the implementation of **CI/CD** using **GitHub Actions** for a C# WPF project. We explore the setup of a self-hosted runner, modifications to GitHub Actions workflows, unit tests creation, deployment strategies, and more. These steps ensure continuous integration and deployment of your application.

---

### **1. Introduction to CI/CD and GitHub Actions** 🎉

CI/CD is a vital practice for ensuring smooth development workflows. Below is an overview of CI/CD principles and how **GitHub Actions** facilitates automation:

- **Continuous Integration (CI)**: Ensures automatic testing of every change made to the codebase to avoid bugs.
- **Continuous Deployment (CD)**: Automatically deploys the application to production or a testing environment after the code passes tests.

**GitHub Actions** allows users to automate workflows within their repository, including building, testing, and deployment tasks triggered by specific events like code pushes.

---

### **2. Step-by-Step Setup** 🛠️

#### **2.1 Creating a GitHub Repository 🗂️**

If you haven’t created a GitHub repository, here’s how to do it:

1. Visit [GitHub](https://github.com) and log in.
2. Click on the **+** icon at the top-right and select **New repository**.
3. Name your repository (e.g., `MyWpfProject`) and set its visibility to **Public** or **Private** based on your needs.

---

#### **2.2 Setting Up a Self-Hosted Runner** 🖥️

A **self-hosted runner** allows running GitHub Actions on your hardware, offering more control and customization. Here’s how to set it up:

1. Open your repository and go to the **Settings** tab.
2. In the sidebar, under **Actions**, select **Runners**.
3. Click **Add self-hosted runner** and choose your operating system (Windows, macOS, or Linux).
4. Download the runner and follow the setup instructions (see sample scripts for Linux and Windows below).

Example for Linux:
```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux.tar.gz -L https://github.com/actions/runner/releases/download/v2.296.1/actions-runner-linux-x64-2.296.1.tar.gz
tar xzf ./actions-runner-linux.tar.gz
./config.sh --url https://github.com/YOUR_USER/YOUR_REPO --token YOUR_TOKEN
```

Example for Windows:
```powershell
mkdir actions-runner
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.296.1/actions-runner-win-x64-2.296.1.zip -OutFile actions-runner.zip
Expand-Archive actions-runner.zip -DestinationPath actions-runner
cd actions-runner
.\config.cmd --url https://github.com/YOUR_USER/YOUR_REPO --token YOUR_TOKEN
```

---

#### **2.3 Installing and Running the Self-Hosted Runner** 🏃‍♂️

After setting up the runner, it's time to run it:

1. **Run the Runner**:
   - On **Linux**:  
     ```bash
     ./run.sh
     ```
   - On **Windows**:  
     ```powershell
     .\run.cmd
     ```

2. **(Optional) Running as a Service**:
   - **Linux**:  
     ```bash
     sudo ./svc.sh install
     sudo ./svc.sh start
     ```
   - **Windows**:  
     ```powershell
     .\svc.cmd install
     .\svc.cmd start
     ```

---

### **3. Modifying the GitHub Actions Workflow** ⚙️

#### **3.1 Workflow Configuration** 📝

The **GitHub Actions workflow** defines the tasks to be performed when specific events occur, like code pushes. 

To integrate the self-hosted runner into the workflow:

1. Create a `.github/workflows/ci.yml` file.
2. Modify the `runs-on` field from `ubuntu-latest` to **self-hosted**.

Example `ci.yml`:
```yaml
name: CI for C# WPF Project

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: self-hosted

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set Up .NET SDK
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '6.0'

      - name: Restore Dependencies
        run: dotnet restore

      - name: Build Project
        run: dotnet build --configuration Release

      - name: Run Tests
        run: dotnet test --configuration Release
```

---

### **4. Creating Unit Tests for Your Project** 🧪

To ensure code functionality, unit tests are essential. Here's how to set them up in **Visual Studio**:

1. Right-click on the solution and select **Add > New Project**.
2. Choose **xUnit Test Project (.NET Core)** or **NUnit**.
3. Add the following sample test code:
```csharp
using Xunit;

public class ImageHelperTests
{
    [Fact]
    public void ConvertPngToBase64_ReturnsValidBase64String()
    {
        var result = ImageHelper.ConvertPngToBase64("test.png");
        Assert.NotNull(result);
        Assert.Matches("^[A-Za-z0-9+/=]+$", result);  // Ensure it's a valid Base64 string
    }
}
```

---

### **5. Testing and Monitoring the GitHub Actions Workflow** ✅

1. **Push Changes to GitHub** to trigger the workflow.
2. **Monitor the Workflow**:
   - Go to the **Actions** tab in GitHub to view the status of the workflow.
   - Logs for each step (checkout, build, test) will indicate whether it passed or failed.

3. **Debugging**:
   - If an issue arises, review the logs for specific error messages and address missing dependencies, permission issues, or incorrect environment variables.

---

### **6. Deployment to a Testing Environment** 🌍

After successful testing, we can extend the workflow to **deploy** the application to a testing or staging environment:

1. Add a deployment step in your workflow after the **build-and-test** job.
   
Example Deployment Step:
```yaml
jobs:
  deploy:
    runs-on: self-hosted
    needs: build-and-test
    steps:
      - name: Deploy to Remote Server
        run: |
          scp -r ./bin/Release/* user@remote-server:/path/to/deploy
```

---

### **7. UI Testing for WPF Applications** 🧪

For WPF applications, you can integrate **UI tests** into your CI pipeline using tools like **WinAppDriver**:

1. **Install WinAppDriver** on the self-hosted runner.
2. Create a UI Test Project in Visual Studio and use **WinAppDriver** for automating UI tests.
3. Add a step to run these UI tests in the workflow:
```yaml
- name: Run UI Tests
  run: dotnet test --configuration Release --filter "TestCategory=UI"
```

---

### **8. Managing Multiple Environments** 🌎

In production scenarios, you might need to deploy to **multiple environments** (development, testing, staging, production). GitHub Actions allows you to define different steps for each environment:

1. **Environment-Specific Jobs**: Use conditions to trigger deployments to different environments based on the branch or tag.

Example:
```yaml
jobs:
  deploy:
    runs-on: self-hosted
    needs: build-and-test
    if: github.ref == 'refs/heads/main'  # Only deploy from main branch
    steps:
      - name: Deploy to Staging
        run: scp ./bin/Release/* user@staging-server:/path/to/deploy
```

---

### **Conclusion**

This process ensures that your C# WPF project is automatically built, tested, and deployed with **CI/CD** through **GitHub Actions**. By setting up a self-hosted runner, creating unit tests, and deploying to multiple environments, you improve the efficiency and reliability of your development workflow.


---

It looks like you're asking for the **second day report** for **April 1, 2025**, alongside the GitHub Actions YAML file configuration.

Here's a **second-day report** following the structure you're aiming for:

---

## **2025-04-01: GitHub Actions Workflow Automation and Deployment** 🚀

### **Automating the CI/CD Pipeline with GitHub Actions** 🔧

On day two, we focused on refining the **GitHub Actions** workflow for the **C# WPF** project and implementing continuous integration (CI) strategies. Key highlights of the day include the setup for **.NET 8.0 SDK**, handling dependencies, building with **MSBuild**, running tests, and finally deploying to a production branch.

---

### **Step 1: GitHub Actions Workflow for C# WPF Project** 🔄

A well-structured **GitHub Actions** pipeline ensures your application is built, tested, and deployed automatically with each code push or pull request. Here’s a breakdown of the configuration:

#### **1.1 Workflow Setup** 📝

The YAML file defines the triggers that will run the pipeline:

```yaml
name: Potato Workflow

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

- **Triggering events**: This workflow triggers on **push** or **pull_request** events to the **main** branch, ensuring that changes in this primary branch automatically start the CI/CD pipeline.

#### **1.2 Job Configuration** ⚙️

The pipeline uses the **Windows** environment (`windows-latest`) as we're working with **Visual Studio** projects, particularly **C# WPF** and **C++** integration.

```yaml
jobs:
  build:
    runs-on: windows-latest

    strategy:
      matrix:
        dotnet-version: [8.0]
```

- **Matrix strategy**: This ensures that the **.NET 8.0 SDK** version is used for building and testing. This allows flexibility if multiple .NET versions are needed in future updates.

---

### **Step 2: Installing and Configuring Build Tools** 🛠️

#### **2.1 Installing Visual Studio Build Tools** 🧰

In the CI pipeline, the required **Visual Studio Build Tools** are installed to compile **C++ projects** as part of the build process:

```yaml
- name: Install Visual Studio Build Tools
  run: |
    choco install visualstudio2019buildtools --package-parameters "--add Microsoft.VisualStudio.Workload.MSBuildTools --includeRecommended"
```

- **Why Visual Studio Build Tools**: MSBuild is required for compiling projects like **C++**, which are often part of a WPF project when handling native code.

#### **2.2 Restoring .NET Dependencies** 🔄

The pipeline restores project dependencies before building:

```yaml
- name: Restore .NET dependencies
  run: dotnet restore
```

- This step ensures that all required **NuGet** packages are installed, preparing the environment for the build process.

---

### **Step 3: Build Process with MSBuild** 💻

In this section, we configure the **MSBuild** tool to build the project using the following command:

```yaml
- name: Build the project
  run: |
    & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /p:Configuration=Release Fw.sln
  shell: pwsh
```

- **MSBuild command**: The build is executed in **Release** mode using the **Fw.sln** solution file, compiling all the components needed for the WPF application.

---

### **Step 4: Running Unit Tests** 🧪

To ensure the quality of the code, **unit tests** are run during the pipeline:

```yaml
- name: Run tests
  run: dotnet test
```

- This step invokes **dotnet test**, running all **xUnit** or **NUnit** tests that are part of the project. It ensures the correctness of the functionality.

---

### **Step 5: Setting Up Git Credentials for Deployment** 🌍

GitHub Actions needs access to the repository to push changes to production. The following commands configure the necessary Git credentials:

```yaml
- name: Set up Git credentials
  run: |
    git config --global user.name "GitHub Actions"
    git config --global user.email "actions@github.com"
```

- **GitHub Actions identity**: By configuring GitHub Actions as the user, it avoids conflicts during pushes to the production branch.

---

### **Step 6: Deploying to Production** 🚀

The final step involves deploying changes to the **production branch**. The following steps ensure that the **main** branch’s changes are merged into **production**, and then pushed to the repository:

```yaml
- name: Fetch and checkout production branch
  run: |
    git fetch origin
    git checkout production

- name: Merge main into production
  run: |
    git pull origin production

- name: Push changes to production
  run: |
    git push origin production
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- **Fetching and merging**: These steps ensure that **main** is merged into **production**, keeping the production branch up to date.

---

```yml
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
          choco install visualstudio2019buildtools --package-parameters "--add Microsoft.VisualStudio.Workload.MSBuildTools --includeRecommended"


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
