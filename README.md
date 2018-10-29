Docs
--------
/api/doc

Authentication
--------------
Api uses OAuth2 authentication. To authenticate your request add "Authorization" header with value "Bearer access_token"

Fixtures
--------
**User 1**\
   username: artur@gmail.com\
   password: 123

**User 2**\
   username: kirill@gmail.com\
   password: 123
   
**Access Tokens:** Access_Token_For_Artur, Access_Token_For_Kirill

CRUD operations
---------------
- [Create](#create-operation) 
- [Read](#read-operations)
- [Update](#update-operation)
- [Delete](#delete-operations)

## Basic options
 
 | Option                           | Type      | Example                                                    |
 | -------------------------------- | --------  | ---------------------------------------------------------- |
 | serialization_groups             | array     | `'serialization_groups': ['default', 'my_group']`          |
 | access_attribute                 |           |                                                            |
 | serialization_check_access       | boolean   | `'serialization_check_access': false`                      |
 | fetch_field                      | string    | `'fetch_field': 'customIdentifierField'`                   |
 | preset_filters                   | array     | `preset_filters:{availableForUser: '__USER__'}`            |
 | use_lock                         | boolean   | `'use_lock': true`                                         |
 | http_method                      | string    | `'http_method': PUT`                                       |
 | success_status_code              | string    | `'success_status_code' : Response::HTTP_CREATED`           |
 | return_entity                    | boolean   | `'return_entity' : true`                                   |
 | form_options                     | array     | `'form_options': {'validation_groups': ['create']}`        |
 | before_save_events               | array     | `'before_save_events': ['action.before_save_item']`        |
 | after_save_events                | array     | `'after_save_events': ['action.after_save_item']`          |
 | filters                          | array     | `'filters': ['user', 'categories', 'query', 'order-by']`   |
 | before_delete_events             | array     | `'before_delete_events': ['action.before_delete_item']`    |
 | default_per_page                 | integer   | `'default_per_page': 15`                                   |
 | pagerfanta_fetch_join_collection | boolean   | `'pagerfanta_fetch_join_collection': true`                 |
 | pagerfanta_use_output_walkers    | boolean   | `'pagerfanta_use_output_walkers': true`                    |
 
 
---------
## Create operation

---------------
## Read operations

There are two types of read operations: collection operations ([List](#list)) and single item ([Fetch](#fetch)) operations.

### List
Get list of items. \
Object type: collection \
HTTP method: GET 

#### Configuration
1 Add service. Example: 
```php
# config/services.yml

services:
    ...
    action.country.list:
        parent: core.action.abstract
        class: Requestum\ApiBundle\Action\ListAction
        arguments:
            - MyProject\MyBundle\Entity\Сountry
        calls:
            - ['setOptions', 
                [{
                    'filters': ['query', 'order-by', 'name', 'language'],  
                    'preset_filters':{availableForUser: '__USER__', order-by: 'createdAt|desc'},
                }]
            ]
    ...
```
Where: \
`arguments: ... ` - required arguments to the `Requestum\ApiBundle\Action\ListAction` class constructor \
`- MyProject\MyBundle\Entity\Сountry` - entity class (required) \
`['setOptions', ...]` - array of options

2 Add service to routing. Example: 
 ```php
 # config/routing.yml
...
country.list:
    path: /country
    methods: GET
    defaults: { _controller: action.country.list:executeAction }
...
```
#### Request example
```php
http://mysite/country?expand=cities
```
#### Response example 
```php
{
    total: 2,
    entities: 
        [
            {
                id: 1,
                name: 'Australia',
                language: 'English',
                population: 25103900,               
                status: true,
                createdAt: "2018-03-22T10:49:07+00:00",
                cities: 
                    [
                        {
                            id: 11,
                            name: 'Sydney',
                            districts: [112, 113],
                            population: 25103900,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        },
                        {
                            id: 12,
                            name: 'Melbourne',
                            districts: [122],
                            population: 4850740,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        },
                        {
                            id: 13,
                            name: 'Brisbane',
                            districts: [131, 132],
                            population: 2408223,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        }
                    ]
            },
            {
                id: 2,
                name: 'Spain',
                language: 'Spanish',
                population: 46700000,               
                status: false,
                createdAt: "2018-03-23T10:49:07+00:00",
                cities: 
                    [
                        {
                            id: 21,
                            name: 'Madrid',
                            districts: [212],
                            population: 3165235,
                            isCapital: true,
                            createdAt: "2018-03-24T10:49:07+00:00"
                        },
                        {
                            id: 22,
                            name: 'Barcelona',
                            districts: [224],
                            population: 1602386,
                            isCapital: false,
                            createdAt: "2018-03-24T10:49:07+00:00"
                        },
                        {
                            id: 23,
                            name: 'Valencia',
                            districts: [231, 232],
                            population: 786424,
                            isCapital: false,
                            createdAt: "2018-03-24T10:49:07+00:00"
                        }
                    ]
            }
        ]
}
```

#### Additional functionality

#### Sorting
To sort by entity one may add the property name and sort order to the request (pattern: 'field|order'). \
Example ```GET /country?order-by=id|asc```

#### Pagination
[Pagerfanta](https://github.com/whiteoctober/Pagerfanta) is used for pagination and works with DoctrineORM query objects only. \
ApiBundle pagination configured with default options ```pagerfanta_fetch_join_collection = false``` and ```pagerfanta_use_output_walkers = null``` (This setting can be changed in options, see ListAction Additional options). \
One use pagination add ```page={int}``` and ```per-page={int}``` to the request.\
Example: ```GET /country?page=1&per-page=15```

#### Count only
To get the count of query results only one may add ```defaults: { count-only: true }``` 
to the routing config. 

#### Expand
One can use the related entity references instead of full value in the response (can be expanded on demand) by adding annotation ```@Reference``` to entity property, for example:
```php
# YouBundle\Entity\Country.php;
use Requestum\ApiBundle\Rest\Metadata;

class Country
{
    ...
    /**
     * @ORM\OneToMany
     * @Reference
     **/
    protected $cities;
    ...
}
```
Add ```expand``` to the request for expand reference. For multiple references expansion according fields should be separated be commas(NB: no spaces needs here!).
One use the point for expand the field in associated entity. \
Example: 
```php
// GET /country?expand=cities&name=Australia
{
    total: 1,
    entities: 
        [
            {
                id: 1,
                name: 'Australia',
                language: 'English',
                population: 25103900,               
                status: true,
                createdAt: "2018-03-22T10:49:07+00:00",
                cities: 
                    [
                        {
                            id: 11,
                            name: 'Sydney',
                            districts: [112, 113],
                            population: 25103900,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        },
                        {
                            id: 12,
                            name: 'Melbourne',
                            districts: [122],
                            population: 4850740,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        },
                        {
                            id: 13,
                            name: 'Brisbane',
                            districts: [131, 132],
                            population: 2408223,
                            isCapital: false,
                            createdAt: "2018-03-23T10:49:07+00:00"
                        }
                    ]
            }
        ]
}

// GET /country?name=Australia
{
    total: 1,
    entities: 
    [
        {
            id: 1,
            name: 'Australia',
            language: 'English',
            population: 25103900,               
            status: true,
            createdAt: "2018-03-22T10:49:07+00:00",
            cities: 
                [11, 12, 13] 
        }
    ]
}

```

####  Additional options 

#### Default per page (Pagination)
Results per page (20 by default). 
Add ```'default_per_page': {int}``` to options for change.

#### Fetch join collection (Pagination)
Whether the query joins a collection join collection (boolean, false by default).\
Add ```'pagerfanta_fetch_join_collection': true``` to options for change.

#### Use output walkers (Pagination)
Whether to use output walkers pagination mode (boolean, null by default).\
Add ```'pagerfanta_use_output_walkers': true``` to options for change.

#### Serialization
One can serialize properties that belong to chosen groups only. \
One use ```@Serializer\Groups({"firstgroup"})``` annotation to add some field to group. \
To serialize some groups, add them to the option. Example: ```'serialization_groups': ['firstgroup']```

#### Filters

##### Query filter
Available text search in some fields (```LIKE```). Supports wildcards (```*suffix```, ```prefix*```, ```*middle*```) \
To add fields you need to edit the ```createHandlers()``` method in the entity repository. \
Add a filter using ```'filters': ['query']``` option. \
Example:
```php
# YourBundle\Repository\CountryRepository.php

class CountryRepository extends EntityRepository implements FilterableRepositoryInterface
{
    use ApiRepositoryTrait;
    ...
    /**
     * @inheritdoc
     */
    protected function createHandlers()
    {
        return [
            new SearchHandler([
               'language',
               'cities.name' // use the dot for fields of related entities
            ])
        ];
    }
    ...
}
```
Sample query with filter: ``` GET /country?query=*nglish```

##### Sorting
One may add the property name and sort order to the request (pattern: 'field|order') to sort. Example:
```'order-by': 'createdAt|desc'```

##### Filter by properties
Such filtering by entity is available:
- exact matching (Example: ```GET /country?status=false```);
- using comparison operators (`````!=, <=, <>````` etc.) and ```*```, ```'is_null_value'```, ```is_not_null_value``` 
(Example: ```GET /country?status=!=true``` )

To change the filtering logic by association entities or existing filters, one may to make changes to the ```getPathAliases()``` method in the entity repository. 
Example:
```php
# YourBundle\Repository\CountryRepository.php

use Doctrine\ORM\EntityRepository;
use Requestum\ApiBundle\Repository\ApiRepositoryTrait;
use Requestum\ApiBundle\Repository\FilterableRepositoryInterface;

class CountryRepository extends EntityRepository implements FilterableRepositoryInterface
{
    use ApiRepositoryTrait;

    /**
     * @return array
     */
    protected function getPathAliases()
    {
        return [
            ...
            'city' => '[cities][id]',  
            ...
        ];
    }
}
```
##### Custom filter
To create custom filters one need: \
1 Add new Handler. Example:
```php
# YourBundle\Filter\CustomFilteHandler

use Requestum\ApiBundle\Filter\Handler\AbstractHandler;

class CustomFilteHandler extends AbstractHandler
{
    public function handle(QueryBuilder $builder, $filter, $value)
    {
      ... // Some filter logic
    }
    
    protected function getFilterKey()
    {
        return 'customFilterName'; // filter name
    }
}
``` 
2 Add handler to item repository. Example:
```php
# YourBundle\Repository\CountryRepository.php

class CountryRepository extends EntityRepository implements FilterableRepositoryInterface
{
    use ApiRepositoryTrait;
    ...
    /**
     * @inheritdoc
     */
    protected function createHandlers()
    {
        return [
            new CustomFilterHandler()
        ];
    }
    ...
}
```
3 Add custom filter to service. Example:
```php
services:
    ...
    action.country.list:
        parent: core.action.abstract
        class: Requestum\ApiBundle\Action\ListAction
        arguments:
            - MyProject\MyBundle\Entity\Country
        calls:
            - ['setOptions', [{'filters': ['customFilterName']}]]
    ...
```

#### Preset filters
Preset filters with values using by ```preset_filters``` option. \
String value ```__USER__```  can be used as alias for the current authorized user.
Example:
```php
['setOptions', [{'filters':['name'], 'preset_filters':{'status' : 'true', 'user': '__USER__'}}]]
```



### Fetch
Get single item by identifier. \
Object type: item \
HTTP method: GET 

#### Configuration
1 Add service. Example: 
```php
# config/services.yml

services:
    ...
    action.country.fetch:
        parent: core.action.abstract
        class: Requestum\ApiBundle\Action\FetchAction
        arguments:
            - MyProject\MyBundle\Entity\Сountry
        calls:
            calls:
            - ['setOptions', 
                [{
                    'serialization_groups':['full_post', 'default'],
                    'fetch_field': 'email', 
                }]
            ]
    ...
```
Where: \
`arguments: ... ` - required arguments to the `Requestum\ApiBundle\Action\FetchAction` class constructor \
`- MyProject\MyBundle\Entity\Сountry` - entity class (required) \
`['setOptions', ...]` - array of options

2 Add service to routing. Example: 
 ```php
 # config/routing.yml
...
country.fetch:
    path: /country/{id}
    methods: GET
    defaults: { _controller: action.country.fetch:executeAction }
...
```
#### Request example
```php
http://mysite/country/2?expand=cities
```
#### Response example 
```php
{
    id: 1,
    name: 'Australia',
    language: 'English',
    population: 25103900,               
    status: true,
    createdAt: "2018-03-22T10:49:07+00:00",
    cities: 
        [
            {
                id: 11,
                name: 'Sydney',
                districts: [112, 113],
                population: 25103900,
                isCapital: false,
                createdAt: "2018-03-23T10:49:07+00:00"
            },
            {
                id: 12,
                name: 'Melbourne',
                districts: [122],
                population: 4850740,
                isCapital: false,
                createdAt: "2018-03-23T10:49:07+00:00"
            },
            {
                id: 13,
                name: 'Brisbane',
                districts: [131, 132],
                population: 2408223,
                isCapital: false,
                createdAt: "2018-03-23T10:49:07+00:00"
            }
        ]
}
```
#### Additional functionality
#### Expand
One can use the related entity references instead of full value in the response. See [Expand in ListAction](#expand)


#### Additional options
#### Serialization
One can serialize properties that belong to chosen groups only. See [Serialization in ListAction](#serialization)

#### Fetch field
Possibility to use unique property of entity as an identifier (```id``` by default). Add option `'fetch_field'` to change it. \
Example: `'fetch_field': 'email'`.

#### Access attribute
Voters is used for check user permissions. To change the check logic one need: \
1 Make changes to the security config (if needed). Example: 
```php
# config/security.yml
...
access_decision_manager:
    strategy: unanimous
    allow_if_all_abstain: true
...
```
2 Add new voter. Example:
```php
# YourBundle\Security\Entity\CustomVoter.php

use Requestum\ApiBundle\Security\Authorization\AbstractEntityVoter;

class CustomVoter extends AbstractEntityVoter
{
    /**
     * @param $value
     * @param User $user
     *
     * @return bool
     */
    protected function voteOnProperty($value, UserInterface $user = null)
    {
        // some logic
    }
}
```
3 Add new voter to services. Example:
```php
# config/services.yml

services:
...
    voter.contry.owner:
        class: YourBundle\Security\Entity\CustomVoter
        arguments: [[fetch, create, update, delete], YourBundle\Entity\Country, null]
        tags:
            - { name: security.voter }
...
```
4 Add `'access_attribute'` to service config for set attributes to check user permissions (as needed). \
`'access_attribute' : 'fetch'` by default.


---------
### Update operation

----------
### Delete operations




