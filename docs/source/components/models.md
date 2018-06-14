# Models

Read the official [Porto SAP Documentation (#Models)](https://github.com/Mahmoudz/Porto#Models).

## Rules

- All Models **MUST** extend from `App\Ship\Parents\Models\Model`.
- If the name of a model differs from the Container name you have to set the `$container` attribute in the repository. 
[More details](./../components/repositories.md) can be found here.

## Folder Structure

```shell
app
    Containers
        {container-name}
            Models
                User.php
                Person.php
```

## Code Sample

```php
<?php

namespace App\Containers\Demo\Models;

use App\Ship\Parents\Models\Model;

class Demo extends Model
{
    protected $table = 'demos';

    protected $fillable = [
        'label',
        'user_id'
    ];

    protected $hidden = [
        'token',
    ];

    protected $casts = [
        'total_credits'     => 'float',
    ];

    protected $dates = [
        'created_at',
        'updated_at',
    ];

    public function user()
    {
        return $this->belongsTo(\App\Containes\User\Models\User::class);
    }
}
```

Notice the `Demo` Model has a relationship with the `User` Model, which lives in another Container.

## Casts
The `$casts` attribute can be used to cast any of the model's attributes to a specific type when reading from and writing 
to the database. In the shown code sample the `total_credits` is casted to `float`.

More information about the available cast-types can be found in the Laravel 
[eloquent-mutators](https://laravel.com/docs/5.6/eloquent-mutators) documentation.

Date values can be defined within the `$dates` array to be automatically parsed to a `Carbon` class.
