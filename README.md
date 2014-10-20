# hanneskod/classtools

[![Packagist Version](https://img.shields.io/packagist/v/hanneskod/classtools.svg?style=flat-square)](https://packagist.org/packages/hanneskod/classtools)
[![Build Status](https://img.shields.io/travis/hanneskod/classtools/master.svg?style=flat-square)](https://travis-ci.org/hanneskod/classtools)
[![Quality Score](https://img.shields.io/scrutinizer/g/hanneskod/classtools.svg?style=flat-square)](https://scrutinizer-ci.com/g/hanneskod/classtools)
[![Dependency Status](https://img.shields.io/gemnasium/hanneskod/classtools.svg?style=flat-square)](https://gemnasium.com/hanneskod/classtools)

Find, extract and process classes from file system.

> Install using **[composer](http://getcomposer.org/)**. Exists as
> **[hanneskod/classtools](https://packagist.org/packages/hanneskod/classtools)**
> in the **[packagist](https://packagist.org/)** repository.


## Iterator

[ClassIterator](src/Iterator/ClassIterator.php) consumes a [symfony
finder](http://symfony.com/doc/current/components/finder.html) and scans files
for php classes, interfaces and traits.

### Access the class map

`getClassMap()` returns a map of class names to
[SplFileInfo](http://api.symfony.com/2.5/Symfony/Component/Finder/SplFileInfo.html)
objects.

```php
$finder = new Finder;
$iter = new ClassIterator($finder->in('src'));

// Print the file names of classes, interfaces and traits in 'src'
foreach ($iter->getClassMap() as $classname => $splFileInfo) {
    echo $classname.': '.$splFileInfo->getRealPath();
}
```

### Iterate over ReflectionClass objects

ClassIterator is also a
[Traversable](http://php.net/manual/en/class.traversable.php), that on iteration
yields class names as keys and
[ReflectionClass](http://php.net/manual/en/class.reflectionclass.php) objects as
values.

Note that to use reflection the classes found in filesystem must be
included in the environment. Enable autoloading to dynamically load classes from
a ClassIterator.

```php
$finder = new Finder();
$iter = new ClassIterator($finder->in('src'));

// Enable reflection by autoloading found classes
$iter->enableAutoloading();

// Print all classes, interfaces and traits in 'src'
foreach ($iter as $class) {
    echo $class->getName();
}
```

### Filter based on class properties

[ClassIterator](src/Iterator/ClassIterator.php) is filterable and filters are
chainable.

```php
$finder = new Finder();
$iter = new ClassIterator($finder->in('src'));
$iter->enableAutoloading();

// Print all Filter types (including the interface itself)
foreach ($iter->type('Iterator\Filter') as $class) {
    echo $class->getName();
}

// Print definitions in the Iterator namespace whose name contains 'Class'
foreach ($iter->inNamespace('Iterator')->name('/Class/') as $class) {
    echo $class->getName();
}

// Print implementations of the Filter interface
foreach ($iter->type('Iterator\Filter')->where('isInstantiable') as $class) {
    echo $class->getName();
}
```

### Negate filters

Filters can also be negated by wrapping them in `not()` method calls.

```php
$finder = new Finder();
$iter = new ClassIterator($finder->in('src'));
$iter->enableAutoloading();

// Print all classes, interfaces and traits NOT instantiable
foreach ($iter->not($iter->where('isInstantiable')) as $class) {
    echo $class->getName();
}
```

### Transforming classes

Found class, interface and trait definitions can be transformed and written to a
single file.

```php
$finder = new Finder();
$iter = new ClassIterator($finder->in('src'));
$iter->enableAutoloading();

// Print all found definitions in one snippet
echo $iter->minimize();

// The same can be done using
echo $iter->transform(new MinimizingWriter);
```

## Transformer examples

### Wrap code in namespace

```php
$reader = new Reader("<?php class Bar {}");
$writer = new Writer;
$writer->apply(new Action\NamespaceWrapper('Foo'));

// Outputs class Bar wrapped in namespace Foo
echo $writer->write($reader->read('Bar'));
```

### Strip statements

```php
$reader = new Reader("<?php require 'Foo.php'; echo 'bar';");
$writer = new Writer;
$writer->apply(new Action\NodeStripper('Expr_Include'));

// Outputs the echo statement
echo $writer->write($reader->readAll());
```
