# slim-newrelic

[![Build Status](https://travis-ci.org/herloct/slim-newrelic.svg?branch=master)](https://travis-ci.org/herloct/slim-newrelic)
[![Test Coverage](https://codeclimate.com/github/herloct/slim-newrelic/badges/coverage.svg)](https://codeclimate.com/github/herloct/slim-newrelic/coverage)
[![Code Climate](https://codeclimate.com/github/herloct/slim-newrelic/badges/gpa.svg)](https://codeclimate.com/github/herloct/slim-newrelic)
[![license](https://img.shields.io/github/license/herloct/slim-newrelic.svg)](herloct/slim-newrelic)

This library provides New Relic instrumentation for the Slim framework.
When installed this library will ensure that your transactions are properly
named and that your exceptions are properly logged in New Relic.

Using @nordsoftware's [nordsoftware/lumen-newrelic](https://github.com/nordsoftware/lumen-newrelic) as 
reference.

## License

See [LICENSE](LICENSE)

## Requirements

- PHP 5.6 or newer
- [Composer](http://getcomposer.org)
- [Slim](https://www.slimframework.com/) 3.7 or newer

## Usage

### Installation

Run the following command to install the package through Composer:

```sh
composer require herloct/slim-newrelic
```

### Bootstrapping

Example `index.php`.

```php
$config = [
    'settings' => [
        'displayErrorDetails' => true,

        // These two are needed so new relic agent could work
        'addContentLengthHeader' => false,
        'determineRouteBeforeAppMiddleware' => true
    ],

    'errorHandler' => function ($c) {
        $agent = $c->get(\SobanVuex\NewRelic\Agent::class);
        $errorHandler = new \Slim\Handlers\Error($c->get('settings')['displayErrorDetails']);

        return new \Herloct\Slim\Handlers\NewRelicError($agent, $errorHandler);
    },

    \SobanVuex\NewRelic\Agent::class => function ($c) {
        $agent = new \SobanVuex\NewRelic\Agent(
            'Your Application Name',
            'YOUR_NEW_RELIC_LICENSE_KEY'
        );

        return $agent;
    },

    \Herloct\Slim\NewRelicTransactionMiddleware::class => function ($c) {
        $agent = $c->get(\SobanVuex\NewRelic\Agent::class);

        return new \Herloct\Slim\NewRelicTransactionMiddleware($agent);
    }
];

$app = new \Slim\App($config);
$app->add(\Herloct\Slim\NewRelicTransactionMiddleware::class);

$app->get('/hello/{name}', function ($request, $response, $args) {
    return $response->write("Hello " . $args['name']);
})->setName('say_hello');

$app->run();
```

## Customizing transaction names

By default the transaction name will use the route's name. If that fails, it will use the
route's pattern.

If this doesn't meet your requirements, extend the `Herloct\Slim\NewRelicTransactionMiddleware` class and override the
`getTransactionName()` method, then register that middleware class instead.

## Running tests

Clone the project and install its dependencies by running:

```sh
composer install
```

Run the following command to run the test suite:

```sh
vendor/bin/phpunit
```
