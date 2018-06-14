# Views

Read from the [Porto SAP Documentation (#Views)](https://github.com/Mahmoudz/Porto#Views).

## Rules

- Views **SHOULD** be created inside the containers and, in turn, will be automatically available for use in the `WebControllers`.
- All Views are automatically namespaced with the lowercase name of the container.

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                WEB
                    Views
                        welcome.blade.php
                        profile.blade.php
```

## Code Sample

Take a look at the `Welcome` page, that looks like this (simplified example!)

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
<div class="container">
    <div class="content">
        <div class="title">Welcome</div>
    </div>
</div>
</body>
</html>
```

This view can be used within a `WebController` like this:

```php
<?php

namespace App\Containers\Welcome\UI\WEB\Controllers;

use App\Ship\Parents\Controllers\WebController;

class Controller extends WebController
{
    public function sayWelcome()
    {
        return view('welcome');
    }
}

```

## Namespaces

By default all the views are namespaced to the lowercase name of their respective container. For example, if a Container 
is named `Store`  and has a View `product-details`, you can access the view like this `view('store::product-details')`. 
If you try to access a view without the namespace (for example `view('just-welcome')`), it will not find your view.
