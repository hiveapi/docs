# Exceptions

Exceptions are classes the handles errors, and helps developers debug their code in a more efficient way.

## Principles

- Exceptions **CAN** be thrown from anywhere in the application.
- Exceptions **SHOULD** be created inside the Containers. However, generic Exceptions **CAN** be created in the `Ship` layer.

## Rules

- All Exceptions **MUST** extend `App\Ship\Parents\Exceptions\Exception`.
- Shared (generic) Exceptions between all Containers **SHOULD** be created in the `Ship` (i.e., `app/Ship/Exceptions/*`).
- Every Exception **SHOULD** have two properties `httpStatusCode` and `message`. Both properties will be displayed when 
an error occurs. You can override those values while throwing the error.

## Folder Structure

```
app
    Containers
        {container-name}
            Exceptions
                AccountFailedException.php
    Ship
        Exceptions
            IncorrectIdException.php
            InternalErrorException.php
```

## Code Samples

```php
<?php

namespace App\Containers\User\Exceptions;

use App\Ship\Parents\Exceptions\Exception;
use Symfony\Component\HttpFoundation\Response;

class AccountFailedException extends Exception
{
    public $httpStatusCode = Response::HTTP_CONFLICT;

    public $message = 'Failed creating new User.';
    
    public $code = 4711;
}
```

```php
<?php

namespace App\Ship\Exceptions;

use App\Ship\Parents\Exceptions\Exception;
use Symfony\Component\HttpFoundation\Response as SymfonyResponse;

class InternalErrorException extends Exception
{
    public $httpStatusCode = SymfonyResponse::HTTP_INTERNAL_SERVER_ERROR;

    public $message = 'Something went wrong!';
}
```

You can also add custom data to your `Exception`.

```php
<?php

namespace App\Ship\Exceptions;

use App\Ship\Parents\Exceptions\Exception;
use Symfony\Component\HttpFoundation\Response as SymfonyResponse;

class AwesomeExceptionWithCustomData extends Exception
{
    public $httpStatusCode = SymfonyResponse::HTTP_INTERNAL_SERVER_ERROR;

    public $message = 'Something went wrong!';
    
    public $code = 1234;
    
    /*
     * Everything you add here will be automatically added to the ExceptionFormatter on the top level!
     * You can define any structure you want or maybe include translated messages
     */
    public function addCustomData() {
        return [
            'title' => 'nice',
            'description' => 'one fancy description here',
            'foo' => true,
            'meta' => [
                'bar' => 1234,
            ]
        ];
    }
}
```

#### Throwing an Exception in your Application

```php
<?php

throw new AccountFailedException();
```
#### Usage
##### With Log for Debugging:

```php
<?php

throw (new AccountFailedException())->debug($e); // debug() accepts string or \Exception instance
```

##### Overriding the default `message`:

```php
<?php

throw new AccountFailedException('I am the message to be displayed for the user');
```

##### Overwriting pre-set Custom Data

```php
<?php

throw (new AwesomeExceptionWithCustomData())->overrideCustomData(['foo' => 'bar']);
```

## Application Error Codes

HiveApi provides a convenient way to manage all `application error codes` in one place. Therefore, HiveApi provides, 
amongst others, the `\App\Ship\Exceptions\Codes\ApplicationErrorCodesTable` class, which already holds various 
information for multiple errors.

Thereby, one error look like this:

```php
<?php
const BASE_GENERAL_ERROR = [
	'code' => 1001,
	'title' => 'Unknown / Unspecified Error.',
	'description' => 'Something unexpected happened.',
];
```

Note that the `code` is used to be sent back to the client. The `title` and `description`, however, can be used to 
automatically generate a documentation regarding all defined error codes and their meaning. Please note that this 
feature is currently not implemented but will be added later on.

### Linking Exceptions and Error Codes

In order to link an `error code` to an `Exception`, you simply need override the `useErrorCode()` method of the `Exception`.

Consider the following example:

```php
<?php 
class InternalErrorException extends Exception
{
    public $httpStatusCode = SymfonyResponse::HTTP_INTERNAL_SERVER_ERROR;

    public $message = 'Something went wrong!';
	
	public $code = 4711; // this code will be overwritten by the useErrorCode() method!

    public function useErrorCode()
    {
        return ApplicationErrorCodes::BASE_INTERNAL_ERROR;
    }
}
```

Please note that already defined `$code` values may be overwritten by the `useErrorCode()` method! Furthermore, this 
feature is completely optional - you may still use the known `public $code = 4711;` approach to manually set an error 
code.

### Defining Own Error Code Tables

Of course, HiveApi allows you to define your own `CustomErrorCodesTable`. In fact, there already exists such a file 
where you can define your own error codes. Please note that the `ApplicationErrorCodesTable` may be adapted by 
HiveApi - the others will not.

If you like to split the errors in various files, you can easily create a `UserErrorCodesTable` in respective 
namespace and define the errors accordingly. However, you need to manually "register" this code table. This can be 
achieved in the `ErrorCodeManager::getCodeTables()` method.

Now you can easily use your `UserErrorCodesTable::USER_NOT_VERIFIED` error in your `Exception` class.
