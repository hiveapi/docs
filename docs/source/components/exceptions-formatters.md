# Exception Formatters

In HiveApi you can format any `Exception` response the way you want, by using the `ExceptionFormatters`. They act similar 
as [`Transformers`](./../components/transformers.html) but work on `Exception` instead of "normal" objects. 

HiveApi uses [Heimdal](https://github.com/esbenp/heimdal), which allows you to format your API exceptions responses 
using Formatter classes. For more details visit the [official package documentation](https://github.com/esbenp/heimdal).

By default, HiveApi provides some basic `ExceptionFormatters` for outputting `Exceptions` in an appropriate format. 
These Formatters, however, can by modified to your specific needs. For example, in case using the `JSON API` payloads, 
you may change the provided formatters to return [JSON API Error response](http://jsonapi.org/format/#error-objects).

## Rules

- All Formatters **MUST** extend from `HiveApi\Core\Exceptions\Formatters\ExceptionsFormatter`.

## Folder Structure

```
app
    Ship
        Exceptions
            Formatters
                HttpExceptionFormatter.php
                ExceptionFormatter.php
                - ...
```

## Code Sample

```php
<?php

namespace App\Ship\Exceptions\Formatters;

use HiveApi\Core\Exceptions\Formatters\ExceptionsFormatter as CoreExceptionsFormatter;
use Exception;
use Illuminate\Http\JsonResponse;

class AuthorizationExceptionFormatter extends CoreExceptionsFormatter
{
    CONST STATUS_CODE = 403;
    
    public function responseData(Exception $exception, JsonResponse $response)
    {
        return [
            'code'        => $exception->getCode(),
            'message'     => $exception->getMessage(),
            'errors'      => 'You have no access to this resource!',
            'status_code' => self::STATUS_CODE,
        ];
    }

    function modifyResponse(Exception $exception, JsonResponse $response)
    {
        return $response;
    }

    public function statusCode()
    {
        return self::STATUS_CODE;
    }
}
```

- The `responseData()` is where you format the response. This is similar to the `transform()` function in `Transformers`.
- The `STATUS_CODE` is the status code which will be sent in header. (`status_code` could be the same as the header code).
- The `modifyResponse()` allows you to alter the response when needed. Example:

```php
<?php

    public function modifyResponse(Exception $exception, JsonResponse $response)
    {
        // append exception headers to the response headers.
        if (count($headers = $exception->getHeaders())) {
            $response->headers->add($headers);
        }

        return $response;
    }
```

## Creating Your Own Formatter

You can create and add your own Formatters (or override existing ones) at any time. All Formatters live in 
`App/Ship/Exceptions/Formatters`. By default, HiveApi provides formatters to format basic Exceptions (or HTTP 
Exceptions) as well as "common" Exceptions like `AuthenticationException` and so on.

### Registering Your Formatters

In order to inform HiveApi to use your new `AwesomeExceptionFormatter` you need to `register` it. This can be done in the 
`App/Ship/Configs/optimus.heimdal.php` configuration file. Take a look at the `optimus.heimdal.formatters` key. This 
array defines a `key-value` list that declares a mapping between an `Exception` class and the corresponding `Formatter`.

Say, you want to `register` your newly created `AwesomeExceptionFormatter` for all `HttpExceptions` add a new line to 
the **top** of this array, like so:

```php
'formatters' => [
    SymfonyException\HttpException::class => \Your\Custom\Namespace\AwesomeExceptionFormatter::class,

    // the already defined exception formatters from HiveApi
    // ...
]
```

Please note that the order of the formatters matter. When throwing an `Exception` with `throw new XException()` the 
first Formatter that matches respective criteria is used to format the `Exception`. In the respective example, your newly
created `AwesomeExceptionFormatter` would be applied to format and output the Exception to the client.
