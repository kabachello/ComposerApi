# ComposerAPI
A wrapper for [Composer](http://getcomposer.org) to call it's commands from inside your code using a simple object oriented API: turns <code>php composer.phar require monolog/monolog</code> into <code>$composer->require(array('monolog/monolog:*'));</code>.

## Installation
As always, the easiest (and the recommended) way to install is using composer:
```
composer require composer/composer:1.2 kabachello/composerapi:*
```

Note the explicit requirement for <code>composer/composer</code> in a fixed version: this is important to prevent automatic updates of this dependency when running the <code>update</code> command programmatically - see "Known limitations" section below for details.

There is no simple way to install composer an the API without using composer itself. Theoretically you could include ComposerAPI.php in your code manually, but you would need to make sure an installation of "composer/composer" is available under the namespace "\Composer". The trouble is, however, that composer has lot's of dependencies itself, so you will probably end up needing the packaged version (composer.phar) anyway. If so, use the simple composer-install above.

## Quick start
Here is an example, that adds the monolog library to an existing composer.json manifest and installes it with all dependencies:
```php
<?php
// Instantiate ComposerAPI
$composer = new \kabachello\ComposerAPI\ComposerAPI("path_to_the_folder_with_your_composer_json");
// Run the require command for monolog/monolog (latest version). The default output will be symfony's StreamOutput
$output = $composer->require(array('monolog/monolog:*'));
// Fetch the stream
$stream  = $output->getStream();
// Rewind it to get the full contents
rewind($stream);
// Print everything composer had written to the console
echo(stream_get_contents($stream));
?>
```

## Supported commands
All commands take an optional $output argument that must implement the <code>OutputInterface</code> of symfony console. If this argument is not passed, ComposerAPI will use <code>StreamOutput</code> by default.

- *install(array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->install()</code>. This will probably not be used very often because the API mostly makes sense for managing existing installations and not for installing "from scratch".
- *update(array $package_names = null, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->update()</code> or <code>$composer->update(array('monolog/monolog', 'kabachello/composerapi'))</code>
- *require(array $package_names, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->require(array('monolog/monolog:~1.16', 'slim/slim'))</code>
- *remove(array $package_names, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->remove(array('monolog/monolog'))</code>
- *search(array $search_terms, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->search(array('composerapi'))</code>
- *show(array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->show()</code> or <code>$composer->show(array('--latest'))</code>
- *outdated(array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->outdated()</code>
- *suggests(array $package_names = null, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->suggests()</code> or <code>$composer->suggests(array('symfony/event-dispatcher'), array('--tree'))</code>
- *depends($package_name, $version = null, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->depends('doctrine/lexer', array('--tree'))</code>
- *prohibits($package_name, $version = null, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->prohibits('symfony/symfony', '3.1', array('--tree'))</code>
- *validate(array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->validate()</code>
- *config($setting_key, array $setting_values, array $options = null, OutputInterface $output = null)*: 
e.g. <code>$composer->config('repositories.foo', array('vcs', 'https://github.com/foo/bar'))</code>

## Known limitations

### Self-update for composer not working properly
Since composer itself is a dependency of ComposerAPI, calling <code>ComposerAPI->update()</code> will also lead to an attempt to update composer. This won't work in most cases because the dependencies of composer will get updated one-by-one leading to inconsistencies.
*Solution*: Pin composer to a concrete version in your root composer.json to block updating it. If you want to update composer manuall, use the packaged version (composer.phar) just like when installing ComposerAPI.

### Resource consumption
Composer often needs a lot of memory and a lot of time too. Running commands like <code>update</code> will often take longer than PHP's <code>max_execution_time</code> or ecxeed the <code>memory_limit</code>. ComposerAPI tries to increase these limits at runtime, however this will not work if PHP runs in safe mode. Unfortunately, I do not see a simple solution for this issue. Feel free to suggest one!