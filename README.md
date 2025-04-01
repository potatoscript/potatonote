# 2025-03-30:
### **CI/CD with GitHub Actions: Setting Up Self-Hosted Runner for C# WPF Project** 🔧🚀

### **Introduction to CI/CD and GitHub Actions** 🎉

In this section, we'll start with a brief overview of **CI/CD** and **GitHub Actions**:

- **Continuous Integration (CI)**: This process ensures that every change made to the codebase is automatically tested to avoid bugs.
- **Continuous Deployment (CD)**: After testing, your application can be automatically deployed to production or another environment.

**GitHub Actions** is a platform within GitHub that allows you to automate these processes. With Actions, you can create workflows to automatically build, test, and deploy your project when you push changes.

---

### **Step 1: Set Up Your Self-Hosted Runner** 🖥️

#### **Create Your GitHub Repository 🗂️**

If you haven’t already created a GitHub repository, follow these steps:

1. Go to [GitHub](https://github.com).
2. Log in to your account (or sign up).
3. Click the **+** icon at the top right and select **New repository**.
4. Name your repository (e.g., `MyWpfProject`) and create it as **Public** or **Private**, depending on your preference.

---

#### **Set Up a Self-Hosted Runner 🔨**

A **self-hosted runner** allows you to run GitHub Actions on your own hardware instead of GitHub’s infrastructure. This gives you more control and is useful for custom configurations or hardware-specific tasks.

1. **Go to Your GitHub Repository**:
   - Open your repository on GitHub.

2. **Access the Settings**:
   - Click on the **Settings** tab at the top of your repository.

3. **Select Actions**:
   - In the left sidebar under **Actions**, click on **Runners**.

4. **Add Self-Hosted Runner**:
   - Click the **Add self-hosted runner** button.
   - You'll see options for Windows, macOS, and Linux.

5. **Download the Runner**:
   - Choose your runner's **operating system** (Windows, macOS, or Linux).
   - GitHub will provide a script for you to run on your machine. Copy the **download** link.

   - **For Linux (example)**:

     ```bash
     mkdir actions-runner && cd actions-runner
     curl -o actions-runner-linux.tar.gz -L https://github.com/actions/runner/releases/download/v2.296.1/actions-runner-linux-x64-2.296.1.tar.gz
     tar xzf ./actions-runner-linux.tar.gz
     ./config.sh --url https://github.com/YOUR_USER/YOUR_REPO --token YOUR_TOKEN
     ```

   - **For Windows (example)**:

     ```powershell
     mkdir actions-runner
     Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.296.1/actions-runner-win-x64-2.296.1.zip -OutFile actions-runner.zip
     Expand-Archive actions-runner.zip -DestinationPath actions-runner
     cd actions-runner
     .\config.cmd --url https://github.com/YOUR_USER/YOUR_REPO --token YOUR_TOKEN
     ```

   - Replace **`YOUR_USER`** with your GitHub username, **`YOUR_REPO`** with your repository name, and **`YOUR_TOKEN`** with a **personal access token** (you can generate one [here](https://github.com/settings/tokens)).

---

### **Step 2: Install and Run the Self-Hosted Runner** 🏃‍♂️

Once you have the self-hosted runner configured, let’s move on to running it.

#### **Configuring the Runner 🛠️**

Once you’ve downloaded and configured the runner:

1. **Run the Runner**:
   - **For Linux**:

     ```bash
     ./run.sh
     ```

   - **For Windows**:

     ```powershell
     .\run.cmd
     ```

   This will start the runner and connect it to your GitHub repository.

#### **Running the Runner as a Service (Optional) 🔄**

If you want the runner to automatically start every time your machine reboots, you can run it as a service:

- **For Linux**:

  ```bash
  sudo ./svc.sh install
  sudo ./svc.sh start
  ```

- **For Windows**:

  ```powershell
  .\svc.cmd install
  .\svc.cmd start
  ```

---

### **Step 3: Modify Your GitHub Actions Workflow** ⚙️

#### **Understanding Workflow YAML 📝**

A **GitHub Actions Workflow** is defined in a `.yml` file inside the `.github/workflows` directory. This file tells GitHub Actions what to do when a certain event occurs (e.g., pushing code to the repository).

Here’s how to modify your GitHub Actions workflow to use your self-hosted runner.

1. **Create the Workflow YAML File**:
   - In your repository, create a `.github/workflows/ci.yml` file (if it doesn’t exist already).
   
2. **Modify the `runs-on` Field**:
   - Change the **`runs-on`** field from `ubuntu-latest` (or any hosted runner) to **self-hosted**.

Here’s an example `ci.yml` that uses a self-hosted runner and runs tests for a C# WPF project:

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

### **Step 4: Create and Set Up Unit Tests for Your Project** 🧪

Unit tests are essential to verify that your code works as expected.

#### **Setting Up Unit Tests in Visual Studio 🖥️**

1. Open your **C# WPF project** in **Visual Studio**.
2. Right-click on your solution in the Solution Explorer, then choose **Add > New Project**.
3. Choose **xUnit Test Project (.NET Core)** or **NUnit** if you prefer.
4. Add your test methods to the newly created test project, like so:

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

### **Step 5: Test Your GitHub Actions Workflow** ✅

1. **Push Changes to GitHub**:
   - Commit your changes and push them to your GitHub repository.
   
2. **Trigger the Workflow**:
   - GitHub Actions will automatically run the workflow on the `push` event (and any pull requests).

3. **Monitor the Workflow**:
   - Go to the **Actions** tab in your GitHub repository to monitor the workflow.
   - You should see logs for each step (checkout, build, test) and whether they passed or failed.

4. **Check Runner Status**:
   - In the **Actions > Runners** section of your GitHub repository, verify that your self-hosted runner is **online** and **active**.


### **Step 6: Monitoring the Workflow Execution and Debugging Issues** 🕵️‍♂️

Once your **GitHub Actions workflow** is triggered, it’s important to monitor the execution to ensure everything is running smoothly.

#### **Monitoring the Workflow** 🔄
You can monitor the execution of your workflow directly within GitHub:

1. Go to the **Actions** tab of your GitHub repository.
2. Under **Workflows**, you’ll see a list of runs. The latest run will be at the top.
3. Click on the specific run to see detailed logs for each step in your workflow.

Each step of the workflow, like checking out the repository, setting up the .NET SDK, restoring dependencies, building, and running tests, will have its own log. Here’s what you can look for:

- **Green checkmarks** indicate that a step passed successfully.
- **Red X's** indicate failure. When a step fails, it will show an error message in the log. This is where you can start debugging.

#### **Debugging Issues** 🔧

If your workflow fails during execution, there are several common issues to look for:

1. **Missing Dependencies**: Make sure your runner has all the required dependencies installed (e.g., .NET SDK version, tools required for the WPF app).
2. **Incorrect Environment Variables**: Sometimes issues arise if the right environment variables aren’t set, such as database URLs, credentials, or API keys.
3. **Permission Issues**: If your workflow is failing when trying to access a resource (like a file or a database), ensure the runner has the appropriate permissions.

If a specific step fails, you can fix the issue and then commit your changes, which will automatically trigger a new workflow run.

---

### **Step 7: Implementing Deployment to a Testing Environment** 🌍

Once you’ve set up your **CI pipeline** to run tests successfully, the next logical step is to **automatically deploy** your project to a testing or staging environment. 

Here’s how to do it:

#### **Define Deployment in the GitHub Actions Workflow** ⚙️

To deploy your C# WPF project, you might want to copy the built files to a remote server, publish to a web server, or use an **Azure** or **AWS** deployment pipeline. Here’s an example of deploying your build output to a **Windows server** after a successful test:

```yaml
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

  deploy:
    runs-on: self-hosted

    needs: build-and-test  # Ensures deployment happens after build and tests
    steps:
      - name: Deploy to Remote Server
        run: |
          scp -r ./bin/Release/* user@remote-server:/path/to/deploy
```

- **`scp` (Secure Copy Protocol)**: This command securely copies files to a remote server. Replace it with your preferred deployment method.
- **`deploy`**: This step will only run after the **build-and-test** job succeeds, ensuring that deployment only happens after tests are confirmed to pass.


### **Step 8: Running UI Tests on WPF Project** 🧪

Since you're working with a **WPF application**, you might want to automate the testing of its **UI components** (e.g., button clicks, window rendering, etc.). You can achieve this using **UI testing frameworks** like **Appium**, **WinAppDriver**, or **SikuliX**.

Here’s an example of how to set up **WinAppDriver** for UI tests on your self-hosted runner:

1. **Install WinAppDriver** on your self-hosted runner.
   - Download **WinAppDriver** from the official [Microsoft site](https://github.com/microsoft/WinAppDriver/releases).
   - Install it on your runner machine.

2. **Create UI Tests in Visual Studio**:
   - Create a new **UI Test Project** in Visual Studio for your WPF project.
   - Add **WinAppDriver** as a NuGet package in your test project.

3. **Example UI Test Code**:

```csharp
using OpenQA.Selenium.Appium.Windows;
using OpenQA.Selenium.Appium;

public class WpfAppTests
{
    private WindowsDriver<WindowsElement> session;

    public void Setup()
    {
        var options = new AppiumOptions();
        options.AddAdditionalCapability("app", @"C:\Path\To\YourApp.exe");

        session = new WindowsDriver<WindowsElement>(new Uri("http://127.0.0.1:4723"), options);
    }

    [Fact]
    public void TestButtonClick()
    {
        var button = session.FindElementByAccessibilityId("ButtonId");
        button.Click();
        var resultText = session.FindElementByAccessibilityId("ResultText").Text;

        Assert.Equal("Expected Text", resultText);
    }
}
```

4. **Trigger the UI Tests in GitHub Actions**:
   - After you set up WinAppDriver and create your UI tests, modify your GitHub Actions workflow to include the UI tests:

```yaml
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

      - name: Run Unit Tests
        run: dotnet test --configuration Release

      - name: Run UI Tests
        run: dotnet test --configuration Release --filter "TestCategory=UI"
```

This allows you to automate **UI testing** as part of your CI pipeline.

---

### **Step 9: Handling Multiple Environments** 🌎

In a real-world scenario, you’ll likely have multiple environments: **development**, **testing**, **staging**, and **production**.

You can extend your GitHub Actions workflow to deploy to different environments based on branch names, tags, or specific triggers.

Here’s how you can differentiate between **staging** and **production**:

```yaml
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

  deploy-to-staging:
    runs-on: self-hosted
    if: github.ref == 'refs/heads/staging'  # Deploy only for the staging branch
    needs: build-and-test
    steps:
      - name: Deploy to Staging
        run: |
          scp -r ./bin/Release/* user@staging-server:/path/to/deploy

  deploy-to-production:
    runs-on: self-hosted
    if: github.ref == 'refs/heads/main'  # Deploy only for the main branch
    needs: build-and-test
    steps:
      - name: Deploy to Production
        run: |
          scp -r ./bin/Release/* user@production-server:/path/to/deploy
```

- **`if: github.ref == 'refs/heads/staging'`** ensures that deployment only happens when pushing to the **staging** branch.
- Similarly, **`refs/heads/main`** ensures deployment to **production** when pushing to the `main` branch.

---


# 2025-04-01

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
      #- name: Restore dependencies (MSBuild)
      #  run: |
      #    & "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\MSBuild\Current\Bin\MSBuild.exe" /t:restore /p:Configuration=Release Fw.sln
      #  shell: pwsh

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
