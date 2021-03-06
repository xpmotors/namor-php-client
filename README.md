# Namor的PHP客户端

## install
php version >= 7.0
```bash
$ composer require xpmotors/namor-php-client
```
php version >= 5.4 , <7.0
```bash
$ composer require xpmotors/namor-php-client --ignore-platform-reqs
```

## Features
- 支持namor配置变更的实时获取
- 支持拉取配置后自定义的回调处理

## Usage
客户端以cli的方式后台启动执行，支持namor配置的适时获取，并将配置保存在指定的目录供应用程序读取解析

### 客户端示例代码
```php
#!/usr/bin/env php
<?php
require 'vender/autoload.php'; // autoload
use Org\Xpmotors\Namor\Client\NamorClient;

$server = getenv('CONFIG_SERVER');
$appId = getenv('APPID');
$namespacesStr = getenv('NAMESPACE');
$namespacesStr = $namespacesStr ?: 'application,system'; // set up the default values 
$namespaces = explode(',', $namespacesStr); // get namespaces from env

$namor = new NamorClient($server, $appId, $namespaces);

if ($clientIp = getenv('CLIENTIP')) {
    $namor->setClientIp($clientIp);
}

ini_set('memory_limit','128M');
$pid = getmypid();
echo "start [$pid]\n";
$restart = true; //auto start if failed
do {
    $error = $namor->start();
    if ($error) echo('error:'.$error."\n");
}while($error && $restart);
```

### 配置管理

拉取的配置默认保存在脚本所在目录，每个namespace的配置以`namorConfig.{$namespaceName}.php`的方式命名保存

### Docker环境客户端自启动

在docker的启动脚本中加入启动代码，一般的php容器启动脚本是docker-php-entrypoint
```bash
if [ -f "/path/to/start.php" ]; then
    namor_ps=$(ps -aux | grep -c "php /path/to/start.php")
    if [ $namor_ps -eq 1 ]; then
        php /path/to/start.php &
    fi
fi
```
