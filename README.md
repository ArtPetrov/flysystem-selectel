# Flysystem Adapter for Selectel Cloud Storage

## Requirements
This package requires PHP 7.3.

## Installation

You can install the package via composer:

``` bash
$ composer require artPetrov/flysystem-selectel
```

## Upgrade

### From 1.0* to 1.1.0
New setting `container_url` was added. You can set your container's custom domain here (for example, `https://static.example.org`) and this option will be used when retrieving fully qualified URLs to files and directories.

## Usage

``` php
use ArtPetrov\Flysystem\Selectel\SelectelAdapter;
use ArtPetrov\Selectel\CloudStorage\Api\ApiClient;
use ArtPetrov\Selectel\CloudStorage\CloudStorage;
use League\Flysystem\Filesystem;

$api = new ApiClient('selectel-username', 'selectel-password');
$storage = new CloudStorage($api);
$container = $storage->getContainer('container-name');

$adapter = new SelectelAdapter($container);
$filesystem = new Filesystem($adapter);
```

## Symfony 4.3 Integration
**service.yaml**
```yaml
services:

  openstack.selectel.client:
    class: ArtPetrov\Selectel\CloudStorage\Api\ApiClient
    arguments: ['%env(SELECTEL_USERNAME)%','%env(SELECTEL_PASSWORD)%','%env(SELECTEL_AUTH_PATH)%']

  ArtPetrov\Selectel\CloudStorage\CloudStorage:
    arguments: ['@openstack.selectel.client']

  openstack.selectel.storage_container:
    class: ArtPetrov\Selectel\CloudStorage\CloudStorage
    factory: ['@ArtPetrov\Selectel\CloudStorage\CloudStorage','getContainer']
    arguments: ['%env(SELECTEL_CONTAINER_NAME)%']

  ArtPetrov\Flysystem\Selectel\SelectelAdapter:
    arguments:
      $container: '@openstack.selectel.storage_container'

```
**packages/oneup_flysystem.yaml**
```yaml
oneup_flysystem:
  adapters:
    public_uploads_adapter:
      custom:
        service: ArtPetrov\Flysystem\Selectel\SelectelAdapter

  filesystems:
    public_uploads_filesystem:
      adapter: public_uploads_adapter


```


## Laravel Integration

You can use this adapter with Laravel's [Storage System](https://laravel.com/docs/5.5/filesystem).

If you're running Laravel 5.5+ this package will auto-added to your providers list via auto-discovery feature (requires version 1.2+ of this package).

### Laravel <= 5.4
Add `ArtPetrov\Flysystem\Selectel\SelectelServiceProvider::class` to your providers list in `config/app.php`

```php
/*
 * Package Service Providers...
 */
ArtPetrov\Flysystem\Selectel\SelectelServiceProvider::class,
```

### All Laravel versions
Add `selectel` disk to `config/filesystems.php` configuration file (`disks` section):

```php
'selectel' => [
    'driver' => 'selectel',
    'username' => 'selectel-username',
    'password' => 'selectel-password',
    'container' => 'selectel-container',
    'container_url' => 'https://static.example.org',
]
```

`container_url` setting (new in version **1.1.0**) allows you to override default Selectel's CDN domain (if you have custom domain attached). You may omit this setting if you're using default domain, file URLs will look like `http://XXX.selcdn.ru/container_name/path/to/file.txt`, where `XXX` - your unique subdomain (`X-Storage-Url` header value).

Now you can use Selectel disk as

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('selectel')->put('file.txt', 'Hello world');
```

Also you may want to set `selectel` as default disk to ommit `disk('selectel')` calls and use storage just as `Storage::put('file.txt', 'Hello world')`.

For more info please refer to Laravel's [Storage System](https://laravel.com/docs/5.4/filesystem) documentation.

## Unsupported methods
Due to the implementation of the Selectel API some methods are missing or may not function as expected.

### Visibility management
Selectel provides visibility support only for Containers, but not for files. The change of visibility for the entire container instead of a single file/directory may be confusing for adapter users. Adapter will throw `LogicException` on `getVisibility`/`setVisbility` calls.

### Directories management
Currently Selectel Adapter can display and delete only those directories that were created via `createDir` method. Dynamic directories (those that were created via `write`/`writeStream` methods) can not be deleted or listed as directory.

```php
$fs = new Filesystem($adapter);

$fs->createDir('images'); // The 'images' directory can be deleted and will be listed as 'dir' in the results of `$fs->listContents()`.

$fs->write('documents/hello.txt'); // The 'documents' directory can not be deleted and won't be listed in the results of `$fs->listContents()`.
```

## Note on Closing Streams
Selectel Adapter leaves the streams **open** after consuming them. Make sure that you've closed all streams that you opened.

## Change log
Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Testing
``` bash
$ vendor/bin/phpunit
```

## Contributing
Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## License
The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

