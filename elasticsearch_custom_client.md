# How to create a custom ElasticSearch client

Akeneo proposes a few ElasticSearch client wrappers for its indices, but they cannot be used for more global ElasticSearch requests. For instance, these wrappers don't expose any API to request an index settings. They don't even expose the original ElasticSearch client.

In the following document, we are going to write a custom ElasticSearch client, that will give you more flexibility.
As our example will remain simple, this client will have two public methods:
- one that return the wrapped ElasticSearch client as is
- one that will return two settings of a given index

## Writing the client
```php
<?php


namespace AppBundle\ElasticSearch;


use Akeneo\Bundle\ElasticsearchBundle\IndexConfiguration\Loader;
use Elasticsearch\ClientBuilder;

class Client
{
    /** @var ClientBuilder  */
    private $builder;

    /** @var array  */
    private $hosts;

    /** @var \Elasticsearch\Client  */
    private $client;

    /**
     * Client constructor.
     * @param ClientBuilder $builder
     * @param array $hosts
     */
    public function __construct(ClientBuilder $builder, array $hosts)
    {
        $this->builder = $builder;
        $this->hosts = $hosts;

        $builder->setHosts($this->hosts);
        $this->client = $builder->build();
    }

    /**
     * @return \Elasticsearch\Client
     */
    public function get() : \Elasticsearch\Client
    {
        return $this->client;
    }

    /**
     * @param string $indexName
     * @return array
     */
    public function getPaginationSettings(string $indexName) : array
    {
        $settings = $this->client->indices()->getSettings(['index' => $indexName, 'name' => 'index.max*']);

        return $settings[$indexName]['settings']['index'];
    }
}
```
## Registration
Even if services are autowired, the Client constructor need an array of hosts. As this cannot be guessed by the autowiring system, it has to be defined in the `services.yml` file.
```yaml
services:
  # default configuration for services in *this* file
  _defaults:
    autowire: true
    autoconfigure: true
    public: false

  # makes classes in src/AppBundle available to be used as services
  AppBundle\:
    resource: '../../src/AppBundle/*'
    # you can exclude directories or files
    # but if a service is unused, it's removed anyway
    exclude: '../../src/AppBundle/{Entity,Repository}'

  AppBundle\ElasticSearch\Client:
    arguments:
      $hosts: ["%index_hosts%"]
```
## Usage
The new Client can be used as any other service.
```php
<?php
namespace AppBundle\Controller;
use AppBundle\ElasticSearch\Client;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\JsonResponse;
class ElasticSearchController extends Controller
{
    /**
     * @param Client $client
     * @return JsonResponse
     */
    public function getPaginationSettingsAction(Client $client)
    {
        return new JsonResponse($client->getPaginationSettings($this->getParameter('product_index_name')));
    }
}
```
## Final word
This is a very basic example, that obviously may be extended to fit any need.
