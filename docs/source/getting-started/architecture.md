# Architecture Pattern

The two most common architectures, used for building projects on top of HiveApi are:

- **Porto** (Route Request Controller Action Task Model Transformer).
- **MVC** (Model View Controller. The HiveApi MVC version is a little different than the standard MVC)

Porto is the HiveApi recommended architecture for building scalable APIs. However, it also support building APIs using 
the popular and well-known MVC architecture (with slight modifications).

> Heads up!
> 
> HiveApi features are written using Porto, and can be used by any architecture.

Below you will see how you can both any of the architectures to build your project.

## Porto

### Introduction

Porto is an architecture pattern that consists of 2 layers, called **Containers** and **Ship** layer.

The **Container** layer holds your application business logic.  This is similar to Modular, DDD and plugins 
architectures design. HiveApi, however, allows separating the business logic into multiple folders called **Containers**. 
The **Ship** layer, on the other hand, holds the infrastructure code (i.e., shared code between all **Containers**). 
This code is rarely modified at all.

HiveApi features themselves are developed using the Porto Software Architectural Pattern. This means, features provided 
by HiveApi live in their own Containers.

Spending 15 minutes, reading the [Porto Document](https://github.com/Mahmoudz/Porto) before getting started, is a great 
investment of time.

### The Containers Layer

Read about the `Containers` layer [here](https://github.com/Mahmoudz/Porto#Containers-Layer)

#### Removing Containers

HiveApi comes with some default containers (e.g., for `Authentication` or `User` management). All containers are 
optional and can be easily re-written or extended.

Let's say you don't want to use the built in documentation generator feature of HiveApi. In order to get rid of that 
feature you can simply delete the `Documentation` container from your application. 

To remove a container, simply delete the folder then run `composer update` to remove its dependencies.

#### Create new Container

In order to extend your application with new features, you can call follow various approaches.

**Option 1) Using the Code Generator:**

Call the command `php artisan hive:generate:container` from the command line. A wizard will guide you through the 
process of creating the most important aspects.

Refer to the [Code Generator](./../features/code-generator.html) page for more details.

<a name="manual-new-container"></a>
**Option 2) manually:**

1. Create a folder in the `app\Containers` folder.
2. Start creating components (i.e., `Actions`, `Tasks`) and wiring them all together.
3. The `Ship` layer will autoload and register everything for you.

For the autoloading to work flawlessly you **MUST** adhere to the component's naming conventions and directories. So you 
need to refer to the `documentation page` of the component when creating it.

#### Naming Conventions

- Containers names **SHOULD** start with Capital Letters. Use CamelCase to rename Containers.
- Namespace should be the same as the container name (i.e., if container name is `Printer`, the corresponding namespace 
should be `App\Containers\Printer`).
- Container MAY be named to anything however. A good practice, however, is to name it to its most important `Model` name. 

> Example
>
> If the user story is "_A `User` can create a `Books` and `Books` can have `Comments`_" then you could have 3 
Containers (i.e., `User`, `Book`, `Comments` ).

### The Ship Layer

Read about the `Ship` layer **[here](https://github.com/Mahmoudz/Porto#Port-Layer)**

## MVC 

### MVC Introduction

Due to the popularity of MVC, and the fact that many developers don't have enough time to learn about new architecture 
patterns, HiveApi also supports the MVC architecture. That is 97% compatible with the `Laravel MVC`.

Below you will learn how you can build your API on top of HiveApi, using your previous knowledge of the Laravel framework.

### Difference between Standard MVC and HiveApi MVC

The Porto architecture, does not replace the MVC architecture, but rather extends it. So `Models`, `Views`, `Routes`
and `Controllers` still exist, but in different places with a strict set of responsibilities for each component.

### Setup an HiveApi MVC Project

#### 1) First get a fresh version of HiveApi

#### 2) Create the Application

If you open `app/Containers/` you will see a list of containers, whereas each container provide some features for you. 
However, you don't need to modify them, whether you are using the Porto or MVC architecture. So forget about all these 
folders for now.

All we need is to create a new folder (i.e., a new `Container`) called `Application` (which holds your MVC application). 
This is an alternative to the `app` folder on the root of the Laravel project. This folder will hold all your `Models`, 
`Views`, `Routes`, `Controllers` files, as you know it from a regular Laravel project.

#### 3) Create route file

In Laravel 5.6, the `Route` files live in the `routes/` folder on the root of the project. But in HiveApi MVC, the routes 
files should live in:

- `app/Containers/Application/UI/API/Routes/` (for API Routes)
- `app/Containers/Application/UI/WEB/Routes/` (for WEB Routes)

Create `api.php` at `app/Containers/Application/UI/API/Routes/api.php` (i.e., Laravels `routes/api.php`)
Create `web.php` at `app/Containers/Application/UI/API/Routes/web.php` (i.e., Laravels `routes/web.php`)

In both files create all your endpoints as you would in Laravel.

> Heads up!
>
> You must use `$router->` instead of the facade `Route::` in the route files.

Example:

```php
<?php

// Use this `$router` variable instead of Route::
$router->get('/', function () {
    return view('welcome');
});

// DO not use the `Route` facade
Route::get('/', function () {
    return view('welcome');
});
```

#### 4) Create Controller

In Laravel 5.6, the `Controller` classes live in the `app/Http/Controllers/` folder. But in HiveApi MVC, the `Controller` 
classes should live in:

- `app/Containers/Application/UI/API/Controllers/Controller.php` (to handle API Routes) and **MUST** extend from `App\Ship\Parents\Controllers\ApiController`
- `app/Containers/Application/UI/WEB/Controllers/Controller.php` (to handle WEB Routes) and **MUST** extend from `App\Ship\Parents\Controllers\WebController`

#### 5) Create Models

In Laravel 5.6, the `Model` classes live in the root of the `app/` folder. But in HiveApi MVC, the `Model` classes should 
live in `app/Containers/Application/Models`.

All model **MUST** `extend` from `App\Ship\Parents\Models\Model`.

> Note the `User` Model should remain in the `User` Container (`app/Containers/User/Models/User.php`), to keep all the 
features working without any modifications.

#### 6) Create Views

In Laravel 5.6, the `View` files live in the `resources/views/` folder. In HiveApi MVC, the `View` files can live in that 
same directory or/and in this container folder `app/Containers/Application/UI/WEB/Views/`.

#### 7) Create Transformers

In Laravel 5.6, the `Transformer` classes live in the `app/Transformers/` folder. But in HiveApi MVC, the `Transformer`
classes should live in `app/Containers/Application/UI/API/Transformers/`.

Transformers, in turn, **MUST** extend from `App\Ship\Parents\Transformers\Transformer`.

#### 8) Create Service Providers

In Laravel 5.6, the `Service Provider` classes live in the `app/Providers/` folder. But in Hiveapi MVC, the `Service 
Provider` classes can live in `app/Containers/Application/Providers/`. You can, however, put them anywhere else.

If you want the `Service Providers` to be automatically loaded (without having to register it in the `config/app.php` 
file), rename your file to `MainServiceProvider.php` (full path `app/Containers/Application/Providers/MainServiceProvider.php`). 
Otherwise you can create `Service Providers` anywhere and register them manually in Laravels `app.php` configuration file.

#### 9) Create Migrations

In Laravel 5.6, the `Migration` classes live in the `database/migrations/` folder on the root of the project. In HiveApi 
MVC, the `Migration` classes can live in that same directory or/and in this container folder 
`app/Containers/Application/Data/Migrations/`.

#### 10) Create Seeds

In Laravel 5.6, the `Database Seeder` files live in the `database/migrations/` folder on the root of the project. In 
HiveApi MVC, the `Database Seeder` files can live in that same directory or/and in this container folder 
`app/Containers/Application/Data/Seeders/`.

#### More Classes

All other class types work the same way, you can refer to the documentation for where to place them and what they should 
extend. For more details you can always get in touch with us on **Slack**.

### How to use HiveApi features

HiveApi features are all provided as `Actions` & `Tasks` classes. 

- Each `Action` class has single function `run` which does one feature only.
- Each `Task` class has single function `run` which does one job only (a tiny piece of the business logic).

You can use `Actions`/`Tasks` classes anyway you want:

- Using HiveApi Facade with HiveApi caller style `$user = \Hive::call('Car@GetDriversAction', [$request->id]);`
- Using HiveApi Facade with full class name `$user = \Hive::call(GetDriversAction::class, [$request->id]);`
- Using the helper `call()` function with full class name `$user = $this->call(GetDriversAction::class, [$request->id]);`
- Using the helper `call()` function with HiveApi caller style `$user = $this->call('Car@GetDriversAction', [$request->id]);`
- Without HiveApi involvement using plain PHP `$user = $action = new GetDriversAction::class; $action->run($request->id);`
- Without HiveApi involvement using plain Laravel IoC `$user = \App::make(GetDriversAction::class)->run($request->id);`

Be creative, at the end of the day it's a class with a function.
