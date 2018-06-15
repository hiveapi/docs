# Validation

HiveApi internally uses the powerful [Laravel validation](https://laravel.com/docs/validation) system. However, HiveApi 
validations must be defined within the `Requests` components, since every request might have different rules.

The Validations rules are automatically applied, once injecting the Request in the Controller. Requests helps 
validating data sent to the API, accessibility, ownership and more...

## Request

Consider the following example for a validation within a Request

```php
<?php

namespace App\Containers\User\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class RegisterUserRequest extends Request
{
    /**
     * @return  array
     */
    public function rules()
    {
        return [
            'email'    => 'required|email|max:200|unique:users,email',
            'password' => 'required|min:20|max:300',
            'name'     => 'required|min:2|max:400',
        ];
    }
}
```

### Using a Request (with Validation) within a Controller

**Usage from Controller Example:**

```php
<?php 
    public function registerUser(RegisterUserRequest $request)
    {
        // if this method is called, the Request was successfully validated!
        $user = Hive::call(RegisterUserAction::class, [$request->toTransporter()]);
        
        return $this->transform($user, UserTransformer::class);
    }
```

## Responses

If the Request data cannot be validated, the framework responds with an `Exception` similar to this

```json
{
  "errors": {
    "email": [
      "The email field is required."
    ],
    "password": [
      "The password field is required."
    ]
  },
  "status_code": 422,
  "message": "The given data failed to pass validation."
}
```
