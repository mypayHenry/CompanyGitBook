# 資料庫操作習慣

除了select之外，凡當其他insert、update、delete需要連續操作多張表時，要記得做以下動作

```php
<?php
    $db = \DB::connection('該資料庫';
    $db->beginTransaction();
    try {
        # do something
        $db->commit();
    } catch (Exception $exception) {
        # do some Error proccessing
        $db->rollback();
    }
?>
```

