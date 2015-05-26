# HttpClient Middleware for PSR-7

Missing interfaces for HTTP-Client middlewares using PSR-7 messages.
There are four 

## A Simple Http Client

Very simple http client interface that avoids factories by specifing
all request variables directly.

```php
<?php

interface SimpleClientInterface
{
    /**
     * @param string $method
     * @param string|UriInterface $uri
     * @param array $options - Vendor-specific options, don't rely on for interop.
     * 
     * Standard compliant client must implement the following options:
     *
     * - "headers" is a list of headers, for example ["Content-Type: application/json"]
     * - "body" contains the HTTP body if sending POST, PUT, ...
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function request($method, $uri, array $options = []);
}
```

Usage:

```php
<?php

$client = createMySimpleClient();
$response = $client->request('GET', 'http://php.net');
```

## Http Client Interface

A more flexible HTTP client allows access to the Request object
and provides a factory to create Request and Uri objects to 
hide the underyling implementation.

```php
<?php
interface ClientInterface
{
    /**
     * @return \Psr\Http\Message\RequestInterface
     */
    public function createRequest();

    /**
     * @param string $uri - If uri is provided the result of parse_url() is set as parts for UriInterface
     * @return \Psr\Http\Message\UriInterface
     */
    public function createUri($uri = null);

    /**
     * @param \Psr\Http\Message\RequestInterface $request
     * @param array $options - Vendor-specific options, don't rely on for interop.
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function send(RequestInterface $request, array $options = []);
}
```

Usage:

```php
<?php

$client = createMyHttpClient();
$request = $client->createRequest();
    ->withMethod('GET')
    ->withUri($client->createUri("http://php.net"))
    ->withHeader("X-Foo", "Bar");

$response = $client->send($request);
```

## Asynchronuous Http-Client

Optional interface when you want your client to be async. Uses promises
response that looks exactly the same as [React
Promises](https://github.com/reactphp/promise) to avoid having to build a new
one.

```php
<?php
interface AsyncClientInterface
{
    /**
     * @param \Psr\Http\Message\RequestInterface $request
     * @param array $options - Vendor-specific options, don't rely on for interop.

     * @return PromiseInterface
     */
    public function sendAsync(RequestInterface $request, array $options = []);

    /**
     * @param string $method
     * @param string|UriInterface $uri
     * @param array $options - Vendor-specific options, don't rely on for interop.
     *
     * @return PromiseInterface
     */
    public function requestAsync($method, $uri, array $options = []);
}

/**
 * API Compatible to React\PromiseInterface to allow usage.
 */
interface PromiseInterface
{
    /**
     * @return PromiseInterface
     */
    public function then(callable $onFulfilled = null, callable $onRejected = null, callable $onProgress = null);
}
```

## RequestFilter and ResposneFilter for Plugins (OAuth, HTTP-Basic, ...)

If your HTTP client supports plugins, use these interfaces to allow others to
easily plugin into all libraries that support them.

For plugin authors, implement these interfaces and instantly work with all
HTTP Clients that use the HttpClient Middleware.

```php
<?php
interface RequestFilterInterface
{
    /**
     * @return \Psr\Http\Message\Request $request
     */
    public function filterRequest(Request $request);
}

interface ResponseFilterInterface
{
    /**
     * @return \Psr\Http\Message\Request $request
     */
    public function filterResponse(Response $response);
}
```