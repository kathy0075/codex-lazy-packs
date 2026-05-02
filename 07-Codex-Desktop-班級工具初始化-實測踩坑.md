# 07 班級工具初始化 - Codex Desktop 實測踩坑紀錄

> 實測日期：2026-05-02  
> 環境：Windows PowerShell + Codex Desktop + Google Drive + GitHub CLI  
> 實測目標：依照 #07 初始化 `2026database` 班級工具工作區

---

## 最終成功狀態

本次成功初始化的專案：

```text
G:\我的雲端硬碟\2026database
```

GitHub repo：

```text
https://github.com/kathy0075/2026database
```

repo 狀態：

```text
visibility: PRIVATE
default branch: main
```

Firebase：

```text
project id: teacherstudy-b16f3
database: projects/teacherstudy-b16f3/databases/(default)
```

Obsidian 筆記：

```text
C:\Users\kathy\OneDrive\文件\Obsidian Vault\2026database\專案工作總覽.md
```

Codex skills：

```text
C:\Users\kathy\.codex\skills\startup-sync\SKILL.md
C:\Users\kathy\.codex\skills\shutdown-sync\SKILL.md
C:\Users\kathy\.codex\skills\project-init-sync\SKILL.md
```

---

## 踩坑 1：目前工作資料夾不是 Git repo

一開始在 `G:\我的雲端硬碟\2026database` 執行 Git 狀態會出現：

```text
fatal: not a git repository (or any of the parent directories): .git
```

這不是錯誤，而是 #07 初始化流程的正常起點。

處理方式：

```powershell
git init
git config windows.appendAtomically false
```

`windows.appendAtomically false` 對 Google Drive / OneDrive 資料夾很重要，可以降低 Git 更新 ref 時失敗的機率。

---

## 踩坑 2：不要把參考用懶人包 repo 提交進新專案

本次為了寫回懶人包，曾在 `2026database` 裡 clone：

```text
codex-lazy-packs/
```

這只是參考 repo，不屬於 `2026database` 專案。

所以 `.gitignore` 要加：

```gitignore
codex-lazy-packs/
```

否則容易把整包懶人包誤提交到班級工具 repo。

---

## 踩坑 3：GitHub repo 不存在時，要先查再建立

先查 repo 是否已存在：

```powershell
gh repo view kathy0075/2026database --json name,url,visibility,defaultBranchRef
```

如果出現：

```text
Could not resolve to a Repository
```

代表 repo 還不存在，可以建立。

本次因為使用者沒有明確要求公開網站，所以採安全預設：

```powershell
gh repo create 2026database --private --source=. --remote=origin --push
```

如果之後要 GitHub Pages，再另行改 public 或設定 Pages。

---

## 踩坑 4：GitHub Pages 不應該預設開啟

#07 文件會提到 GitHub Pages，但班級工具或資料庫工作區可能含有設定資訊。

安全做法：

- 使用者沒有明確要求公開，就先建立 private repo。
- 不自動開 GitHub Pages。
- 需要發布頁面時，再由使用者確認 repo 是否可以公開。

本次採用：

```text
PRIVATE repo
GitHub Pages not enabled
```

---

## 踩坑 5：Codex skills 不存在，要補 `SKILL.md`

文件要求三個同步技能：

```text
startup-sync
shutdown-sync
project-init-sync
```

實測時三個都不存在：

```powershell
Test-Path "$env:USERPROFILE\.codex\skills\startup-sync\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\skills\shutdown-sync\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\skills\project-init-sync\SKILL.md"
```

結果是：

```text
False
False
False
```

處理方式是建立：

```text
C:\Users\kathy\.codex\skills\startup-sync\SKILL.md
C:\Users\kathy\.codex\skills\shutdown-sync\SKILL.md
C:\Users\kathy\.codex\skills\project-init-sync\SKILL.md
```

這三個 skills 的用途：

- `startup-sync`：開工時讀 `AGENTS.md`、查 Git 狀態、確認專案設定。
- `shutdown-sync`：收工時查 diff、更新文件、必要時 commit/push。
- `project-init-sync`：新專案初始化時建立 `AGENTS.md`、`README.md`、`.gitignore`、GitHub/Firebase/Obsidian 設定。

---

## 踩坑 6：Obsidian vault 不能用猜的

不要假設 vault 一定在：

```text
C:\Users\kathy\OneDrive\文件\Secondbrain
```

本次實際搜尋後找到：

```text
C:\Users\kathy\OneDrive\文件\Obsidian Vault
```

所以專案筆記建立在：

```text
C:\Users\kathy\OneDrive\文件\Obsidian Vault\2026database\專案工作總覽.md
```

並且回寫到 `AGENTS.md`：

```markdown
## Obsidian

Obsidian vault：C:\Users\kathy\OneDrive\文件\Obsidian Vault
專案筆記：2026database/專案工作總覽.md
```

---

## 踩坑 7：`AGENTS.md` 要記錄實際專案狀態

初始化時不要只放模板，要把本次實際資訊寫進去：

```text
專案名稱：2026database
本機資料夾：G:\我的雲端硬碟\2026database
GitHub repo：https://github.com/kathy0075/2026database
主要分支：main
Firebase project id：teacherstudy-b16f3
Firestore database：projects/teacherstudy-b16f3/databases/(default)
Obsidian vault：C:\Users\kathy\OneDrive\文件\Obsidian Vault
```

這樣下次 Codex 開工時不用重新猜。

---

## 踩坑 8：Firestore 狀態要最後再確認一次

#07 初始化不是只建文件，也要確認 Firebase 仍可用。

本次最後確認：

```powershell
npx.cmd -y firebase-tools@latest firestore:databases:list --project teacherstudy-b16f3
```

成功看到：

```text
projects/teacherstudy-b16f3/databases/(default)
```

這代表專案文件、GitHub、Obsidian 之外，Firebase 資料庫狀態也確認過。

---

## 本次建立的核心檔案

在 `G:\我的雲端硬碟\2026database`：

```text
AGENTS.md
README.md
.gitignore
.firebaserc
firebase.json
firestore.rules
```

在 Obsidian vault：

```text
2026database/專案工作總覽.md
```

在 Codex skills：

```text
startup-sync/SKILL.md
shutdown-sync/SKILL.md
project-init-sync/SKILL.md
```

---

## 快速初始化順序

1. 讀 #07 文件。
2. 檢查 Git / GitHub CLI / Node / Firebase / Codex skills。
3. 建立 `AGENTS.md`、`README.md`、`.gitignore`。
4. `git init`。
5. Google Drive / OneDrive 內加：

```powershell
git config windows.appendAtomically false
```

6. commit 初始化檔案。
7. `git branch -M main`。
8. 建立 GitHub private repo 並 push。
9. 找到真正 Obsidian vault，再建立專案筆記。
10. 補回 `AGENTS.md` 的 Obsidian 路徑。
11. 再 commit/push。
12. 最後確認 Git 狀態、GitHub repo、Codex skills、Obsidian 筆記、Firestore。

---

## 最後驗證清單

```powershell
git status --short
gh repo view kathy0075/2026database --json url,visibility,defaultBranchRef
Test-Path "$env:USERPROFILE\.codex\skills\startup-sync\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\skills\shutdown-sync\SKILL.md"
Test-Path "$env:USERPROFILE\.codex\skills\project-init-sync\SKILL.md"
Test-Path "C:\Users\kathy\OneDrive\文件\Obsidian Vault\2026database\專案工作總覽.md"
npx.cmd -y firebase-tools@latest firestore:databases:list --project teacherstudy-b16f3
```

本次最後結果：

```text
Git working tree: clean
GitHub repo: https://github.com/kathy0075/2026database
GitHub visibility: PRIVATE
default branch: main
Codex skills: all True
Obsidian note: True
Firestore database: found
```

---

## 重要結論

- #07 初始化不要只產生模板，要把實際 repo、Firebase、Obsidian、skills 狀態寫進 `AGENTS.md`。
- 沒有明確要求公開時，GitHub repo 先用 private。
- Google Drive / OneDrive 裡的 Git repo 要設定 `windows.appendAtomically false`。
- Obsidian vault 要實際搜尋確認，不要猜路徑。
- 參考用 repo 要加入 `.gitignore`，避免誤提交。
- 最後一定要做完整驗證，不只看 commit 是否成功。
