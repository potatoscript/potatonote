# 🚀 機能開発ワークフローガイド（GitHub + Projects）

このガイドでは、GitHub の **Issues、ブランチ、Projects** を活用して機能開発を管理するための **ステップバイステップの手順** を解説します。

---

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


