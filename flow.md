# GithubFlow 運用手順（develop蓄積型）

## 目的
本番デプロイのトリガーとなる `main` ブランチへのマージを1回にまとめつつ、複数の機能を同時にリリース可能にするためのワークフローです。現在のGitflowの `develop` ブランチの概念を活用します。運用はGitflowは廃止して、GithubFlowに移行します。



## ブランチ戦略
**`main`**: 本番環境のコードと完全に一致するブランチ。このブランチに変更がマージされると、**本番環境への自動デプロイ**が実行されます。  
**`develop`**: 次回リリース予定の機能を集約する開発用メインブランチ。  

---

## 通常の開発・リリースフロー

### 1. 機能開発
新機能の開発や通常のバグ修正は、常に `develop` からブランチを作成します。
`develop` ブランチを最新化する。
`develop` から `feature/xxx` ブランチを作成する。
ローカルで開発を行い、コミット＆プッシュする。

```
1. git checkout develop
2. git pull
3. git checkout -b feature/xxx
4. git add .
5. git commit -m "修正"
6. git push origin feature/xxx
```

### 2. developへの統合（日々）
開発が完了したら、`develop` に向けてプルリクエストを作成します。
`feature/xxx` から **`develop`** に向けてプルリクエスト（PR）を作成する。
チームメンバーによるコードレビューを実施する。
レビュー承認後、リリースが確定したら、PRを `develop` にマージする。
（必要に応じて）`develop` を参照している環境（ステージング等）で結合テストを行う。

### 3. 本番リリース（不定期）
リリース日が決定し、`develop` に蓄積された機能群を本番環境へ一括反映させる手順です。  
**`develop`** から **`main`** に向けて「リリース用PR」を作成する。（タイトル例: `Release v1.2.0` など）  
タスク担当者が自身のブランチが `develop` にマージされているかを最終確認する。  
リリース用PRを `main` にマージする。マージ後の `develop` ブランチは、今後も使用するので、削除しないことが重要です。  
デプロイ完了後、Githubで`main` ブランチにリリースタグ（`v1.2.0`等）を打つ（`Create a new release`）。

---

## Branch protection rules

## main

- [ ] Restrict creations
- [ ] Restrict deletions
- [ ] Require a pull request before merging
   - [ ] Required approvals = 1
   - [ ] Dismiss stale pull request approvals when new commits are pushed
   - [ ] Require approval of the most recent reviewable push
   - [ ] Require conversation resolution before merging
- [ ] Require status checks to pass
   - [ ] Require branches to be up to date before merging
   - [ ] Add Checks > main Branch Protection     
- [ ] Block force pushes
     
### main Branch Protection

Actions にて、`main` には `develop` しかマージできないように制限する。

---

## 緊急時のバグ修正フロー（Hotfix）
本番環境（`main`）で致命的なバグが発生し、未リリースの `develop` の内容を含めずに修正だけを即時反映させたい場合の手順です。

**`main`** から直接 `hotfix/xxx` ブランチを作成する。バグを修正し、コミット＆プッシュする。  
`hotfix/xxx` から **`main`** に向けてPRを作成し、マージする。先祖返りを防ぐため、同じ修正を `develop` にも反映させる。    
手順: `main`（または `hotfix/xxx`）から **`develop`** に向けてもPRを作成し、マージして同期しておく。  
`Branch protection rules` で、 `Require branches to be up to date before merging` を `ON` にするので、同じ修正を `develop` に反映し忘れても、`develop` から `main` に向けてマージできない状態になるので、ミスは起きない想定です。
