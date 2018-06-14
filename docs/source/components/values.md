# Values

Values are short names for the known `Value Objects` which are simple objects similar to Models in the concept of 
representing data, but they do not get stored in the DB, thus they don't have "official" Ids. They also do not hold 
functionality or change any state, they just hold data.

A Value Object is an immutable object that is defined by its encapsulated attributes. We create Value Object when we 
need it to represent/serve/manipulate some data that is attached as attributes. Usually such Values are destroyed later 
when they are not needed any more.  

## Rules

- All Values **MUST** extend from `App\Ship\Parents\Values\Value`.

## Folder Structure

```shell
app
    Containers
        {container-name}
            Values
                Output.php
                Region.php
                ...
```

## Code Sample

```php
<?php

use App\Ship\Parents\Values\Value;

class Location extends Value
{
    private $lat = null;
    private $long = null;

    protected $resourceKey = 'locations';
    
    public function __construct($lat, $long)
    {
        $this->lat = $lat;
        $this->long = $long;
    }

    public function getCoordinatesAsString()
    {
        return $this->lat . ' - ' . $this->long;
    }
    
    public function getCoordinatesAsArray()
    {
        return [$this->lat, $this->long];
    }
}
```

Note that these Value objects also need to have a `$resourceKey` if you plan to output them via `Serializers` (e.g., see 
the [Response](./../getting-started/responses) page for more details).