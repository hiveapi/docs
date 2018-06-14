# Caching

## Enable / Disable Eloquent Query Caching

By default caching is disabled. To enable the latter, go to `app/Ship/Configs/repository.php` config file and set 
`cache.enabled => true`, or set it from the `.env` file using the `ELOQUENT_QUERY_CACHE` key.

Clients can skip the query caching and request new data by passing specific parameter to the endpoint. For more details, 
we refer to the [Query Parameters](./../features/query-parameters.html) page.

More details can be found at the [offical package documentation](https://github.com/andersao/l5-repository#cache-config).

## Change Different Caching Settings

You can use different cache setting for each repository. To set cache settings on each repository, first the caching 
must be enabled. Second you need to set up some properties on the repository class to override the default values.

You don't need to use the `CacheableRepository` trait or implement the `CacheableInterface` since they already exist on 
the `AbstractRepository` class (`App\Ship\Parents\Repositories\Repository`).
