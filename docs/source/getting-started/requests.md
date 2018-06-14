# Requests

## Form Content Types (W3C)

By default HiveApi is configured to encode simple text/ASCII data as `application/json`. However, it does support other 
types as well. 

### JSON Payload

To tell the web server that you are sending JSON formatted payload (`{"name" : "John Doe", "age": 25}`), you need to 
add the `Content-Type : application/json` request header.

### ASCII payload

To tell the server that you are sending simple text/ASCII payload (`name=John+Doe&age=25`), you need to add the 
`Content-Type : x-www-form-urlencoded` request header.

## HTTP Request Headers

| Header        | Value Sample                        | When to send it                                                              |
|---------------|-------------------------------------|------------------------------------------------------------------------------|
| Accept        | `application/json`                  | **MUST** be sent with every endpoint.                                        |
| Content-Type  | `application/x-www-form-urlencoded` | **MUST** be sent when passing Data.                                          |
| Authorization | `Bearer {Access-Token-Here}`        | **MUST** be sent whenever the endpoint requires (Authenticated User).        |
| If-None-Match | `811b22676b6a4a0489c920073c0df914`  | **MAY** be sent to indicate a specific `ETag` of an prior `Request` to this endpoint. If both `ETags` match (i.e., are the same) a `HTTP 304` (`Not Modified`) is returned. |

> Heads Up!
> 
> Normally you should include the `accept : application/json` HTTP header when you call a JSON API. However, in HiveApi
> you can force your users to send `application/json` by setting `'force-accept-header' => true,` in 
> `app/Ship/Configs/hive.php` or allow them to skip it completely by setting the `'force-accept-header' => false,`. 
> By default this flag is set to true.

## Calling Endpoints

### Calling unprotected Endpoints (example):

```shell
curl -X POST -H "Accept: application/json" -H "Content-Type: application/x-www-form-urlencoded; -F "email=admin@admin.com" -F "password=admin" -F "=" "http://api.hive.local/v1/register"
```

<a name="call-protected-EP"></a>
### Calling protected Endpoints by passing a Bearer Token (example):

```shell
curl -X GET -H "Accept: application/json" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..." -H "http://api.hive.local/v1/users"
```
