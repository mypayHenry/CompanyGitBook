---
description: 一些detail
---

# 使用Xampp建立本地環境

### .NET問題

在專案的背景程式中會需要執行一些與windows系統溝通的操作，此時會發生下列錯誤

```
local.ERROR: Class 'COM' not found {"exception":"[object] (Symfony\\Component\\Debug\\Exception\\FatalThrowableError(code: 0): Class 'COM' not found at D:\\k20-mypay\\app\\Library\\MyPay\\JobHelper.php:29)"}
```

這是因為從 PHP 5.4.5 開始 .net套件就不會直接建入於PHP的核心中，必須在php.ini中最底下新增

```
[COM_DOT_NET]
extension=php_com_dotnet.dll
```

### 輸入上限限制

有些頁面的輸入過大，預設值10000會不夠用，所以要將其限制往上延伸

在**php.ini，php.ini-development，php.ini-production**，尋找max\_input\_vars，並改為

```
max_input_vars = 500000
```

