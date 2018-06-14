# Service Providers

Providers (short names for Service Providers) are the central place of configuring and bootstrapping a Container. They 
are the place where you register container bindings, event listeners, middlewares, routes, other providers, aliases, ... 
to the framework service container.

### Principles

- There are two types of Providers in a Container, the main `Provider` and additional (job specific) `Providers` 
(e.g., `EventsProvider`, `BroadcastsProvider`, `AuthProvider`, `MiddlewareProvider`, `RoutesProvider`).
- A Container **MAY** have one or many Providers, or **MAY** have no Provider at all.
- A Container **CAN** have only one single main Provider.
- The main Provider is the place where all other job specific Providers are registered.
- Third-party package Providers **MUST** be registered inside the Containers main service provider. The same applies to 
their Aliases.
- Providers **CAN** be registered on the `Ship` main Provider, if they are general or are intended to be used across many 
containers. The same applies to their Aliases.

## Rules

- The main Provider will be auto-registered by the `Ship` engine, so no need to register it manually anywhere.
- All Main Providers **MUST** extend from `App\Ship\Parents\Providers\MainProvider`.
- All other types of `Providers` (`EventsProvider`, `BroadcastsProvider`, `AuthProvider`, `MiddlewareProvider`, 
`RoutesProvider`) must extend from their parent providers `Ship/Parents/Providers/***`.
- The Main Provider **MUST** be named `MainServiceProvider` in every container.
- You **SHOULD NOT** register any `Provider` in the framework (`config/app.php`), only the `HiveApiProvider` should be 
registered there.

> Heads up!
> 
> Laravel 5.5 introduces an `auto-discovery` feature that lets you automatically register `ServiceProviders`. Due to 
the nature and structure of HiveApi applications, this features **is turned off**, because it messes up the way how 
`config` files are loaded in HiveApi. This means, that you still need to **manually** register third-party 
`ServiceProviders` in the `ServiceProvider` of a `Container`.

## Folder Structure

```
app
    Containers
        User
            Providers
                EventsServiceProvider.php
                MainServiceProvider.php
                ...
```

## Code Samples

**Main Service Provider Example:**

```php
<?php

namespace App\Containers\Excel\Providers;

use App\Ship\Parents\Providers\MainProvider;

class MainServiceProvider extends MainProvider
{

    public $serviceProviders = [
        // ...			
    ];

    public $aliases = [
        // ...
    ];

    public function register()
    {
        parent::register();

        $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
    }
}
```

> Heads up!
> 
> When defining the `register()` or `boot()` function in your Main provider "only", you must call the parent functions 
(`parent::register()`, `parent::boot()`) from your extended function.

## Register Service Providers

### Containers Main Service Provider

There is no need to register the `MainServiceProvider` anywhere, as it will be automatically registered by HiveApi. The
`MainServiceProvider`, in turn, is responsible for registering all the Containers additional (job specific) Providers.

### Containers Additional Service Providers

You **MAY** add further `Service Providers` in a `Container`. However, in order to get them loaded in the framework you 
**MUST** register them all in the `MainServiceProvider` as follow:

```php
<?php

private $containerServiceProviders = [
    AuthServiceProvider::class,
    EventsServiceProvider::class,
    // ...
    ];
```

The same rule applies to `Aliases`.

### Third party packages Service Providers

If a package requires registering its service provider in the `config/app.php`, you **SHOULD** register its service 
provider in the container where you rely on this package. However, if it is a "generic" package used by the entire 
framework and not one specific Container or feature, you can register that service provider in the 
`app/Ship/Providers/ShipProvider.php`. You should, however, **NEVER** register additional service providers in the 
`config/app.php` file.

## Laravel 5.5 Auto Discovery feature.

This feature is disabled in HiveApi so far. More details can be found [here](./../miscellaneous/faq.html).

## Information about Laravel Service Providers

By default Laravel provides some service providers in its `app/providers` directory. In HiveApi those providers have 
been renamed and moved to the Ship Layer `app/Ship/Parents/Providers/*`:

- `AppServiceProvider`
- `RouteServiceProvider`
- `AuthServiceProvider`
- `BroadcastServiceProvider`
- `EventsServiceProvider`

> Heads up!
> 
> You **SHOULD NOT** touch those providers, instead you have to extend them from a custom provider in order to modify them.
For example: the `app/Containers/Authentication/Providers/AuthProvider.php` is extending the `AuthServiceProvider` in 
order to modify it.

Those providers are not auto-registered by default, thus writing any code will not be available, unless you extend them. 
Once extended, the child provider should be registered in the containers `MainServiceProvider`, which makes the parent 
available through inheritance.

This rule, however, does not apply to the `RouteServiceProvider` since it is required by HiveApi. This Provider is 
directly registered by the `HiveApiProvider`.

Check out [how Service Providers are auto-loaded](./../miscellaneous/faq.html).
