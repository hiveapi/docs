# Request Monitor

HiveApi provides a simple and easy way to monitor and log all the HTTP requests reaching your application. The request 
monitor can be very useful when testing and debugging your frontend applications who consumes your API. Especially when 
the frontend apps (mobile applications, JavaScript applications, ,..) are built by other developers.

The requests monitor is provided by the Debugger Container, more specifically by a dedicated `RequestsMonitorMiddleware` 
middleware.

## Enable Requests Logging

Set the `REQUESTS_DEBUG` to true within your `.env` file. In order for this to start displaying the results you need to 
enable the debugging mode in Laravel by setting `APP_DEBUG` to true in the `.env` as well.

## Usage

Simply tail the log file

```shell
tail -f storage/logs/debugger.log
```

## Change the Default Log File

By default everything is logged in the `debugger.log` file, to change the default file go to 
`app/Containers/Debugger/Configs/debugger.php` config file and specify a file name.

```php
<?php

/*
 |--------------------------------------------------------------------------
 | Log File
 |--------------------------------------------------------------------------
 | What to name the log file in the `storage/log` path.
 */

'log_file' => 'debugger.log',
```

This feature will not run in the `testing` environments, to enable it you need to manually edit the middleware.
