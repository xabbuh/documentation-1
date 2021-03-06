Symfony Bundle
==============

This bundle integrate HTTPlug with the Symfony framework. The bundle helps to register services for all your clients and makes sure all the configuration is in one place. The bundle also feature a toolbar plugin with information about your requests.

This guide explains how to configure HTTPlug in the Symfony framework. See the :doc:`../httplug/tutorial` for examples how to use HTTPlug in general.

Installation
````````````

Install the HTTPlug bundle with composer and enable it in your AppKernel.php.

.. code-block:: bash

    $ composer require php-http/httplug-bundle

.. code-block:: php

    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Http\HttplugBundle\HttplugBundle(),
        );
    }

You will find all available configuration at the :doc:`full configuration </integrations/symfony-full-configuration>` page.

Usage
`````

.. code-block:: yaml

    httplug:
        plugins:
            logger: ~
        clients:
            acme:
                factory: 'httplug.factory.guzzle6'
                plugins: ['httplug.plugin.logger']
                config:
                    verify: false
                    timeout: 2

.. code-block:: php

    $request = $this->container->get('httplug.message_factory')->createRequest('GET', 'http://example.com');
    $response = $this->container->get('httplug.client.acme')->sendRequest($request);


Web Debug Toolbar
`````````````````
.. image:: /assets/img/debug-toolbar.png
    :align: right
    :width: 260px

When using a client configured with ``HttplugBundle``, you will get debug information in the web debug toolbar. It will tell you how many request were made and how many of those that were successful or not. It will also show you detailed information about each request.

The web profiler page will show you lots of information about the request and also how different plugins changes the message. See example screen shots below.

.. image:: /assets/img/symfony-profiler/dashboard.png
    :width: 200px
    :align: left

.. image:: /assets/img/symfony-profiler/request-stack.png
    :width: 200px
    :align: left

.. image:: /assets/img/symfony-profiler/error-plugin-failure.png
    :width: 200px
    :align: left

|clearfloat|

The body of the HTTP messages is not captured by default because of performance reasons. Turn this on by changing the ``captured_body_length`` configuration.

.. code-block:: yaml

    httplug:
        toolbar:
            captured_body_length: 1000 # Capture the first 1000 chars of the HTTP body

The toolbar is automatically turned off when ``kernel.debug = false``. You can also disable the toolbar by configuration.

.. code-block:: yaml

    httplug:
        toolbar: false

You can configure the bundle to show debug information for clients found with discovery. You may also force a specific client to be found when a third party library is using discovery. The configuration below makes sure the client with service id ``httplug.clients.my_guzzle5`` is returned when calling ``HttpClientDiscovery::find()`` . It does also make sure to show debug info for asynchronous clients.

.. note::

    Ideally, you would always use dependency injection and never rely on auto discovery to find a client.

.. code-block:: yaml

    httplug:
        clients:
            my_guzzle5:
                factory: 'httplug.factory.guzzle5'
        discovery:
            client: 'httplug.clients.my_guzzle5'
            async_client: 'auto'

For normal clients, the auto discovery debug info is enabled by default. For async clients, debug is not enabled by default to avoid errors when using the bundle with a client that can not do async. To get debug information for async clients, set ``discovery.async_client`` to ``'auto'`` or an explicit client.

You can turn off all interaction of the bundle with auto discovery by setting the value of ``discovery.client`` to ``false``.

Discovery of Factory Classes
````````````````````````````

If you want the bundle to automatically find usable factory classes, install and enable ``puli/symfony-bundle``. If you do not want use auto discovery, you need to specify all the factory classes for you client. The following example show how you configure factory classes using Guzzle:

.. code-block:: yaml

    httplug:
        classes:
            client: Http\Adapter\Guzzle6\Client
            message_factory: Http\Message\MessageFactory\GuzzleMessageFactory
            uri_factory: Http\Message\UriFactory\GuzzleUriFactory
            stream_factory: Http\Message\StreamFactory\GuzzleStreamFactory



Configure Clients
`````````````````

You can configure your clients with default options. These default values will be specific to you client you are using. The clients are later registered as services.

.. code-block:: yaml

    httplug:
        clients:
            my_guzzle5:
                factory: 'httplug.factory.guzzle5'
                config:
                    # These options are given to Guzzle without validation.
                    defaults:
                        verify_ssl: false
                        timeout: 4
            acme:
                factory: 'httplug.factory.curl'
                config:
                    CURLOPT_CONNECTTIMEOUT: 4
                    CURLOPT_SSL_VERIFYHOST: false

.. code-block:: php

    $httpClient = $this->container->get('httplug.client.my_guzzle5');
    $httpClient = $this->container->get('httplug.client.acme');

    // will be the same as ``httplug.client.my_guzzle5``
    $httpClient = $this->container->get('httplug.client');

The bundle has client factory services that you can use to build your client. If you need a very custom made client you could create your own factory service implementing ``Http\HttplugBundle\ClientFactory\ClientFactory``. The built-in services are:

* ``httplug.factory.curl``
* ``httplug.factory.buzz``
* ``httplug.factory.guzzle5``
* ``httplug.factory.guzzle6``
* ``httplug.factory.react``
* ``httplug.factory.socket``

Plugins
```````

Clients can have plugins. Generic plugins from ``php-http/plugins`` (e.g. retry or redirect) can be configured globally. You can tell the client which of those plugins to use, as well as custom plugins that you configured a service for.

Additionally you can configure any of the ``php-http/plugins`` specifically on a client. For some plugins this is the only place where they can be configured.
The order in which you specify the plugins **does** matter.

.. code-block:: yaml

    // services.yml
    acme_plugin:
          class: Acme\Plugin\MyCustomPlugin
          arguments: ["%some_parameter%"]

.. code-block:: yaml

    // config.yml
    httplug:
        plugins:
            cache:
                cache_pool: 'my_cache_pool'
        clients:
            acme:
                factory: 'httplug.factory.guzzle6'
                plugins:
                    - 'acme_plugin'
                    - 'httplug.plugin.cache'
                    - 'httplug.plugin.retry'
                    - add_host:
                            host: "http://localhost:8000"
                    - header_defaults:
                            "X-FOO": bar
                    - authentication:
                            acme_basic:
                                type: 'basic'
                                username: 'my_username'
                                password: 'p4ssw0rd'


Authentication
``````````````

You can configure a client with authentication. Valid authentication types are ``basic``, ``bearer``, ``service`` and ``wsse``. See more examples at the :doc:`full configuration </integrations/symfony-full-configuration>`.

.. code-block:: yaml

    // config.yml
    httplug:
        plugins:
            authentication:
                my_wsse:
                    type: 'wsse'
                    username: 'my_username'
                    password: 'p4ssw0rd'

        clients:
            acme:
                factory: 'httplug.factory.guzzle6'
                plugins: ['httplug.plugin.authentication.my_wsse']

Special HTTP Clients
````````````````````

If you want to use the ``FlexibleHttpClient`` or ``HttpMethodsClient`` from the ``php-http/message`` package you may specify that on the client configuration.

.. code-block:: yaml

    // config.yml
    httplug:
        clients:
            acme:
                factory: 'httplug.factory.guzzle6'
                flexible_client: true

            foobar:
                factory: 'httplug.factory.guzzle6'
                http_methods_client: true

List of Services
````````````````

+-------------------------------------+-------------------------------------------------------------------------+
| Service id                          | Description                                                             |
+=====================================+=========================================================================+
| ``httplug.message_factory``         | Service* that provides the `Http\Message\MessageFactory`                |
+-------------------------------------+-------------------------------------------------------------------------+
| ``httplug.uri_factory``             | Service* that provides the `Http\Message\UriFactory`                    |
+-------------------------------------+-------------------------------------------------------------------------+
| ``httplug.stream_factory``          | Service* that provides the `Http\Message\StreamFactory`                 |
+-------------------------------------+-------------------------------------------------------------------------+
| ``httplug.client.[name]``           | There is one service per named client.                                  |
+-------------------------------------+-------------------------------------------------------------------------+
| ``httplug.client``                  | | If there is a client named "default", this service is an alias to     |
|                                     | | that client, otherwise it is an alias to the first client configured. |
+-------------------------------------+-------------------------------------------------------------------------+
| | ``httplug.plugin.content_length`` | | These are plugins that are enabled by default.                        |
| | ``httplug.plugin.decoder``        | | These services are private and should only be used to configure       |
| | ``httplug.plugin.error``          | | clients or other services.                                            |
| | ``httplug.plugin.logger``         |                                                                         |
| | ``httplug.plugin.redirect``       |                                                                         |
| | ``httplug.plugin.retry``          |                                                                         |
| | ``httplug.plugin.stopwatch``      |                                                                         |
+-------------------------------------+-------------------------------------------------------------------------+
| | ``httplug.plugin.cache``          | | These are plugins that are disabled by default and only get           |
| | ``httplug.plugin.cookie``         | | activated when configured.                                            |
| | ``httplug.plugin.history``        | | These services are private and should only be used to configure       |
|                                     | | clients or other services.                                            |
+-------------------------------------+-------------------------------------------------------------------------+

\* *These services are always an alias to another service. You can specify your own service or leave the default, which is the same name with `.default` appended.*


Usage for Reusable Bundles
``````````````````````````

Rather than code against specific HTTP clients, you want to use the HTTPlug ``Client`` interface. To avoid building your own infrastructure to define services for the client, simply ``require: php-http/httplug-bundle`` in your bundles ``composer.json``. You SHOULD provide a configuration option to specify the which HTTP client service to use for each of your services. This option should default to ``httplug.client``. This way, the default case needs no additional configuration for your users, but they have the option of using specific clients with each of your services.

The only steps they need is ``require`` one of the adapter implementations in their projects ``composer.json`` and instantiating the ``HttplugBundle`` in their kernel.

.. |clearfloat|  raw:: html

    <div style="clear:left"></div>
