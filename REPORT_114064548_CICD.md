# HW4 - CI/CD Report

課程名稱：雲端原生軟體開發與最佳實踐 Cloud Native Development and Best Practice  
作業名稱：HW4 - CI/CD  
學號：114064548  
姓名：莊杰霖  
GitHub Repository：https://github.com/a87769989000/cicd-lab

## 1. CI Pipeline 說明

本作業使用 GitHub Actions 建立 CI pipeline。Pipeline 設定於 `.github/workflows/ci_114064548.yaml`，觸發條件為 `push`，也就是每次將程式碼 push 到 GitHub repository 時，GitHub Actions 會自動執行檢查。

Pipeline 使用 `ubuntu-latest` runner，並透過 `actions/setup-node@v4` 安裝 Node.js 22。依據專案的 `package-lock.json`，使用 `npm ci` 安裝固定版本的 dependencies，確保 CI 環境與專案鎖定版本一致。

CI 中包含三個主要檢查：

1. TypeScript typecheck：執行 `npm run typecheck`，實際指令為 `tsc --noEmit`，用來確認 TypeScript 型別是否正確。
2. Prettier check：執行 `npm run format:check`，實際指令為 `prettier --check .`，用來確認程式碼格式是否符合專案設定。
3. Test：執行 Vitest 測試，並產生 JUnit XML 測試報告，讓測試結果可以顯示在 GitHub Actions 結果頁面。

若 typecheck、Prettier check 或 test 任一項失敗，GitHub Actions job 會顯示 failed，符合本作業「任一檢查失敗時 pipeline 應顯示失敗」的要求。

## 2. `.github/workflows/ci_114064548.yaml` 主要內容

```yaml
name: CI 114064548

on:
  push:
    branches:
      - '**'

permissions:
  contents: read
  checks: write
  actions: read

jobs:
  verify:
    name: Typecheck, format, and test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: TypeScript typecheck
        run: npm run typecheck

      - name: Prettier check
        run: npm run format:check

      - name: Run tests and create JUnit report
        run: |
          mkdir -p reports
          npm test -- --reporter=default --reporter=junit --outputFile=reports/vitest-junit.xml

      - name: Publish test results to GitHub Actions
        uses: dorny/test-reporter@v2
        if: always() && hashFiles('reports/vitest-junit.xml') != ''
        with:
          name: Vitest Test Results
          path: reports/vitest-junit.xml
          reporter: java-junit
          fail-on-error: true
```

## 3. 使用工具與策略

本作業使用的工具如下：

- GitHub Actions：作為 CI pipeline 平台。
- actions/checkout@v4：在 runner 中 checkout repository 原始碼。
- actions/setup-node@v4：安裝 Node.js 22，並啟用 npm cache。
- npm ci：根據 `package-lock.json` 安裝 dependencies，確保 CI 環境穩定且可重現。
- TypeScript compiler：透過 `tsc --noEmit` 檢查型別，不輸出編譯檔案。
- Prettier：檢查程式碼格式是否符合 `.prettierrc` 設定。
- Vitest：執行專案測試。
- dorny/test-reporter@v2：讀取 Vitest 產生的 JUnit XML，將測試結果顯示於 GitHub Actions 結果頁面。

設計策略是將 typecheck、format check、test 拆成不同步驟。這樣當 pipeline 失敗時，可以直接從 GitHub Actions 的 step 名稱判斷是哪一項檢查失敗，方便除錯與修正。

## 4. CI 執行結果截圖

請在 GitHub repository 進入 Actions 頁面，選擇最新的 `CI 114064548` workflow run，截圖成功執行結果並貼在此處。

成功截圖應包含：

- Workflow 名稱：`CI 114064548`
- Job 狀態為 success
- TypeScript typecheck、Prettier check、Run tests 等 step 都成功
- GitHub Actions 頁面中顯示 Vitest test result

請將成功截圖貼在這裡：

[請在 PDF 中貼上成功執行截圖]`n`n成功執行連結：https://github.com/a87769989000/cicd-lab/actions/runs/26293669357

## 5. 失敗案例說明

本作業的失敗案例使用 Prettier 格式錯誤示範。我另外建立 `failure-demo-114064548` 分支，在 `src/app.ts` 中故意加入不符合 `.prettierrc` 的格式，接著 push 到 GitHub，讓該分支產生 failed pipeline。

範例：

```ts
const message = 'format error example';
```

由於專案 `.prettierrc` 設定 `singleQuote: true`，Prettier 期待字串使用單引號，因此 `npm run format:check` 會失敗。GitHub Actions 的 `Prettier check` step 會顯示 failed，整個 pipeline 也會顯示 failed。

修正方式是執行：

```bash
npm run format
```

或手動把格式改回符合 Prettier 規則，例如：

```ts
const message = 'format error example';
```

修正後再次 commit 並 push，GitHub Actions pipeline 應回到 success。

請將失敗截圖貼在這裡：

[請在 PDF 中貼上 pipeline failed 截圖]`n`n失敗執行連結：https://github.com/a87769989000/cicd-lab/actions/runs/26293739822

## 6. 結論

本 CI pipeline 在每次 push 時自動執行 TypeScript typecheck、Prettier check 與 Vitest test。透過 GitHub Actions 的 step 狀態，可以清楚看到每項檢查結果；透過 JUnit XML 與 test reporter，也能在 Actions 結果頁面查看測試結果。此設計能在程式碼進入主分支前提早發現型別錯誤、格式錯誤與測試失敗，提升專案品質。
