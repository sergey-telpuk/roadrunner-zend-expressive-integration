# [roadrunner](https://github.com/spiral/roadrunner)-zend-expressive-integration

The entrypoint file can look like bellow: 
```php

declare(strict_types=1);

ini_set('display_errors', 'stderr');

chdir(dirname(__DIR__));
require 'vendor/autoload.php';

$relay = new Spiral\Goridge\StreamRelay(STDIN, STDOUT);
$psr7 = new Spiral\RoadRunner\PSR7Client(new Spiral\RoadRunner\Worker($relay));

$app = (function () {
    $container = require 'config/container.php';
    $app = $container->get(\Zend\Expressive\Application::class);
    $factory = $container->get(\Zend\Expressive\MiddlewareFactory::class);
    (require 'config/pipeline.php')($app, $factory, $container);
    (require 'config/routes.php')($app, $factory, $container);

    return $app;
})();


while ($req = $psr7->acceptRequest()) {
    try {
        $response = $app->handle($req);
        $psr7->respond($response);
    } catch (\Throwable $e) {
        $psr7->getWorker()->error((string)$e);
    }
}
```
