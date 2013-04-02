﻿WebLoader [![Build Status](https://secure.travis-ci.org/janmarek/WebLoader.png?branch=master)](http://travis-ci.org/janmarek/WebLoader)
=======================

Component for CSS and JS files loading

Author: Jan Marek
Licence: MIT

Example
-------

Control factory in Nette presenter:

```php
<?php

protected function createComponentCss()
{
	$files = new WebLoader\FileCollection(WWW_DIR . '/css');
	$files->addFiles(array(
		'style.css',
		WWW_DIR . '/colorbox/colorbox.css',
	));

	$compiler = WebLoader\Compiler::createCssCompiler($files, WWW_DIR . '/temp');

	$compiler->addFilter(new WebLoader\Filter\VariablesFilter(array('foo' => 'bar'));
	$compiler->addFilter(function ($code) {
		return cssmin::minify($code, "remove-last-semicolon");
	});

	$control = new WebLoader\Nette\CssLoader($compiler, '/webtemp');
	$control->setMedia('screen');

	return $control;
}
```

Template:

```html
{control css}
```

Example with Nette Framework extension used
-------------------------------------------

Extension is registered in bootstrap.php:

```php
$webloaderExtension = new \WebLoader\Nette\Extension();
$webloaderExtension->install($configurator);
```

Configuration in config.neon:

```html
services:
	wlCssFilter: WebLoader\Filter\CssUrlsFilter(%wwwDir%)

webloader:
	css:
		default:
			files:
				- style.css
				- {files: ["*.css", "*.less"], from: %appDir%/presenters} # Nette\Utils\Finder support
			filters:
				- @wlCssFilter
	js:
		default:
			remoteFiles:
				- http://ajax.googleapis.com/ajax/libs/jquery/1.7/jquery.min.js
				- http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.16/jquery-ui.min.js
			files:
				- %appDir%/../libs/nette/nette/client-side/forms/netteForms.js
				- web.js
```

BasePresenter.php:

```php
public function createComponentCss()
{
	return new \WebLoader\Nette\CssLoader(
		$this->context->webloader->cssDefaultCompiler, // service generated by extension
		$this->template->basePath . '/webtemp'
	);
}

public function createComponentJs()
{
	return new \WebLoader\Nette\JavaScriptLoader(
		$this->context->webloader->jsDefaultCompiler, // service generated by extension
		$this->template->basePath . '/webtemp'
	);
}
```