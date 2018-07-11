# Authentication

Middlewares are probably the best solution to apply a solid `Authentication` for your API. With HiveApi you can use two 
pre-defined `Authentication Middlewares`, to protect your endpoints:

- API Authentication: `auth:api`
- Web Authentication: `auth:web`

## API Authentication (with OAuth 2.0)

To protect an API endpoints from being accessed by unauthenticated users, you may apply the `auth:api` middleware.

```php
<?php

$router->get('secret/info', [
    'uses'       => 'Controller@getSecretInfo',
    'middleware' => [
        'auth:api',
    ],
]);

```

All endpoints that are protected with the `auth:api` middleware are only accessible when sending them a valid `access 
token`. The `auth:api` middleware is provided by the official [Laravel Passport](https://laravel.com/docs/passport) 
package. So you can read its documentation for more details.

### Overview  

OAuth lets you authenticate using different methods, these methods are called `grants`. In order how to decide, which 
grant type you should use, please refer to [this website](https://oauth2.thephpleague.com/authorization-server/which-grant/) 
and keep reading this documentation.

**Definitions**
- The client credentials are `client_id` & `client_secret`.
- The proxy is an endpoint, that you should call instead of calling the OAuth server endpoints directly. The proxy 
endpoint, in turn, will append the client credentials to your request and calls the OAuth server for you, then returns 
its response back to the client. Each `first-client` application should have its own proxy endpoints (at least one of 
`/login` and one of `/refresh-token`). By default, HiveApi provides an `Admin Web Client` endpoint.

> You can Login to the first party app with proxy or without proxy, while for the third party you only need to login 
> without proxy. (same apply to refreshing token).
>
> For first party apps:
> - With Proxy << best and easiest way, (requires manually generating clients creating proxy endpoints for each client)
> - Without Proxy << if your frontend is not exposing the client credentials, you can call the Auth server endpoints 
directly without proxy.
>
> For third party apps:
> - Without Proxy << you don't need a proxy for the third party clients as they usually integrate with your API from 
the backend side which protects the client credentials.

### A: First-Party Clients

First-party clients (i.e., your own frontend / mobile / web / ... application) usually consumes your private API. Those 
clients need to use the `Resource Owner Credentials Grant` (a.k.a `Password Grant Tokens`).

When this grant type is used, your server needs to authenticate the client application first (ensuring the request is 
sent from your trusted frontend application) and then needs to check if the user credentials are correct (ensuring the 
user is registered and has the proper access rights), before issuing an access token.

#### Note:

- On `register`, the API returns user data. You will need to log that user in (using the same credentials he passed) to 
get his `access token` and make other API calls.
- On `login`, the API returns the users `access token` and a `refresh token`. You will need to request the user data by 
making another call to the user endpoint, using his private token.

#### Example

1) Create a `password` client in your database to represent one of your applications (e.g., your mobile application). 
Call `php artisan passport:client --password` to generate a new password client.

2) After registration the user can enter his credentials (i.e., email & password) in your mobile application login screen.

3) Your mobile application should send a `POST` request to `http://api.example.develop/v1/oauth/token` containing the 
user credentials (`username` and `password`) and the client credentials (`client_id` and `client_secret`) in addition to 
the `scope` and `grant_type=password`:

##### Request

```shell
curl --request POST \
  --url http://api.example.develop/v1/oauth/token \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'username=admin%40local.host&password=admin&client_id=2&client_secret=SGUVv02b1ppQCgI7ZVeoTZDN6z8SSFLYiMOzzfiE&grant_type=password&scope='
```

##### Response

```json
{
  "token_type": "Bearer",
  "expires_in": 31536000,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUz...",
  "refresh_token": "TPSPA1S6H8Wydjkjl+xt+hPGWTagL..."
}
```

4) Your mobile application should save the `access token` and start requesting secure data, by sending the latter in 
the HTTP Header `Authorization = Bearer {Access-Token}`.

More information can be found at the official [Laravel Passport](https://laravel.com/docs/5.6/passport#password-grant-tokens) 
documentation.

> WARNING:
> 
> The Client ID and Secret should not be stored in JavaScript or browser cache, or made accessible in any way.

So in case of web applications (i.e., Angular, Vue, ... applications) you need to hide your client credentials behind a 
proxy. By default, HiveApi provides a `Login Proxy` to use for all your trusted first party clients.

#### Login with Proxy for First-Party Clients

The overall idea is to create a designated endpoint for each trusted client, to be used for login.

HiveApi, by default, has one URL ready for your Web Admin Dashboard (i.e., `clients/web/admin/login`). You can add more 
as you need for each of your trusted first party clients applications (example: `clients/web/users/login`, 
`clients/mobile/users/login`).

Behind the scene, that endpoint appends the corresponding client ID and secret to your request and makes another 
call to your OAuth server with all the required data. This way, the client does not need to send the ID and secret with 
the request. Further, the client uses his own URL, which gives even more control to which client is accessing your API. 
Then, it returns the authentication response back to the client with the proper `access tokens` in it.

> Heads up!
> 
> You have to manually extract the Client credentials from the database and put them in the `.env`, for each client.

When running `passport:install` it automatically creates one client for you with a new ID, so you can use that for your 
first app. Or you can use `php artisan passport:client --password` to generate them.

```
# Example ENV File
CLIENT_WEB_ADMIN_ID=2
CLIENT_WEB_ADMIN_SECRET=VkjYCUk5DUexJTE9yFAakytWCOqbShLgu9Ql67TI
```

#### Login without Proxy for First-Party Clients

Login from your application by sending a `POST` request to `http://api.example.develop/v1/oauth/token` with 
`grant_type=password`, the user credentials (`username` & `password`), client credentials (`client_id` & `client_secret`) 
and finally the `scope` for this token (can be empty).

### B: For Third-Party Clients

Third-party clients (custom external applications, who wants to integrate with your API) always consumes your public 
API (external API) only.

For third-party clients you need to use the `Client credentials grant` (a.k.a `Personal Access Tokens`). This grant 
type is the simplest and is suitable for machine-to-machine authentication.

With this grant type your server needs to authenticate the client application only, before issuing an access token.

#### Example 

1) User logs in to your clients application interface (an external application made for your users only), go to 
settings, create a new client (of type `personal`) and copy the ID and Secret. This step can be done via an API request 
as well, if you prefer. You may generate a personal client for testing purposes using 
`php artisan passport:client --personal`.

2) The user adds the client credentials to his "server side software" and sends a `POST` request to 
`http://api.example.develop/v1/oauth/token` containing the issued client credentials (`client_id` and `client_secret`) 
in addition to the `scope` and `grant_type=client_credentials`:

##### Request

```shell
curl --request POST \
  --url http://api.example.develop/v1/oauth/token \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=1&client_secret=y1RbtnOvh9rpA91zPI2tiVKmFlepNy9dhHkzUKle&grant_type=client_credentials&scope='
```

##### Response

```json
{
  "token_type": "Bearer",
  "expires_in": 31536000,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni...",
  "refresh_token": "ZFDPA1S7H8Wydjkjl+xt+hPGWTagX..."
}
```

3) The Client will be granted an `access token` to be saved. Then the client can start requesting secure data, by sending 
the `access token` in the HTTP Header `Authorization = Bearer {Access-Token}`.

Note: When a new user is registered, will be issued a personal Access Token automatically. Check the User 
"Registration page".

More information can be obtained via the official [Laravel Passport](https://laravel.com/docs/5.6/passport#personal-access-tokens) 
documentation

#### Login without Proxy for Third-Party Clients

We usually do not need a proxy for third-party clients as they are most likely making calls form their servers, thus 
the Client ID and Secret should be secure and not exposed to the users.

Login by sending a `POST` request to `http://api.example.develop/v1/oauth/token` with `grant_type=client_credentials`, 
Client Credentials (`client_id` & `client_secret`) and finally the `scope` (can be empty).

Once issued, you can use that `access token` to make requests to protected endpoints. The `access token` should be sent 
in the `Authorization` header of type `Bearer` (example: `Authorization = Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUz...`)

> Heads up!
> 
> There is no "session state" when using tokens for authentication

### Login With Custom Attributes

By default, HiveApi allows `User`s to login with their `email` address. However, you may want to also allow `username` 
and `phone` to login your users. 
  
Here is, how to configure and use this feature.

- You may need to adapt your database accordingly (e.g., add the respective field to the `users` table).
- You may need to adapt the `Task` that `create` a `User` object (e.g., the `CreateUserByCredentialsTask`) accordingly 
  to support the new fields. This may also affect your `Register` logic.
- Check the `App\Containers\Authentication\Configs\authentication-container` Configuration file and check the `login` 
params in order to configure this feature.
- Adapt the `ProxyApiLoginTransporter` accordingly to support your new Login Fields. These fields need to be added 
to `properties`

### Logout

Logout by sending a `DELETE` request to `http://api.example.develop/v1/logout/` containing the valid `access token` in 
the header.

```json
{
  "message": "Token revoked successfully."
}
```

## Web Authentication

To protect a `Web` endpoint from being accessible by unauthenticated users you can use the `auth:web` Middleware.

```php
<?php

$router->get('private/page', [
    'uses'       => 'Controller@showPrivatePage',
    'middleware' => [
        'auth:web',
    ],
]);
```

This middleware is provided by HiveApi and is different than the default Laravel Auth Middleware. If authentication 
failed, users will be redirected to a login page. To change the login page view go to the config file 
`app/Ship/Configs/hive.php`, and set the name of your login page there as follow:

```php
<?php

    /*
    |--------------------------------------------------------------------------
    | The Login Page URL
    |--------------------------------------------------------------------------
    */
    
    'login-page-url' => 'login',
```

This will be search for a `login.html`, `login.php`, or `login.blade.php` file.

## Refresh Token

In case your server is issuing a short-living `access tokens`, the users will need to refresh their access tokens via the 
refresh token that was provided to them when the logging in.

### Refresh Token with proxy for first-party clients

By default HiveApi provide this ready to use endpoint `http://api.example.develop/v1/clients/web/admin/refresh` for the 
Web Admin Dashboard Client to be used when you need to refresh tokens for that client. You can, of course, create as many 
other endpoints as you want for each client. See the code within 
`app/Containers/Authentication/UI/API/Routes/ProxyRefreshForAdminWebClient.v1.public.php` and create similar ones for 
each client. The most important change will be the `env('CLIENT_WEB_ADMIN_ID')` and `env('CLIENT_WEB_ADMIN_SECRET'),` 
passed to the `ProxyApiRefreshAction`.

Those proxy refresh endpoints work in 2 ways. Either by passing the `refresh_token` manually to the endpoint. Or by 
passing it with the HttpCookie. In both cases the code will work and the server will reply with a response similar to 
this:

```json
{
  "token_type": "Bearer",
  "expires_in": 31500,
  "access_token": "tnJ1eXAiOiJKV1QiLCJhbGciOiJSUzI1Zx...",
  "refresh_token": "ZFDPA1S7H8Wydjkjl+xt+hPGWTagX..."
}
```

Note that the response contains a new `access token` for login as well as a new `refresh token`.

### Refresh Token without Proxy for First-Party or Third-Party Clients

The request to `http://api.example.develop/v1/oauth/token` should contain `grant_type=refresh_token`, the `client_id` & 
`client_secret`, in addition to the `refresh_token` and finally the `scope` which could be empty.

## Force Email Confirmation

By default a user does not have to confirm his email address to be able to login. However, to enforce users to confirm 
their email (i.e., to prevent unconfirmed users from accessing the API), you can set 
`'require_email_confirmation' => true,` in `App\Containers\Authentication\Configs\authentication.php`.

When the email confirmation is enabled (i.e., value set to `true`), the API throws an exception, if the `User` is not 
yet `confirmed`.
