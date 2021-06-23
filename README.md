PHP class map generator
=======================

![Integrity check](https://github.com/baraja-core/class-map-generator/workflows/Integrity%20check/badge.svg)

Find all classes in specific directory.

📦 Installation
---------------

It's best to use [Composer](https://getcomposer.org) for installation, and you can also find the package on
[Packagist](https://packagist.org/packages/baraja-core/class-map-generator) and
[GitHub](https://github.com/baraja-core/class-map-generator).

To install, simply use the command:

```shell
$ composer require baraja-core/class-map-generator
```

You can use the package manually by creating an instance of the internal classes, or register a DIC extension to link the services directly to the Nette Framework.

How to use
----------

```php
$map = \Baraja\ClassMapGenerator\ClassMapGenerator::createMap(__DIR__);
```

📄 License
-----------

`baraja-core/class-map-generator` is licensed under the MIT license. See the [LICENSE](https://github.com/baraja-core/template/blob/master/LICENSE) file for more details.
