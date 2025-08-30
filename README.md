## 🔗 [Setup Connection to Azure PostgreSQL in VS Code](https://github.com/heartlanguage2024/heartlanguage2024/wiki/20250604-(水)-Setup-Connection-of-Azure-PostgreSQL-in-VS-Code)

---

## 🥔 Load data from Temporary Database

```csharp
private async Task<List<potatoModel>> LoadDatabaseDataToJsonAsync()
{
    Expression<Func<potatoModel, bool>> condition =
        e => e.GroupName == group; 

    return await viewModel.LoadDataFromDatabaseToTempDb<potatoModel>(condition);
}
```


---

## 🧠　ConfigureAwait(true)

### ✅ When to use `ConfigureAwait(true)` (or omit it)

Use it when **you need to interact with the UI or other context-sensitive resources after the `await`**.

#### 🔧 Example – WPF/WinForms UI update

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    // Runs on the UI thread
    var data = await GetDataAsync(); // ConfigureAwait(true) is implied

    // UI update — MUST run on the UI thread
    myLabel.Text = data;
}
```

If you had used `ConfigureAwait(false)` here, the `myLabel.Text = data;` line might throw an exception or silently fail because it’s not running on the UI thread.

---

### ❌ Don’t use `ConfigureAwait(false)` if...

* You need to access:

  * UI controls (`Label`, `TextBox`, `Canvas`, etc.)
  * SynchronizationContext-bound services (like ASP.NET legacy `HttpContext`)
* You're writing code inside a **UI layer** (WPF, WinForms, Xamarin, MAUI)

---

### 🧠 Summary Table

| Context                 | Use `ConfigureAwait(false)` | Use `ConfigureAwait(true)` (or omit) |
| ----------------------- | --------------------------- | ------------------------------------ |
| UI code (WPF, WinForms) | ❌ No                        | ✅ Yes                                |
| Updating UI elements    | ❌ No                        | ✅ Yes                                |
| Background service code | ✅ Yes                       | ❌ No                                 |
| Library method (no UI)  | ✅ Yes                       | ❌ No                                 |
| ASP.NET Core            | ✅ Yes                       | ❌ No (no SynchronizationContext)     |
| ASP.NET (classic)       | ⚠️ Maybe                    | ⚠️ Maybe (depends on `HttpContext`)  |

---

### 🧪 Advanced Tip

If you **accidentally use `ConfigureAwait(false)` in UI code**, you may get exceptions like:

> `InvalidOperationException: The calling thread cannot access this object because a different thread owns it.`

This happens because UI frameworks enforce **single-threaded UI access**, and you're trying to touch UI elements from a background thread.


---

## 🥔 CREATE (To Add/Insert data)  
```csharp
var newItem = new potatoModel
{
    Title = "AX11-14",
    Spec = TableControl.TitleLabel.Content.ToString(),
    Dim1 = 1120,
    Space = "～",
    Dim2 = 1460,
    Deleteable = 1
};
await viewModel.AddtoTempDatabase(new List<potatoModel> { newItem });
```


## 🥔 READ (To load data and bind to UI instance) 
```csharp
var items = await viewModelCommon.LoadTableDataAsync(item => item.Id == 1);
Thickness.DataContext = new ObservableCollection<potatoModel>(items);
Qty.DataContext = new ObservableCollection<potatoModel>(items);
```

## 🥔 READ (To load data and display on DataGrid) 
```csharp
Expression<Func<potatoModel, bool>> filter = item => item.Deleteable != -1;
await TableControl.LoadDataToTable(filter);
```

## 🥔 UPDATE (To Update database with Dictionary)
```csharp
var updatedValues = new Dictionary<string, object>
{
    { textBox.Name, textBox.Text }
};
var conditions = new Dictionary<string, object>
{
    { "Id", 1 }
};
await DbHelper.UpdateDatabase<potatoModel>(updatedValues, conditions, "potato");
```

## 🥔 UPDATE (To Update database with Dictionary at the DataGrid)
```csharp
private void TextBox_TextChanged(object sender, TextChangedEventArgs e)
{
    try
    {
        if (sender is TextBox textBox)
        {
            var items = TableControl.DataGrid.ItemsSource as List<potatoModel>;
            TextBoxHelper.HandleTextChanged(textBox, items, viewModel.UpdateTempDatabase);
        }
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Error updating JSON file: {ex.Message}", "Error", MessageBoxButton.OK, MessageBoxImage.Error);
    }
}
```

## 🥔 DELETE (To Delete data)  
```csharp
if (TableControl.deleteParameter is potatoModel selectedItem)
{
    await viewModel.RemoveDataFromTempDatabaseAsync(selectedItem);
    await LoadDataToTable();
}
```


---

# 🚀 機能開発ワークフローガイド（GitHub + Projects）

このガイドでは、GitHub の **Issues、ブランチ、Projects** を活用して機能開発を管理するための **ステップバイステップの手順** を解説します。


## ✅ 1. フィーチャーブランチの作成とリモートへのプッシュ

ローカルのGitターミナルで以下を実行します：

```bash
# 最新のmainブランチに切り替え
git checkout main
git pull origin main

# 新しいフィーチャーブランチを作成
git checkout -b feature/your-feature-name

# 作成したブランチをリモートにプッシュ
git push -u origin feature/your-feature-name
```

---

## ✅ 2. GitHub Issueの作成とProjectへの紐付け

1. GitHub上の対象リポジトリにアクセス
2. 上部タブの **「Issues」** をクリック
3. **「New Issue」** をクリック
4. 以下を入力：
   - **タイトル**：短く分かりやすい機能名（例：`ログインバリデーションの実装`）
   - **説明**：詳細な内容・目的など
5. 右サイドバーで以下を設定：
   - **Assignees（担当者）**：自分を選択
   - **Projects**：対象の GitHub プロジェクト（例：`MyApp開発`）を選択
   - **Labels（ラベル）**：`feature` や `enhancement` などを設定

---

## ✅ 3. Issue をブランチおよびプロジェクトにリンクする

### 🔗 3.1. Issue からプロジェクトステータスを設定

1. 作成したIssueを開き、右サイドバーを確認
2. **Projects** 項目の横にある矢印をクリックしてカードを展開
3. **Status** を `In Progress（作業中）` などに変更

これにより、GitHub Project 上でIssueの進捗が自動で管理されます。

---

### 🔗 3.2. Issue とフィーチャーブランチをリンクする

ブランチを作成してGitHubにプッシュした後、以下の手順でIssueに紐付けます：

1. 作成済みの **GitHub Issue** を開く  
2. 右サイドバーを下へスクロールし、**Development** セクションを探す  
3. **「Link a branch（ブランチをリンク）」** をクリック  
4. 表示されたダイアログで、該当するブランチ名（例：`feature/your-feature-name`）を選択または入力  
5. **「Link branch」** をクリック

リンクが完了すると：

- **Development** セクションにブランチ名が表示されます
- Issue はそのブランチと開発作業に紐付きます
- 後で Pull Request を作成すると、**Pull requests** タブに表示されます
- GitHub Projects に自動化が設定されている場合、ステータスが自動更新されることもあります

---

## 🎯 Why Both Appear?

When you:
- Create an **issue** and add it to the project → it appears as one item.
- Create a **pull request** and also add it to the same project (or use "Closes #issue") → the PR is a **separate card**.

They appear separately **even if the PR closes the issue**.

---

## ✅ How to Combine Them or Manage Cleaner?

There are a few strategies to clean this up:

---

### **Option 1: Use Only Issues in the Project**
- Only track **issues** in your project.
- Don’t add PRs directly to the board.
- Let PRs just be linked to their corresponding issue (e.g., using `Closes #123`).
- When the PR is merged → issue is closed → project shows issue as done.

🔁 **Auto-updates** happen based on the issue status.

---

### **Option 2: Use PRs Only for Feature Work**
- If your workflow prefers PR tracking in projects:
  - Don’t add issues.
  - Add only PRs and manage their progress (e.g., add statuses like "In Review", "Ready to Merge").
- Issues are just "discussion" or "planning", and you close them via PRs, but don’t track them on the board.

---

### **Option 3: Use Automation to Sync**
You can automate this using **GitHub Actions** or manually with project fields:

- If a PR is merged and it closes an issue → auto-move both PR and issue to “Done” (using rules or GitHub Actions).
- Use the `linked pull requests` field to connect them visually.

---

### 👇 Example of PR Closing Issue and Reducing Duplication

#### 1. Create an issue:
```markdown
Implement Beam ComboBox selection logic
```

#### 2. In the PR:
```markdown
### Changes
- ComboBox now shows `Title`, assigns `Dim3`
- Delay added after update using `BeamItem_Reset_Delay`

Closes #45
```

#### 3. In the project:
- Only add issue #45
- When PR is merged → issue #45 is closed → project updates → **no duplicate PR card needed**

---

## ✨ Tips to Keep It Clean

| Action | Result |
|--------|--------|
| Use `Closes #issue` in PR | Auto closes issue when PR is merged |
| Avoid manually adding both issue and PR to the project | Prevents duplicate tracking |
| Use automation rules in GitHub Project (Beta) | Keeps board clean and synced |
| Consider using only one of Issue or PR for tracking | Depends on your team workflow |

---

## ✅ **To auto-close an issue when a pull request is merged:**

You need to include a special **closing keyword** followed by the issue number in either:

- The **pull request description**, or  
- A **commit message** (that’s part of the PR and gets merged into the default branch).

---

### ✅ Valid keywords include:
- `Closes #123`
- `Fixes #123`
- `Resolves #123`

Where `#123` is your issue number.

> ❗️Just referencing the issue number (like `#123`) without one of the keywords will **not** close the issue.

---


### 🔧 Try These Steps to Force Kill It

#### ✅ 1. Use Command Line (force kill)
Open **Command Prompt as Administrator** and run:

```bash
taskkill /F /IM Fw.exe
```

- `/F` = force termination
- `/IM` = image name

If it says "Access Denied" or doesn't kill it, continue below.

---

#### ✅ 2. Kill by PID (process ID)
Since your error shows the PID (`13948`), you can kill it directly:

```bash
taskkill /F /PID 13948
```

Replace `13948` with whatever PID you see in Task Manager.

---

#### ✅ 3. Use PowerShell (admin)
```powershell
Stop-Process -Name Fw -Force
```

Or:

```powershell
Stop-Process -Id 13948 -Force
```

---

#### ✅ 4. Restart `explorer.exe` (if it's a system file lock)
Sometimes shell processes hold it — restart Explorer:

```bash
taskkill /F /IM explorer.exe
start explorer.exe
```

---

#### ✅ 5. Reboot (last resort)
If nothing else works, just reboot the PC to unlock the file.

---

