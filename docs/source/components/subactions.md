# SubActions

Read from the [Porto SAP Documentation (#Sub-Actions)](https://github.com/Mahmoudz/Porto#Sub-Actions).

## Rules

- All SubActions **MUST** extend from `App\Ship\Parents\Actions\SubAction`.

## Folder Structure

```shell
app
    Containers
        {container-name}
            Actions
                ValidateAddressSubAction.php
                BuildOrderSubAction.php
                ...
```

### Code Sample

`ValidateAddressSubAction`

```php
<?php

namespace App\Containers\Shipment\Actions;

use App\Containers\Recipient\Models\Recipient;
use App\Containers\Recipient\Tasks\UpdateRecipientTask;
use App\Containers\Shipment\Tasks\ValidateAddressWithEasyPostTask;
use App\Containers\Shipment\Tasks\ValidateAddressWithMelissaDataTask;
use App\Ship\Parents\Actions\SubAction;

class ValidateAddressSubAction extends SubAction
{
    public function run(Recipient $recipient)
    {
        $hasValidAddress = true;

        $easyPostResponse = Hive::call(ValidateAddressWithEasyPostTask::class, [$recipient]);

        // ...
    }
}
```

> Heads up!
> 
> Every feature available for `Actions`, is also available for `SubActions`.
