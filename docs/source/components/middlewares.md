# Middlewares

Middleware provide a convenient mechanism for filtering and manipulating HTTP `Requests` entering your application or 
`Responses` sent back to the client. You can read more about middlewares in the 
[official Laravel documentation](https://laravel.com/docs/middleware).

## Principles

- In HiveApi there are two types of middlewares: General (applied to all the endpoints by default) and Endpoint Middlewares 
(applied to specific endpoints).
- The Middlewares **CAN** be placed in `Ship` layer or the `Container` layer depend on their roles.

## Rules

- If the Middleware is placed inside a Container it **MUST** be registered inside that Container.
- To register Middlewares in a Container the container needs to have a `MiddlewareServiceProvider`. And like all other 
Container Providers it **MUST** be registered in the `MainServiceProvider` of that Container.
- General Middlewares (like some default Laravel Middlewares) **SHOULD** live in the Ship layer `app/Ship/Middlewares/*` 
and are registered in the `Ship` main `ServiceProvider`.
- Third-Party packages Middleware **CAN** be registered in Containers or on the Ship layer (wherever they make more sense).

For example, The `jwt.auth` middleware "provided by the JWT package" is registered in the Authentication Container 
(`Containers/Authentication/Providers/MiddlewareServiceProvider.php`).

## Folder Structure

```
app
    Containers
        {container-name}
            Middlewares
                WebAuthentication.php
    Ship
        Middleware
            Http
                EncryptCookies.php
                VerifyCsrfToken.php
```

## Code Sample

```php
<?php

namespace App\Containers\Authentication\Middlewares;

use App\Ship\Engine\Butlers\Facades\ContainersButler;
use App\Ship\Parents\Middlewares\Middleware;
use Closure;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Http\Request;

class WebAuthentication extends Middleware
{
    protected $auth;

    public function __construct(Guard $auth)
    {
        $this->auth = $auth;
    }

    public function handle(Request $request, Closure $next)
    {
        if ($this->auth->guest()) {
            return response()->view(ContainersButler::getLoginWebPageName(), [
                'errorMessage' => 'Credentials Incorrect.'
            ]);
        }

        return $next($request);
    }
}
```

### Registering a Middleware within a Container

```php
<?php

namespace App\Containers\Authentication\Providers;

use App\Containers\Authentication\Middlewares\WebAuthentication;
use App\Ship\Parents\Providers\MiddlewareProvider;
use Tymon\JWTAuth\Middleware\GetUserFromToken;
use Tymon\JWTAuth\Middleware\RefreshToken;

class MiddlewareServiceProvider extends MiddlewareProvider
{
    protected $middleware = [
    ];

    protected $middlewareGroups = [
        'web' => [
        ],

        'api' => [
        ],
    ];

    protected $routeMiddleware = [
        'jwt.auth'         => GetUserFromToken::class,
        'jwt.refresh'      => RefreshToken::class,
        'auth:web'         => WebAuthentication::class,
    ];

    public function boot()
    {
        $this->loadContainersInternalMiddlewares();
    }
}
```

### Registering a Middleware within the Ship 

```php
<?php

namespace App\Ship\Kernels;

use App\Ship\Middlewares\Http\ProcessETagHeadersMiddleware;
use App\Ship\Middlewares\Http\ProfilerMiddleware;
use App\Ship\Middlewares\Http\ValidateJsonContent;
use Illuminate\Foundation\Http\Kernel as LaravelHttpKernel;

class HttpKernel extends LaravelHttpKernel
{
    /**
     * The application's global HTTP middleware stack.
     * These middleware are run during every request to your application.
     * 
     * @var array
     */
    protected $middleware = [
        // Laravel middleware's
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Ship\Middlewares\Http\TrimStrings::class,
        \App\Ship\Middlewares\Http\TrustProxies::class,

        // CORS package middleware
        \Barryvdh\Cors\HandleCors::class,
    ];

    /**
     * The application's route middleware groups.
     * 
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Ship\Middlewares\Http\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Ship\Middlewares\Http\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            ValidateJsonContent::class,
            'bindings',
            ProcessETagHeadersMiddleware::class,
            ProfilerMiddleware::class,
            // The throttle Middleware is registered by the RoutesLoaderTrait in the Core
        ],
    ];

    /**
     * The application's route middleware.
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'can'      => \Illuminate\Auth\Middleware\Authorize::class,
        'auth'     => \Illuminate\Auth\Middleware\Authenticate::class,
    ];
}
```
