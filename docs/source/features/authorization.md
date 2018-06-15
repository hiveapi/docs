# Authorization

HiveApi provides a Role-Based Access Control (RBAC) from its Authorization Container. Behind the scenes HiveApi uses 
[Laravels Authorization](https://laravel.com/docs/master/authorization) functionality that was introduced in version 
5.1.11 with the helper package [laravel-permission](https://github.com/spatie/laravel-permission). You can always refer 
to the correspond documentation for more information.

## Usage 

Authorization in HiveApi is very simple and easy. 

1) First you need to make sure you have a seeded Super Admin, an `admin` role and optionally your custom permissions 
(usually permissions should be statically created in the code). HiveApi provides most of these features for you, you 
can find the code at any container `.../Data/Seeders/*` directory.

2) Second create some basic `Roles`, and attach some `Permissions` to these roles.

3) Now start creating `Users` (or use any existing user), and assign them to the newly created roles.

4) Finally, you need to protect your endpoints by Permissions (or/and Roles). The right place to do that is the Requests class.

## Example 

Protecting an endpoint with a specific permission

```php
<?php

namespace App\Containers\User\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class DeleteUserRequest extends Request
{

    /**
     * Define which Roles and/or Permissions has access to this request.
     *
     * @var  array
     */
    protected $access = [
        'permissions' => 'delete-users', // Accepts Array and String ['delete-users', 'create-users'],
        'roles'       => '',
    ];

    /**
     * @return  bool
     */
    public function authorize()
    {
        return $this->check([
            'hasAccess|isOwner',
        ]);
    }
}
```

For a detailed explanation of this example, please visit the [Requests](./../components/requests.html) page.

## Responses

If the authorization fails, HiveApi throws an `Exception` that may look similar to this.

```json
{
  "errors": "You have no access to this resource!",
  "status_code": 403,
  "message": "This action is unauthorized."
}
```

## Seeding Users

By default, HiveApi ships with a `Super Admin` with Access to the Admin Dashboard. This user has the role `admin` 
assigned by default. The `admin` role, however, **has no permissions assigned**. This Super Admin Credentials are:

+ email: admin@admin.com
+ password: admin

This user is seeded by `app/Containers/Authorization/Data/Seeders/AuthorizationDefaultUsersSeeder_3.php`. 
 
To give permissions to the `admin` role (or any other role), you can use the dedicated endpoints (from your custom 
Admin Interface) or use this command `php artisan apiato:permissions:toRole admin` to assign all permissions available 
in the system.

Checkout each container `Seeder` directories `app/Containers/{container-name}/Data/Seeders/`, to edit the default 
`Users`, `Roles` and `Permissions`.

## Roles & Permissions guards

By default HiveApi uses a single guard called `web` for all it's roles and permissions, you can add/edit this behavior 
and support multiple guards at any time. Refer to the 
[laravel-permission](https://github.com/spatie/laravel-permission#using-multiple-guards) package for more details.

## Permissions Inheriting with Levels

When you create a role you can set an optional parameter, called `level` (set to `0` by default). The default seeded 
`admin` role has it set to `999`.

Level allows inheriting permissions. Role with higher level is inheriting permission from roles with lower level.

Below is a short example of how it works:

Imagine, you have three roles: `User`, `Moderator` and `Admin`. 
`User` has a permission to read articles, `Moderator` can manage comments and `Admin` can create articles. 
`User` has a level 1, `Moderator` level 2 and `Admin` level 3. 
It means, `Moderator` and `Administrator` has also permission to read articles, but `Administrator` can manage 
comments as well.

```php
if ($user->getRoleLevel() > 10) {
    //
}
```

If a `User` has multiple roles, the `getRoleLevel()` method returns the highest one. If you don't need the permissions 
inheriting feature, simply ignore the optional level parameter when creating roles.
