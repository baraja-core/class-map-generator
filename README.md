# PHP Class Map Generator

![Integrity check](https://github.com/baraja-core/class-map-generator/workflows/Integrity%20check/badge.svg)

A lightweight PHP library for scanning directories and generating a complete map of all PHP classes, interfaces, and traits with their corresponding file paths.

## 🎯 Key Features

- **Recursive directory scanning** - Automatically traverses all subdirectories to find PHP files
- **Token-based parsing** - Uses PHP's native `token_get_all()` for reliable class detection
- **Full namespace support** - Correctly resolves fully qualified class names with namespaces
- **Supports all class types** - Detects classes, interfaces, and traits
- **Memory efficient** - Includes garbage collection optimization for large codebases
- **Zero dependencies** - Pure PHP implementation with no external requirements
- **PHP 8.0+ compatible** - Built for modern PHP applications

## 🏗️ Architecture

The library consists of a single, focused class that handles all scanning and parsing logic:

```
┌─────────────────────────────────────────────────────────────────┐
│                     ClassMapGenerator                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌──────────────────┐    ┌───────────┐  │
│  │   createMap()   │───►│  Directory Scan  │───►│  Output   │  │
│  │   (public)      │    │  (recursive)     │    │  Map      │  │
│  └─────────────────┘    └────────┬─────────┘    └───────────┘  │
│                                  │                              │
│                                  ▼                              │
│                         ┌──────────────────┐                    │
│                         │  findClasses()   │                    │
│                         │  (private)       │                    │
│                         └────────┬─────────┘                    │
│                                  │                              │
│                                  ▼                              │
│                         ┌──────────────────┐                    │
│                         │  Token Parser    │                    │
│                         │  - T_NAMESPACE   │                    │
│                         │  - T_CLASS       │                    │
│                         │  - T_INTERFACE   │                    │
│                         │  - T_TRAIT       │                    │
│                         └──────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### How It Works

1. **Directory Iteration**: The `createMap()` method uses `RecursiveDirectoryIterator` to traverse all files in the specified directory and its subdirectories.

2. **File Filtering**: Only files with the `.php` extension are processed, skipping all other file types.

3. **Token Parsing**: Each PHP file is read and parsed using PHP's `token_get_all()` function, which breaks down the source code into tokens.

4. **Namespace Detection**: The parser tracks `T_NAMESPACE` tokens to build the current namespace context for class resolution.

5. **Class Detection**: When `T_CLASS`, `T_INTERFACE`, or `T_TRAIT` tokens are encountered, the parser extracts the class name and combines it with the current namespace.

6. **::class Constant Handling**: The parser intelligently skips `::class` constant usage (e.g., `SomeClass::class`) to avoid false positives.

7. **Memory Management**: After processing each file, `gc_mem_caches()` is called to address a known PHP memory issue with `token_get_all()`.

## 📦 Installation

It's best to use [Composer](https://getcomposer.org) for installation, and you can also find the package on
[Packagist](https://packagist.org/packages/baraja-core/class-map-generator) and
[GitHub](https://github.com/baraja-core/class-map-generator).

To install, simply use the command:

```shell
$ composer require baraja-core/class-map-generator
```

You can use the package manually by creating an instance of the internal classes, or register a DIC extension to link the services directly to the Nette Framework.

### Requirements

- PHP 8.0 or higher
- No external dependencies

## 🚀 Usage

### Basic Usage

Generate a class map for a directory:

```php
use Baraja\ClassMapGenerator\ClassMapGenerator;

$map = ClassMapGenerator::createMap(__DIR__ . '/src');
```

### Return Value

The `createMap()` method returns an associative array where:
- **Keys** are fully qualified class names (including namespace)
- **Values** are absolute file paths

```php
// Example output:
[
    'App\Controllers\HomeController' => '/var/www/project/src/Controllers/HomeController.php',
    'App\Models\User' => '/var/www/project/src/Models/User.php',
    'App\Services\AuthService' => '/var/www/project/src/Services/AuthService.php',
]
```

### Practical Examples

#### Finding a Specific Class File

```php
$map = ClassMapGenerator::createMap(__DIR__ . '/src');

if (isset($map['App\Services\PaymentService'])) {
    $filePath = $map['App\Services\PaymentService'];
    echo "PaymentService is located at: {$filePath}";
}
```

#### Listing All Classes in a Namespace

```php
$map = ClassMapGenerator::createMap(__DIR__ . '/src');

$controllers = array_filter(
    $map,
    fn($class) => str_starts_with($class, 'App\\Controllers\\'),
    ARRAY_FILTER_USE_KEY
);

foreach ($controllers as $className => $filePath) {
    echo "{$className}\n";
}
```

#### Building an Autoloader

```php
$map = ClassMapGenerator::createMap(__DIR__ . '/src');

spl_autoload_register(function (string $class) use ($map): void {
    if (isset($map[$class])) {
        require_once $map[$class];
    }
});
```

#### Caching the Class Map

For performance optimization in production environments:

```php
$cacheFile = __DIR__ . '/cache/classmap.php';

if (file_exists($cacheFile)) {
    $map = require $cacheFile;
} else {
    $map = ClassMapGenerator::createMap(__DIR__ . '/src');

    file_put_contents(
        $cacheFile,
        '<?php return ' . var_export($map, true) . ';'
    );
}
```

#### Scanning Multiple Directories

```php
$directories = [
    __DIR__ . '/src',
    __DIR__ . '/lib',
    __DIR__ . '/modules',
];

$fullMap = [];

foreach ($directories as $dir) {
    $fullMap = array_merge($fullMap, ClassMapGenerator::createMap($dir));
}
```

## 🔍 What Gets Detected

The library detects the following PHP constructs:

| Construct | Detected | Example |
|-----------|----------|---------|
| Classes | Yes | `class MyClass {}` |
| Abstract classes | Yes | `abstract class BaseController {}` |
| Final classes | Yes | `final class ImmutableValue {}` |
| Interfaces | Yes | `interface PaymentGateway {}` |
| Traits | Yes | `trait Loggable {}` |
| Anonymous classes | No | `new class {}` |
| Enums | No | `enum Status {}` |

## ⚠️ Limitations

- **Anonymous classes** are not detected (they have no name to map)
- **PHP Enums** (PHP 8.1+) are not detected by the current implementation
- **Files must be syntactically valid PHP** - parsing errors may cause issues
- **Multiple classes per file** are fully supported, but only the class matching the filename is typically recommended

## 🔧 Technical Notes

### Memory Management

The library calls `gc_mem_caches()` after processing each file to address a [known PHP memory issue](https://bugs.php.net/70098) where `token_get_all()` doesn't release memory properly. This ensures stable memory usage when scanning large codebases.

### Path Resolution

The library attempts to get the real path of each file using `getRealPath()`. If this fails (e.g., on some edge cases with symlinks), it falls back to `getPathname()`.

### Inspiration

This library is based on the [Symfony ClassLoader component](https://github.com/symfony/class-loader/blob/master/ClassMapGenerator.php), adapted and modernized for PHP 8.0+ with strict typing and simplified API.

## 👨‍💻 Author

**Jan Barášek** - [https://baraja.cz](https://baraja.cz)

## 📄 License

`baraja-core/class-map-generator` is licensed under the MIT license. See the [LICENSE](https://github.com/baraja-core/class-map-generator/blob/master/LICENSE) file for more details.
