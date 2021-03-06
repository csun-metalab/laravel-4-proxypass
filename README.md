# Laravel 4 Proxy Pass

**NOTE**: If you are looking for the Proxy Pass package for Laravel 5.0 and up, you'll want to visit https://github.com/csun-metalab/laravel-proxypass instead.

Composer package for Laravel 4 that resolves the correct absolute URLs when behind a proxy

This package is built for version 4.2 of Laravel. It modifies the functionality
of the `url()`, `asset()`, and other helper methods.

To install from Composer, use the following command:

```
composer require csun-metalab/laravel-4-proxypass
```

## Installation

First, add the following lines to your .env.php file to leverage the proxy attributes:

```
'PROXY_ACTIVE' => true,
'PROXY_PATH_HEADER' => 'HTTP_X_FORWARDED_PATH',
```

You may also add the following optional lines to your .env.php file to leverage the ability to FORCE the URL and schema without having to pass through a load balancer or proxy:

```
# http or https: PUBLIC_SCHEMA_OVERRIDE=https
'PUBLIC_SCHEMA_OVERRIDE' => '',

# Example: PUBLIC_URL_OVERRIDE=http://some-other-domain.example.com/some-other-directory
'PUBLIC_URL_OVERRIDE' => '',
```

Next, add the service provider to your `providers` array in Laravel as follows:

```
'providers' => [
   //...

   CSUNMetaLab\ProxyPass4\Providers\ProxyPassServiceProvider::class,

   // You can also use the following depending on Laravel convention:
   // 'CSUNMetaLab\ProxyPass4\Providers\ProxyPassServiceProvider',

   //...
],
```

Finally, run the following Artisan command to publish the configuration:

```
php artisan config:publish csun-metalab/laravel-4-proxypass
```

## Environment Variables

The two environment variables you added to your .env.php file are the following:

### PROXY_ACTIVE

Set this to `true` to enable the proxying functionality or `false` to disable it.

### PROXY_PATH_HEADER

This is the PHP-interpreted value of the request header sent from your proxy. The
default is `HTTP_X_FORWARDED_PATH` (the computed value of `X-Forwarded-Path`)

## Trusted Proxies

This package also has the ability to allow only certain proxy servers to modify
the necessary values in order to set the proper absolute URL.

By default, all proxy servers are allowed to modify the values; if, however,
the following value is set in your .env.php file then you can create a whitelist of proxies:

### TRUSTED_PROXIES

This is a comma-delimited list of hostnames/IP addresses that are allowed to
perform proxying functions.

```
'TRUSTED_PROXIES' => '192.168.1.10,www.example.com,192.168.3.12',
```

The above example would allow the following three proxy servers to provide
proxying functionality:

* 192.168.1.10 (would come from the `REMOTE_ADDR` value in PHP)
* www.example.com (would come from the `X-Forwarded-Server` header from the web server)
* 192.168.3.12 (would come from the `REMOTE_ADDR` value in PHP)

## Usage Example

Let's say you have an application hosted at `http://laravel.example.com` but that is
not the location you want to show to the world. Instead, you want to show a URL of
`http://www.example.com/laravel` so you place your Laravel application behind a proxy.

However, you notice that while the front page loads properly, none of the URLs you
have written with the `url()`, `asset()`, or other helpers work with that URL and instead
continue writing `http://laravel.example.com` as their base path.

You can configure your proxy to add a request header along with your `ProxyPass` and
`ProxyPassReverse` directives in Apache (ensure you have `mod_headers` enabled and
`mod_proxy` enabled as well):

```
ProxyPass        /laravel http://laravel.example.com
ProxyPassReverse /laravel http://laravel.example.com

<Location /laravel>
RequestHeader set X-Forwarded-Path "http://www.example.com/laravel"
</Location>
```

Now all of your URLs using the `url()`, `asset()`, and other helpers will be written
correctly!
