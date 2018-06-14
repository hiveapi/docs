# Versioning 

Since Laravel does not support API versioning, HiveApi provide a very easy way to implement versioning for your API.

When creating a new API endpoint, specify the version number in the route file name following this naming format 
`{endpoint-name}.{version-number}.{documentation-name}.php`.

Example:

- `MakeOrder.v1.public.php`
- `MakeOrder.v2.public.php`
- `ListOrders.v1.private.php`

## Usage

The endpoint inside that route file will be automatically accessible by adding the version number to the URL. Example: 

- `http://api.hive.local/v1/register`
- `http://api.hive.local/v1/orders`
- `http://api.hive.local/v2/stores/123`

## Version the API in Header instead of URL

First remove the URL version prefix:

1. Edit `app/Ship/Configs/hive.php` and set the prefix to `'enable_version_prefix' => 'false'`.
2. Implement the Header versioning anyway you prefer. This is not implemented in yet.
