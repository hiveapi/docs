# Responses

In HiveApi you can define your own response payload or use one of the supported serializers. Currently the supported 
serializers are (`ArraySerializer`, `DataArraySerializer` and `JsonApiSerializer`) provided by 
[Fractal](http://fractal.thephpleague.com/transformers/).

By default HiveApi uses `DataArraySerializer`. Below is an example of the response payload.

```javascript
{
  "data": [
    {
      "id": 100,
      ...
      "relation 1": {
        "data": [ // multiple items
          {
            "id": 11,
			  ...
          }
        ]
      },
      "relation 2": {
        "data": { // single item
          "id": 22,
          ...
          }
        }
      }
    },
    ...
  ],
  "meta": {
    "pagination": {
      "total": 999,
      "count": 999,
      "per_page": 999,
      "current_page": 999,
      "total_pages": 999,
      "links": {
        "next": "http://api.hive.local/v1/accounts?page=999"
      }
    }
  },
  "include": [ // the other resources that may be included dynamically on request
    "xxx",
    "yyy"
  ],
  "custom": []
}
```

## Paginated Response:

When the returned data is paginated the response payload will contain a `meta` description with information about the 
pagination.

```javascript
{
  "meta": {
    "pagination": {
      "total": 999,
      "count": 999,
      "per_page": 999,
      "current_page": 999,
      "total_pages": 999,
      "links": {
        "next": "http://api.hive.local/v1/accounts?page=999"
      }
    }
  },
  "include": [ // what can be included
    "xxx",
    "yyy"
  ],
  "custom": []
}
```

## Including Resources

The `include` field in the response informs the client about relationships that can be include on request. The client, 
in turn, may tell the server by adding additional query parameter, for example `/users?include=roles`

For more details read the `Relationships` section in the [Query Parameters](./../features/query-parameters.html) page.

## Change the default Response payload:

By default, HiveApi returns the data by applying the `DataArray` Fractal Serializer (`League\Fractal\Serializer\DataArraySerializer`). 
To change this behaviour, you can adapt your `.env` file accordingly:

```text
API_RESPONSE_SERIALIZER=League\Fractal\Serializer\DataArraySerializer
```

Currently, the supported Serializers are
* `ArraySerializer`
* `DataArraySerializer`
* `JsonApiSerializer`

More details can be found at the [Fractal](http://fractal.thephpleague.com/transformers) website and 
[Laravel Fractal Wrapper](https://github.com/spatie/laravel-fractal).

In case of returning JSON data (`JsonApiSerializer`), you may also want to check some JSON response standards:

* [JSON API](http://jsonapi.org/format/) (very popular and well documented)
* [JSEND](https://labs.omniti.com/labs/jsend) (very basic)
* [HAL](http://stateless.co/hal_specification.html) (useful in case of hypermedia)

## Resource Keys

### JsonApiSerializer 

The selected serializer allows appending a `ResourceKey` to the transformed resource. You can set the `ResourceKey` in 
your response payload in 2 ways:

1. Manually set it via the respective parameter in the `$this->transform()` call. Note that this will only set the
`top level` resource key and does not affect the resource keys from `included` resources!
2. Specify it on the respective `Model` by overriding the `$resourceKey` (`protected $resourceKey = 'FooBar';`). 
If no `$resourceKey` is defined at `Model` level, the lower-cased, pluralized `ShortClassName` is used as key. For example, the 
`ShortClassName` of `App\Containers\User\Models\User::class` is simply `User`. The resulting `$resourceKey`, therefore, is
`users`.

### DataArraySerializer

By default the `object` keyword is used as a resource key for each response, and is manually defined in each transformer, 

## Error Responses Formats

Visit each feature, example the Authentication and there you will see how an unauthenticated response looks like, same 
for Authorization, Validation and so on.

## Building a Responses from the Controller

Checkout the [Controller Response Builder Helper functions](./../components/controllers.html) for more information on this 
topic.
