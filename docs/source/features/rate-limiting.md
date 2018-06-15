# Rate Limiting (API Throttling)

HiveApi uses the default Laravel middleware for rate limiting (throttling). All API endpoints are throttled to prevent 
abuse and ensure stability. The exact number of calls that your client can make per day varies based on the type of 
request you are making.

The rate limit window is `1` minute per endpoint, with most individual calls allowing for `30` requests in each window. 
In other words, each client is allowed to make `30` calls per endpoint every `1` minute (for each unique access token).

You can change these values within the `app/Ship/Configs/hive.php` config file, or in your `ENV` file.

```php
'throttle' => [
    'enabled' => env('API_RATE_LIMIT_ENABLED', true),
    'attempts' => env('API_RATE_LIMIT_ATTEMPTS', '30'),
    'expires' => env('API_RATE_LIMIT_EXPIRES', '1'),
]
```

```
API_RATE_LIMIT_ENABLED=true
API_RATE_LIMIT_ATTEMPTS=30
API_RATE_LIMIT_EXPIRES=1
```

HiveApi automatically returns how many hits you can preform on an endpoint in the response header:

```
X-RateLimit-Limit : 30
X-RateLimit-Remaining : 29
```

## Enable/Disable Rate Limiting:

The API rate limiting middleware is enabled by default and applied to all endpoints by default. To disable it this 
feature set `API_RATE_LIMIT_ENABLED` to `false` in your `.env` file.
