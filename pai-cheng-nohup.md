# 排程Nohup

程式碼中排程除了單獨的：

```php
\Queue::pushOn("nohup39buy", new MailCenter($MailVO));
```

還會有這種的：

```php
$jobId = Queue::pushOn("bg", new VoucherCenter('refundCashflow', $reocrds));
JobHelper::nohup($jobId, "api", "CON");
```

trace code後會看到如下：

```php
/**
 * 對賈伯斯射後不理
 * @param integer $jobId
 * @param string $website
 * @param string $execType WEB/CON
 */
public static function nohup(int $jobId, string $website = "api", string $execType = "WEB")
{
    $php = config('app.php_path');
    $artisan = base_path() . "/artisan";
    $cmd = "nohup";
    $execType = ($execType == 'CON') ? 'CON' : 'WEB'; // 強制轉換只有兩種模式
    self::makeBackground($php, [$artisan, $cmd, $jobId, $website, $execType]);
}
```

看起來還執行了同class底下的makeBackground function：

```php
/**
 * 背景執行
 * @param string $command
 * @param array $args
 */
public static function makeBackground(string $command, array $args = array())
{
    $cmd = "\"" . $command . "\"";

    if (is_array($args)) {
        foreach ($args as $arg) {
            $cmd .= " " . "\"" . $arg . "\"";
        }
    }

    if (strtoupper(substr(PHP_OS, 0, 3)) === 'WIN') {
        try {
            $wsc = new \COM("WScript.Shell");
            $wsc->run($cmd, 0, false);
            Log::info("執行背景程式成功，指令為：" . $cmd);
            Log::info("Job Id：" . ($args[2] ?? ""));
        } catch (\Exception $e) {
            Log::error("背景程式執行失敗，指令為：" . $cmd);
        }
    } else {
        exec("nohup $cmd > /dev/null 2>&1 &");
        Log::info("執行背景程式成功，完整的指令為：");
        Log::info("nohup $cmd >/dev/null 2>&1 &");
        Log::info("Job Id：" . ($args[2] ?? ""));
    }
}
```

可以看到這段function前面幾行組成$cmd的部分，以上述的範例套入的話會變成這樣

```php
$cmd = ' \"php\" \"artisan\" \"nohup\" \"$jobId\" \"api\" \"CON\" ';
```

乍看就是php artisan 指令，然後下面的程式碼就是看執行PHP的環境，如果環境是Windows就執行Windows Script Shell (WScript.Shell)，是linux環境的話直接使用PHP原生exec function 去執行artisan指令。

那再回來到 nohup這個指令，在 `app\Console\Commands\Nohup.php` 可以找到它，我們來看看handle

```php
public function handle()
{
    if (strtolower(config("queue.default")) == 'database') {
        // 發動背景處理以及目標處理肯定是同一台主機
        $arguments = $this->argument('jobName');
        $jobId = $arguments[0] ?? 0;
        $website = $arguments[1] ?? "api";
        $execType = $arguments[2] ?? "WEB";
        if ($execType == 'WEB') {
            $url = config("websites." . $website . ".protocol") . "://" . config("websites." . $website . ".host") . "/api/jobs/caller";
            $sysCode = config("websites." . $website . ".keyCenterMapSystemCode");
            $domainName = config("websites." . $website . ".host");

            $JobsCallerRequestVO = new JobsCallerRequestVO();
            $JobsCallerRequestVO->SysCode = $sysCode;
            $JobsCallerRequestVO->Cmd = "JobsCaller";
            $JobsCallerRequestVO->JobId = $jobId;
            $JobsCallerRequestVO->Token = $jobId;

            try {
                $api = APIHelper::post($url, $JobsCallerRequestVO, $sysCode, $sysCode, $domainName);
                $json = json_decode($api);
                if (isset($json->ResultCode) && $json->ResultCode == APIResultCode::Success) {
                    Log::info("背景處理 Job 呼叫 Jobs Web 處理完成。");
                    Log::info("Job Id：" . $jobId . "」");
                } else {
                    if (isset($json->ResultCode)) {
                        Log::info("呼叫 Jobs Web 處理失敗，失敗原因為：" . $json->ResultMsg);
                    } else {
                        Log::info("呼叫 Jobs Web 處理失敗，失敗原因為：回傳格式無法辨識。");
                    }
                }
            } catch (\Exception $e) {
                Log::info("呼叫 Jobs Web 處理失敗，失敗原因為：" . $e->getMessage());
            }
        } elseif ($execType == 'CON') {
            JobHelper::runJobById($jobId);
        }
    }
}
```

此外可以看到website與execType預設為 api 與 WEB。

這裡有一點要注意，公司柱列是使用database，如果柱列之後改成redis上面運行要注意。

此外看到execType分別有兩種模式，一種是 WEB端、另一種是CON

WEB跟CON其實都是去呼叫 JobHelper 中的 runJobById 這支 function，其目的是為了排程分流
