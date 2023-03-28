# 測試區更新

### 如何判別該動哪台主機?

分web (如 corp.usecase.cc) 跟 ap (api.usecase.cc) 層，如果要在測試機線上更正不知道分辨在哪一層，有個簡單方式。

程式碼中會有 post 的這個動作，比如：EloquentPayoffRepository.php這種類型會大量訪問資料庫的檔案，每隻function都會有一個post的動作去取得資料庫資料。

```php
$apiHelper = APIHelper::post(config("websites.api.protocol") . "://" . config("websites.api.host") . "/api/payoff/batchInquirer", $PayoffBatchInquirerRequestVO, config("websites.api.keyCenterMapSystemCode"));
```

那在這個動作前都屬於web層，後則屬於AP層。

### 修正程式碼之後?

修正完後要重啟swoole，打開 /etc/rc.local，看到如下這段：

```hgignore
su developer -c "/bin/php /home/service/k20-mypay/webroot/artisan swoole:http restart &"
```

執行之後測試站就會更新
