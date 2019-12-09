# Awesome Symfony

A curated list of useful Symfony snippets.

Contributions are highly encouraged and very welcome :)

## Table of Contents

- [Configuration](#configuration)
    - [Assets](#assets)
    - [Directories, Paths](#directories-paths)
        - [Get the Project Root Directory](#get-the-project-root-directory)
    - [Email Errors](#email-errors)
        - [Email Logs Related to 5xx Errors (action_level: critical)](#email-logs-related-to-5xx-errors-action_level-critical)
        - [Email Logs Related to 4xx Errors (action_level: error)](#email-logs-related-to-4xx-errors-action_level-error)
        - [Do Not Email Logs for 404 Errors (excluded_404)](#do-not-email-logs-for-404-errors-excluded_404)
    - [Import](#import)
        - [Import Mixed Configuration Files](#import-mixed-configuration-files)
        - [Import All Resources From a Directory](#import-all-resources-from-a-directory)
        - [Import Configuration Files Using Glob Patterns](#import-configuration-files-using-glob-patterns)
    - [Log](#log)
        - [Enable the Monolog processor PsrLogMessageProcessor](#enable-the-monolog-processor-psrlogmessageprocessor)
        - [Hide Event Logs](#hide-event-logs)
        - [Organizing Log Files Using Channels (Log Messages to Different Files)](#organizing-log-files-using-channels-log-messages-to-different-files))
    - [Security](#security)
        - [Impersonating Users](#impersonating-users)
    - [Session](#session)
    - [Profiler](#profiler)
- [Console](#console)
    - [Parallel Asset Dump](#parallel-asset-dump)
- [Controller](#controller)
    - [Cookie](#cookie)
    - [Directories, Paths, and Filesystem](#directories-paths-and-filesystem)
    - [Download](#download)
    - [JSON](#json)
    - [Flash Messages](#flash-messages)
    - [Form](#form)
    - [Redirect](#redirect)
    - [Request](#request)
    - [Response](#response)
    - [Routing](#routing)
        - [External URLs](#external-urls)
        - [External URLs - Using a Key to Reference a URL](#external-urls-using-a-key-to-reference-a-url)
    - [Service](#service)
        - [Retrieve a Service](#retrieve-a-service)
    - [YAML](#yaml)
        - [Parse YAML File](#parse-yaml-file)
- [Environment Variables](#environment-variables)
    - [Custom Loader for Environment Variables](#custom-loader-for-environment-variables)
- [Routing](#routing)
    - [Generate Absolute URL](#generate-absolute-url)
    - [Trailing Slash with an Optional Parameter](#trailing-slash-with-an-optional-parameter)
- [Twig](#twig)
    - [Absolute URLs](#absolute-urls)
    - [Assets Versioning](#assets-versioning)
    - [Get the Authenticated Username](#get-the-authenticated-username)
    - [Get the Base URL](#get-the-base-url)
    - [Localized Date String](#localized-date-string)
    - [Inject All GET Parameters in a Route](#inject-all-get-parameters-in-a-route)
    - [Make the `form_rest()` and `form_end()` not Display a Specific Field](#make-the-form_rest-and-form_end-not-display-a-specific-field)
    - [Override the 404 Error Template](#override-the-404-error-template)
    - [Render a Controller Asynchronously](#render-a-controller-asynchronously)
    - [Render Just the Close Form HTML Tag](#render-just-the-close-form-html-tag)
    - [Render a Template without a Specific Controller for a Static Page](#render-a-template-without-a-specific-controller-for-a-static-page)
    - [Using Localized Data (Date, Currency, Number, ...)](#using-localized-data-date-currency-number-)

## Configuration

### Assets

#### Assets Base URL
`Symfony 2.6`
```yaml
# app/config/config.yml
framework:
    templating:
        assets_base_urls:
            http: ['http://cdn.domain.com']
            ssl:  ['https://secure.domain.com']
        packages:
            # ...
```

`Symfony 2.7+`
```yaml
# app/config/config.yml
framework:
    assets:
        base_path: ~
        base_urls: ['http://cdn.domain.com', 'https://secure.domain.com']
```
[[1]](http://symfony.com/blog/new-in-symfony-2-7-the-new-asset-component)

#### Assets Base URL (Protocol-Relative)
```yaml
# app/config/config.yml 
framework: 
    templating: 
        assets_base_urls: '//static.domain.com/images'
```

#### Assets Versioning
`Symfony 2.6`
```yaml
# app/config/config.yml
framework:
    templating:
        assets_version: 'v5'
        assets_version_format: '%%s?version=%%s'
```

`Symfony 2.7+`
```yaml
# app/config/config.yml
framework:
    assets:
        version: 'v5'
        version_format: '%%s?version=%%s'
```
[[1]](http://symfony.com/blog/new-in-symfony-2-7-the-new-asset-component)

#### Named Assets
```yaml
# app/config/config.yml 
assetic:
    assets:
        bootstrap_js:
            inputs:
                - '@AppBundle/Resources/public/js/jquery.js'
                - '@AppBundle/Resources/public/js/bootstrap.js'
```

Using in Twig templates:
```twig
{% javascripts
    '@bootstrap_js'
    '@AppBundle/Resources/public/js/*' %}
        <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```

#### Context-Aware CDNs
```yaml
# app/config/config.yml
framework:
    assets:
        base_urls:
            - 'http://static1.domain.com/images/'
            - 'https://static2.domain.com/images/'
```

Using in Twig templates:
```twig
{{ asset('logo.png') }}
{# in a regular page: http://static1.domain.com/images/logo.png #}
{# in a secure page:  https://static2.domain.com/images/logo.png #}
```
[[1]](http://symfony.com/blog/new-in-symfony-2-7-the-new-asset-component#context-aware-cdns)

#### Packages (Different Base URLs)

To specify different base URLs for assets, group them into packages:
```yaml
# app/config/config.yml
framework:
    # ...
    assets:
        packages:
            avatars:
                base_urls: 'http://static_cdn.domain.com/avatars'
```

Using the `avatars` package in a Twig template:
```twig
<img src="{{ asset('...', 'avatars') }}" />
```

### Directories, Paths

#### Get the Project Root Directory

Use the config parameter `%kernel.root_dir%/../`:
```yaml
some_service:
    class: \path\to\class
    arguments: [%kernel.root_dir%/../]
```

`Symfony 2`
In a Controller:

`$projectRoot = $this->container->getParameter('kernel.root_dir');`

`Symfony 3.3`

In a Controller:

`$projectRoot = $this->get('kernel')->getProjectDir();`

`Symfony 4+`

Using autowiring (argument binding):
```yaml
# config/services.yaml
services:
    _defaults:
        bind:
            string $projectDir: '%kernel.project_dir%'
```

Then in your class:
```php
class YourClass
{
    private $projectDir;

    public function __construct(string $projectDir)
    {
        $this->$projectDir = $projectDir;
    }

    // ...
```

### Email Errors

#### Email Logs Related to 5xx Errors (`action_level: critical`)
```yaml
# app/config/config_prod.yml 
monolog:
    handlers:
        mail:
            type: fingers_crossed
            action_level: critical
            handler: buffered
        buffered:
            type: buffer
            handler: swift
        swift:
            type: swift_mailer
            from_email: error@domain.com
            to_email: error@domain.com
            subject: An Error Occurred!
            level: debug
```

#### Email Logs Related to 4xx Errors (`action_level: error`)
```yaml
# app/config/config_prod.yml
monolog:
    handlers:
        mail:
            type: fingers_crossed
            action_level: error
            handler: buffered
        buffered:
            type: buffer
            handler: swift
        swift:
            type: swift_mailer
            from_email: error@domain.com
            to_email: error@domain.com
            subject: An Error Occurred!
            level: debug
```

#### Do Not Email Logs for 404 Errors (`excluded_404`)
```yaml
# app/config/config_prod.yml
monolog:
    handlers:
        mail:
            type: fingers_crossed
            action_level: error
            excluded_404:
                - ^/
            handler: buffered 
        buffered:
            type: buffer
            handler: swift
        swift:
            type: swift_mailer
            from_email: error@domain.com
            to_email: error@domain.com
            subject: An Error Occurred!
            level: debug
```

### Import

#### Import Mixed Configuration Files
```yaml
# app/config/config.yml
imports:
    - { resource: '../common/config.yml' }
    - { resource: 'dynamic-config.php' }
    - { resource: 'parameters.ini' }
    - { resource: 'security.xml' }
    # ...
```

#### Import All Resources From a Directory
```yaml
# app/config/config.yml
imports:
    - { resource: '../common/' }
    - { resource: 'acme/' }
    # ...
```

#### Import Configuration Files Using Glob Patterns
`Symfony 3.3`
```yaml
# app/config/config.yml
imports:
    - { resource: "*.yml" }
    - { resource: "common/**/*.xml" }
    - { resource: "/etc/myapp/*.{yml,xml}" }
    - { resource: "bundles/*/{xml,yaml}/services.{yml,xml}" }
    # ...
```
[[1]](http://symfony.com/blog/new-in-symfony-3-3-import-config-files-with-glob-patterns)

### Log

#### Enable the Monolog processor PsrLogMessageProcessor
```yaml
# app/config/config_prod.yml
services:
    monolog_processor:
        class: Monolog\Processor\PsrLogMessageProcessor
        tags:
            - { name: monolog.processor }
```

#### Hide Event Logs
```yaml
# app/config/dev.yml
monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
            channels: "!event"
```

#### Organizing Log Files Using Channels (Log Messages to Different Files)
```yaml
# app/config/config_prod.yml
monolog:
    handlers:
        main:
            type: stream
            path: "%kernel.logs_dir%/%kernel.environment%.log"
            level: debug
            channels: ["!event"]
        security:
            type: stream
            path: "%kernel.logs_dir%/security-%kernel.environment%.log"
            level: debug
            channels: "security"
```
[[1]](http://symfony.com/doc/current/logging/channels_handlers.html)

### Security

#### Impersonating Users
```yaml
# app/config/security.yml
security:
    firewalls:
        main:
            # ...
            switch_user: true
```

Switching the user in the URL:
`http://domain.com/path?_switch_user=john`

### Session

#### Define Session Lifetime
```yaml
# app/config/config.yml
framework:
    session:
        cookie_lifetime: 3600
```

### Profiler

#### Enable the Profiler on Prod For Specific Users
```yaml
# app/config/config.yml
framework:
    # ...
    profiler:
       matcher:
           service: app.profiler_matcher

services:
    app.profiler_matcher:
        class: AppBundle\Profiler\Matcher
        arguments: ["@security.context"]
```

```php
namespace AppBundle\Profiler; 

use Symfony\Component\Security\Core\SecurityContext; 
use Symfony\Component\HttpFoundation\Request; 
use Symfony\Component\HttpFoundation\RequestMatcherInterface; 

class Matcher implements RequestMatcherInterface 
{ 
    protected $securityContext; 

    public function __construct(SecurityContext $securityContext)
    {
        $this->securityContext = $securityContext; 
    } 

    public function matches(Request $request)
    { 
        return $this->securityContext->isGranted('ROLE_ADMIN'); 
    }
}
```

## Console

### Parallel Asset Dump
`Symfony 2.8`
```bash
    $ php app/console --env=prod assetic:dump --forks=4
```

`Symfony 3+`
```bash
    $ php bin/console --env=prod assetic:dump --forks=4
```

## Controller

### Cookie

#### Set a Cookie
```php
use Symfony\Component\HttpFoundation\Cookie;

$response->headers->setCookie(new Cookie('site', 'bar'));
```

### Directories, Paths, and Filesystem

#### Root Directory of the Project
The parameter `kernel.root_dir` points to the `app` directory. To get to the root project directory, use `kernel.root_dir/../`
```php
realpath($this->getParameter('kernel.root_dir')."/../")
```

#### Check If a Path is Absolute
```php
use Symfony\Component\Filesystem\Filesystem;

//...

$fs = new FileSystem();
$fs->isAbsolutePath('/tmp'); // return true
$fs->isAbsolutePath('c:\\Windows'); // return true
$fs->isAbsolutePath('tmp'); // return false
$fs->isAbsolutePath('../dir'); // return false
```
[[1]](symfony.com/doc/current/components/filesystem.html)

### Download

#### Download (Serve) a Static File
```php
use Symfony\Component\HttpFoundation\BinaryFileResponse;

// ...

return new BinaryFileResponse('path/to/file');
```

#### Check If a File Exists
```php
use Symfony\Component\Filesystem\Filesystem;

//...

$fs = new FileSystem();
if (!$fs->exists($filepath)) {
    throw $this->createNotFoundException();
}
```
[[1]](symfony.com/doc/current/components/filesystem.html)

#### Download a File Without Directly Expose it and Change the Filename
```php
use Symfony\Component\HttpFoundation\BinaryFileResponse;
use Symfony\Component\Filesystem\Filesystem;

$filename = // define your filename
$basePath = $this->getParameter('kernel.root_dir').'/../uploads';
$filePath = $basePath.'/'.$filename;

$fs = new FileSystem();
if (!$fs->exists($filepath)) {
    throw $this->createNotFoundException();
}

$response = new BinaryFileResponse($filePath);
$response->trustXSendfileTypeHeader();
$response->setContentDisposition(
    ResponseHeaderBag::DISPOSITION_INLINE,
    $filename,
    iconv('UTF-8', 'ASCII//TRANSLIT', $filename)
);

return $response;
```

#### Using `X-Sendfile` Header with `BinaryFileResponse`

BinaryFileResponse supports `X-Sendfile` (Nginx and Apache). To use of it,
you need to determine whether or not the `X-Sendfile-Type` header should be
trusted and call `trustXSendfileTypeHeader()` if it should:
```php
BinaryFileResponse::trustXSendfileTypeHeader();
```
or
```php
$response = new BinaryFileResponse($filePath);
$response->trustXSendfileTypeHeader();
```

### Flash Messages

#### Set Multiple Flash Messages
```php
use Symfony\Component\HttpFoundation\Session\Session;

$session = new Session();
$session->start();

$session->getFlashBag()->add(
    'warning',
    'Your config file is writable, it should be set read-only'
);
$session->getFlashBag()->add('error', 'Failed to update name');
$session->getFlashBag()->add('error', 'Invalid email');
```
[[1]](https://symfony.com/doc/current/components/http_foundation/sessions.html#flash-messages)

### Form

#### Get Errors for All Form Fields
`Symfony 2.5+`
```php
$allErrors = $form->getErrors(true);
```

### JSON

#### Avoiding XSSI JSON Hijacking (only GET requests are vulnerable)
Pass an associative array as the outer-most array to `JsonResponse` 
and not an indexed array so that the final result is an object: 
`{"object": "not inside an array"}`
instead of an array: 
`[{"object": "inside an array"}]`

#### Create a JSON Response with `JsonResponse` Class
```php
use Symfony\Component\HttpFoundation\JsonResponse;

$response = new JsonResponse();
$response->setData(array(
    'name' => 'John'
));
```

#### Create a JSON Response with `Response` Class
```php
use Symfony\Component\HttpFoundation\Response;

$response = new Response();
$response->setContent(json_encode(array(
    'name' => 'John',
)));
$response->headers->set('Content-Type', 'application/json');
```

#### Set JSONP Callback Function
```php
$response->setCallback('handleResponse');
```

### Redirect

#### Redirect to Another URL
```php
return $this->redirect('http://domain.com');
```
or
```php
use Symfony\Component\HttpFoundation\RedirectResponse;

$response = new RedirectResponse('http://domain.com');
```

### Request

#### Get the Request Object

`Symfony 2`
```php
namespace Acme\FooBundle\Controller;

class DemoController
{
   public function showAction()
   {
       $request = $this->getRequest();
       // ...
   }
}
```

`Symfony 3`
```php
namespace Acme\FooBundle\Controller;

use Symfony\Component\HttpFoundation\Request;

class DemoController
{
   public function showAction(Request $request)
   {
       // ...
   }
}
```
[[1]](https://github.com/symfony/symfony/blob/2.8/UPGRADE-3.0.md#frameworkbundle)

#### Get the Request Raw Data Sent with the Request Body
```php
$content = $request->getContent();
```

#### Fetch a GET Parameter
```php
$request->query->get('site');
```

#### Fetch a GET Parameter in array format (data['name'])
```php
$request->query->get('data')['name'];
```

#### Fetch a POST Parameter
```php
$request->request->get('name');
```

#### Fetch a GET Parameter Specifying the Data Type
```php
$isActive = $request->query->getBoolean('active');
$page     = $request->query->getInt('page'); 
```
Other methods are:
- getAlpha('param');
- getAlnum('param');
- getDigits('param');
[[1]](http://symfony.com/doc/current/components/http_foundation.html#accessing-request-data)

### Response

#### Set a HTTP Status Code
```php
use Symfony\Component\HttpFoundation\Response;

$response->setStatusCode(Response::HTTP_NOT_FOUND);
```

### Routing

#### External URLs
```yaml
google_search:
    path: /search
    host: www.google.com
```

```twig
<a href="{{ url('google_search', {q: 'Jules Verne'}) }}">Jules Verne</a>
```

#### External URLs - Using a Key to Reference a URL
```yaml
framework:
    assets:
        packages:
            symfony_site:
                version: ~
                base_urls: 'https://symfony.com/images'
```

Add images from the URL above into your views, using the "symfony_site" key in the second argument of asset():
```twig
<img src="{{ asset('logos/header-logo.svg', 'symfony_site') }}">
```

#### Generate Absolute URL
`Symfony 2`
```php
$this->generateUrl('blog_show', array('slug' => 'my-blog-post'), true);
```

`Symfony 3`
```php
$this->generateUrl('blog_show', array('slug' => 'my-blog-post'), UrlGeneratorInterface::ABSOLUTE_URL);
```

#### Trailing Slash with an Optional Parameter
```yaml
my_route:
    pattern:  /blog/{var}
    defaults: { _controller: TestBundle:Blog:index, var: ''}
    requirements:
        var: ".*"
```

### Service

#### Retrieve a Service
```php
$this->get('service.name');
```
or
```php
$this->container->get('service.name'),
```

`Symfony 4+`

Using autowiring, just type-hint the desired service. E.g. getting the routing service:

```php
use Symfony\Component\Routing\RouterInterface;

class SomeClass
{
    private $router;

    public function __construct(RouterInterface $router)
    {
        $this->router = $router;
    }

    public function doSomething($id)
    {
        $url = $this->router->generate('route_name', ['id' => $id]);

        // ...
    }

    // ...
```

### YAML

#### Parse YAML File
```php
use Symfony\Component\Yaml\Exception\ParseException;

try {
    $value = Yaml::parse(file_get_contents('/path/to/file.yml'));
} catch (ParseException $e) {
    printf("Unable to parse the YAML string: %s", $e->getMessage());
}
```
[[1]](http://symfony.com/doc/current/components/yaml.html#using-the-symfony-yaml-component)

## Environment Variables

### Custom Loader for Environment Variables
`Symfony 4.4`
```yaml
# config/services.yaml
bind:
    string $name: '%env(name)%'
```

Implement the `EnvVarLoaderInterface` in a service:
```php
namespace App\Env;

use Symfony\Component\DependencyInjection\EnvVarLoaderInterface;

class ConsulEnvVarLoader implements EnvVarLoaderInterface
{
    public function loadEnvVars(): array
    {
        $response = file_get_contents('http://127.0.0.1:8500/v1/kv/website-config');

        $consulValue = json_decode($response, true)[0]['Value'];
        $decoded = json_decode(base64_decode($consulValue), true);

        // e.g.:
        // array:1 [
        //     "name" => "my super website"
        // ]

        return $decoded;
    }
}
```

Update the consul KV:
```bash
./consul  kv put website-config '{"name": "Symfony read this var from consul"}'
```
[[1]](https://github.com/symfony/symfony/pull/34295#issuecomment-551899027)

## Twig

### Absolute URLs
`Symfony 2.6`
```twig
{{ asset('logo.png', absolute = true) }}
```

`Symfony 2.7+`
```twig
{{ absolute_url(asset('logo.png')) }}
```

### Assets Versioning
`Symfony 2.6`
```twig
{{ asset('logo.png', version = 'v5') }}
```

`Symfony 2.7+`
Version is automatically appended.
```twig
{{ asset('logo.png') }}

{# use the asset_version() function if you need to output it manually #}
{{ asset_version('logo.png') }}
```
[[1]](http://symfony.com/blog/new-in-symfony-2-7-the-new-asset-component#template-function-changes)

### Get the Authenticated Username
```twig
{{ app.user.username }}
```

### Localized Date String
In your Twig template, you can use [pre-defined](http://twig.sensiolabs.org/doc/extensions/intl.html) or [custom date formats](http://userguide.icu-project.org/formatparse/datetime) with the `localizeddate`:

```twig
{{ blog.created|localizeddate('none', 'none', 'pt_BR', null, "cccc, d MMMM Y 'às' hh:mm aaa")}}
```
The pattern `"cccc, d MMMM Y 'às' hh:mm aaa"` will show the date in this format:

`domingo, 5 janeiro 2014 às 03:00 am`

#### Get the Base URL
```php
{{ app.request.getSchemeAndHttpHost() }}
```

### Inject All GET Parameters in a Route
```twig
{{ path('home', app.request.query.all) }}
```

### Make the `form_rest()` and `form_end()` not Display a Specific Field
Mark the field as rendered (`setRendered`)
```twig
{% do form.somefield.setRendered %}
```

### Render a Template without a Specific Controller for a Static Page
Use the special controller FrameworkBundle:Template:template in the route definition:
```yaml
# AppBundle/Resources/config/routing.yml
static_page:
    path: /about
    defaults:
        _controller: FrameworkBundle:Template:template
        template: AppBundle:default:about.html.twig
```

### Override the 404 Error Template
Create a new `error404.html.twig` template at:
```twig
app/Resources/TwigBundle/views/Exception/
```
[[1]](https://symfony.com/doc/current/controller/error_pages.html#example-404-error-template)

### Render a Controller Asynchronously
```twig
{{ render_hinclude(controller('AppBundle:Features:news', {
    'default': 'Loading...'
})) }}
```
[[1]](http://symfony.com/doc/current/templating/hinclude.html)

### Render Just the Close Form HTML Tag
```twig
{{ form_end(form, {'render_rest': false}) }}
```

### Using Localized Data (Date, Currency, Number, ...)
Enable the `intl` twig extension in `config.yml` or `services.yml` file:

```yaml
services:
    twig.extension.intl:
        class: Twig_Extensions_Extension_Intl
        tags:
            - { name: twig.extension }
```
