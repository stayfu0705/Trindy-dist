# Trindy — Informatica → Trinity 轉換工具

將 Informatica PowerCenter/PowerMart 的 Mapping XML 自動轉換為 Trinity 的匯入檔（`.gz`），並同時產出欄位分析報告（Excel）。

---

## 1. 環境需求

- **作業系統**：Windows
- **Java**：8 ～ 21 皆可
  - 可使用內建 JDK（放在 `java\` 資料夾），或使用系統已安裝的 Java（在 PATH 中）。

---

## 2. 目錄結構（**第一次使用前請先建好**）

請在你要放置工具的位置（建議 `C:\Trindy`）建立以下結構：

```
C:\Trindy\
├─ Trindy.bat            ← 啟動腳本
├─ bin\
│  └─ Trindy.jar         ← 從 GitHub Release 下載後放這裡
├─ cfg\
│  └─ Trindy.conf        ← 設定檔（內容見第 4 節）
├─ java\                 ← （選用）內建 JDK；若用系統 Java 可省略
│  └─ bin\java.exe
├─ input\                ← 放「要轉換的 Informatica XML」
├─ log\                  ← 執行記錄（自動產生）
└─ output\
   ├─ excel\             ← 欄位分析報告（.xlsx）
   ├─ gz\                ← Trinity 匯入用檔（.gz）
   └─ topology\          ← 流程拓樸圖（.txt）
```

> 重點：**`bin`、`cfg`、`input`、`log`、`output\excel`、`output\gz`** 這幾個資料夾請先手動建立。

---

## 3. 安裝步驟

1. 到 Release 頁下載最新的 `Trindy.jar`：
   **https://github.com/stayfu0705/Trindy-dist/releases/latest**
2. 依第 2 節建好 `C:\Trindy` 的資料夾結構。
3. 把下載的 `Trindy.jar` 放到 `C:\Trindy\bin\`。
4. 在 `C:\Trindy\cfg\` 建立 `Trindy.conf`（內容見第 4 節）。
5. （選用）若要用內建 Java，把 JDK 放到 `C:\Trindy\java\`；否則確認系統已安裝 Java 8～21。
6. 把 `Trindy.bat`（見第 6 節）放在 `C:\Trindy\` 根目錄。

---

## 4. 設定檔 `cfg\Trindy.conf`

```ini
# 此路徑最重要，所有子目錄皆由此延伸
set.TRINDY_HOME=C:/Trindy

TRINDY_BIN_PATH=%TRINDY_HOME%/bin
TRINDY_CFG_PATH=%TRINDY_HOME%/cfg
TRINDY_JAVA_PATH=%TRINDY_HOME%/java
TRINDY_INPUT_PATH=%TRINDY_HOME%/input
TRINDY_LOG_PATH=%TRINDY_HOME%/log
TRINDY_OUTPUT_PATH=%TRINDY_HOME%/output
TRINDY_EXCEL_PATH=%TRINDY_HOME%/output/excel
TRINDY_GZ_PATH=%TRINDY_HOME%/output/gz

# Java 執行：預設用 C:/Trindy/java/bin/java；設為空白則用系統 PATH 的 java
java.home=%TRINDY_JAVA_PATH%

# Trinity 匯出預設值（可被命令列參數覆蓋）
default.entity=DEFAULT
default.category=TEST
```

> 若不放內建 JDK、改用系統 Java，把 `java.home=` 後面留空即可。

---

## 5. 使用方式

**最簡單：** 把要轉換的 Informatica XML 放進 `input\`，然後雙擊（或在命令列執行）`Trindy.bat`。
工具會自動掃描 `input\` 內所有 `.xml` 並轉換。

執行完到這裡拿結果：
- `output\gz\` → Trinity 匯入用的 `.gz`
- `output\excel\` → 欄位分析報告 `.xlsx`
- `log\` → 執行記錄（出錯時先看這裡）

### 命令列參數（選填）

| 參數 | 說明 |
|---|---|
| `-i <路徑>` | 指定單一 XML 或資料夾；省略時自動掃描 `input\` |
| `-entity <名稱>` | 指定 Trinity Business Entity（預設 `DEFAULT`） |
| `-category <名稱>` | 指定 Trinity Category（預設 `TEST`） |
| `-o <資料夾>` | 自訂輸出資料夾 |
| `-h` / `-?` | 顯示說明 |

範例：

```bat
Trindy.bat -i input\m_SAMPLE.xml -entity 00_ETL_MIG -category LOADING
```

---

## 6. 啟動腳本 `Trindy.bat`

放在 `C:\Trindy\` 根目錄。它會自動讀 `cfg\Trindy.conf` 找 Java，再執行 `bin\Trindy.jar`：

```bat
@echo off
setlocal EnableDelayedExpansion
set "ROOT=%~dp0"
set "JAR=%ROOT%bin\Trindy.jar"
set "CONF=%ROOT%cfg\Trindy.conf"

if not exist "%JAR%" (
    echo [ERROR] 找不到 Trindy.jar，請放到 bin\ 後再執行。
    pause & exit /b 1
)

set "JAVA_HOME_CFG=java"
if exist "%CONF%" (
    for /f "usebackq eol=# tokens=1,* delims==" %%a in ("%CONF%") do (
        set "KEY=%%a" & set "VAL=%%b" & set "KEY=!KEY: =!"
        if /i "!KEY!"=="java.home" ( set "VAL=!VAL: =!" & if not "!VAL!"=="" set "JAVA_HOME_CFG=!VAL!" )
    )
)

if exist "%ROOT%%JAVA_HOME_CFG%\bin\java.exe" (
    set "JAVA_EXE=%ROOT%%JAVA_HOME_CFG%\bin\java.exe"
) else if exist "%JAVA_HOME_CFG%\bin\java.exe" (
    set "JAVA_EXE=%JAVA_HOME_CFG%\bin\java.exe"
) else (
    set "JAVA_EXE=java"
)

"%JAVA_EXE%" -jar "%JAR%" %*
if not %ERRORLEVEL%==0 ( echo [ERROR] 執行失敗，請看 log\ 目錄。& pause & exit /b 1 )
exit /b 0
```

---

## 7. 常見問題

- **執行沒反應 / 報錯** → 先看 `log\` 裡最新的記錄檔。
- **找不到 Java** → 安裝 Java 8～21,或把 JDK 放進 `java\`、或把 `java.home=` 留空改用系統 Java。
- **轉出的 `.gz` 匯入 Trinity 後要人工確認的地方** → 工具會在 Excel 報告的「特殊元件備忘」頁與 log 中標記 `[MANUAL-WARN]` / `[MANUAL-ERROR]`,請依說明在 Trinity UI 補設定。
