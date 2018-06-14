# Seeders

Seeders (short name for Database Seeders) are classes made to seed the database with real data. Tthis data usually 
should exist in the application after the installation, for example the default `Users`, their associated `Roles` and 
`Permissions`, or a list of available `Countries` for shippment.

## Principles

- Seeders **SHOULD** be created in the Containers. 
- If the container is using a third-party package that publishes a Seeder class, this class should be manually placed 
in the Container that make use of it. Do not rely on the package to place it on its right location.

## Rules

- Seeders **SHOULD** be in the right directory inside the container to be loaded.
- To avoid any conflict between containers seeders classes, you **SHOULD** always prepend the Seeders of each container 
with the container name. For example use `UserPermissionsSeeder`, `ItemPermissionsSeeder`). If 2 seeders classes have 
the same name but live in different containers, one of them will not be loaded.
- If you wish to order the seeding of the classes, you can just append `_1`, `_2` to your classes.

## Folder Structure

```
app
    Containers
        {container-name}
            Data
                Seeders
                    ContainerNameRolesSeeder_1.php
                    ContainerNamePermissionsSeeder_2.php
                    ...
```

## Code Samples

`RoleSeeder`

```php
<?php

namespace App\Containers\Order\Data\Seeders;

use App\Ship\Parents\Seeders\Seeder;
use HiveApi\Core\Foundation\Facades\Hive;

class OrderPermissionsSeeder_1 extends Seeder
{

    public function run()
    {
        Hive::call('Authorization@CreatePermissionTask', ['approve-reject-orders']);
        Hive::call('Authorization@CreatePermissionTask', ['find-orders']);
        Hive::call('Authorization@CreatePermissionTask', ['list-orders']);
        // ...
    }
}
```

Note that one `Seeder` class may seed multiple `Model` classes.

## Running the Seeders

After registering the `Seeders` you can run this command:

```shell
php artisan db:seed
```

To run specific `Seeder` class you can specific its class in the parameter as follow:

```shell
php artisan db:seed --class="App\Containers\X\Data\Seeders\YourCustomSeeder"
```

### Migrate & Seed at the same time

```shell
php artisan migrate --seed
```

For more information about the Database Seeders read the [official Laravel documentation](https://laravel.com/docs/master/seeding).

## HiveApi Seeder Commands

### Testing

It's useful sometimes to create a big set of testing data. HiveApi facilitates this task:

1. Open `app/Ship/Seeders/SeedTestingData.php` and write your testing data here.
2. Run this command any time you want this data available (example at staging servers):

```shell
php artisan hive:seed:test
```

### Deployment

HiveApi also provides a `Seeder` for your production data.  You can call the artisan command

```shell
php artisan hive:seed:deploy
```
