---
layout: post
title: "Extractor, a PHP Library for compressed packages"
date: 2014-09-13
comments: true
author: Marc Morera
categories:
    - php
    - extractor
    - compressed
---
Extractor is a simple php library for extracting all files from compressed
packages. Available formats are

* zip
* Rar
* Phar
* Tar
* Gz
* Bz2

You can find the source in the
[Github repository](https://github.com/mmoreram/extractor).

Extractor uses the Finder Symfony component, so the result of extracting all
compressed files given a package is nothing more than a Finder instance ready
to be iterated and configured.

``` php
<?php

use Symfony\Component\Finder\Finder;
use Mmoreram\Extractor\Filesystem\TemporaryDirectory;
use Mmoreram\Extractor\Resolver\ExtensionResolver;
use Mmoreram\Extractor\Extractor;

$temporaryDirectory = new TemporaryDirectory();
$extensionResolver = new ExtensionResolver();
$extractor = new Extractor(
    $temporaryDirectory,
    $extensionResolver
);

/**
 * @var Finder $files
 */
$files = $extractor->extractFromFile('/tmp/myfile.rar');
foreach ($files as $file) {

    echo $file->getRealpath() . PHP_EOL;
}
```

You can use the temporary folder of your Filesystem using a `TemporaryDirectory`
instance or you can use a `SpecificDirectory` instance if you want to specify
where all files should be extracted.

``` php
use Mmoreram\Extractor\Filesystem\SpecificDirectory;
use Mmoreram\Extractor\Resolver\ExtensionResolver;
use Mmoreram\Extractor\Extractor;

$specificDirectory = new SpecificDirectory('/my/specific/path');
$extensionResolver = new ExtensionResolver();
$extractor = new Extractor(
    $specificDirectory,
    $extensionResolver
);
```

You can also work with remote files.

``` php
use Symfony\Component\Finder\Finder;
use Mmoreram\Extractor\Filesystem\TemporaryDirectory;
use Mmoreram\Extractor\Resolver\ExtensionResolver;
use Mmoreram\Extractor\Extractor;

$specificDirectory = new TemporaryDirectory();
$extensionResolver = new ExtensionResolver();
$extractor = new Extractor(
    $specificDirectory,
    $extensionResolver
);

/**
 * @var Finder $files
 */
$files = $extractor
    ->extractFromFile('http://host.com/my-compressed-file.zip');
```