# Migration Files

Migration files (short name for Database Migration Files are the _version control_ of your database. They are very 
useful for generating and documenting the database tables.

## Rules

- Migrations **SHOULD** be created inside the respective `Containers`.
- Migrations will be autoloaded by HiveApi
- There is no need to publish the Database Migrations yourself. Just run the `artisan migrate` command and Laravel 
will read the Migrations from your Containers.

## Structure

```shell
app
    Containers
        {container-name}
            Data
                Migrations
                    2200_01_01_000001_create_users_table.php
                    2200_01_02_000001_add_fields_to_users_table.php
                    ...
```

## Code Samples

Below is a code sample for the  `2200_01_01_000001_create_users_table` migration file.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down()
    {
        Schema::drop('users');
    }
}
```

For more information about the Database Migrations read the [official Laravel Docs](https://laravel.com/docs/master/migrations).
