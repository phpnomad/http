# phpnomad/http

[![Latest Version](https://img.shields.io/packagist/v/phpnomad/http.svg)](https://packagist.org/packages/phpnomad/http)
[![Total Downloads](https://img.shields.io/packagist/dt/phpnomad/http.svg)](https://packagist.org/packages/phpnomad/http)
[![PHP Version](https://img.shields.io/packagist/php-v/phpnomad/http.svg)](https://packagist.org/packages/phpnomad/http)
[![License](https://img.shields.io/packagist/l/phpnomad/http.svg)](https://packagist.org/packages/phpnomad/http)

`phpnomad/http` defines the interfaces and enum that describe an HTTP request and response as data. It is not an HTTP client and it is not an HTTP server. It is the contract layer both of those roles build on, so the same `Request` and `Response` types flow through every PHPNomad package that touches HTTP.

Outbound HTTP lives in `phpnomad/fetch`, which returns a `Response` implementation after making a call. Inbound HTTP lives in `phpnomad/rest`, which hands a `Request` to a controller and expects a `Response` back. Concrete implementations come from host integrations such as the WordPress integration, the FastRoute integration, or the Guzzle integration. Because every stack agrees on the same contracts, controllers and client calls written against `phpnomad/http` move between runtimes without rewrites. This package is used by Siren in production and sits underneath most of the PHPNomad framework's HTTP-facing code.

## Installation

```bash
composer require phpnomad/http
```

## Quick Start

A typical consumer is a REST controller that reads parameters off a `Request` and builds a `Response` with the fluent setters. The controller declares its HTTP verb with the `Method` enum, and the actual `Request` and `Response` instances are supplied by whichever integration package is handling inbound traffic.

```php
<?php

namespace MyApp\Posts\Rest;

use MyApp\Posts\Services\PostService;
use PHPNomad\Http\Enums\Method;
use PHPNomad\Http\Interfaces\Request;
use PHPNomad\Http\Interfaces\Response;
use PHPNomad\Rest\Interfaces\Controller;

class CreatePost implements Controller
{
    protected Response $response;
    protected PostService $posts;

    public function __construct(Response $response, PostService $posts)
    {
        $this->response = $response;
        $this->posts = $posts;
    }

    public function getResponse(Request $request): Response
    {
        $title = $request->getParam('title');
        $body = $request->getParam('body');

        if (!$title || !$body) {
            $this->response->setError('Title and body are required.', 400);
            return $this->response;
        }

        $post = $this->posts->create($title, $body);

        return $this->response
            ->setJson(['id' => $post->getId(), 'title' => $post->getTitle()])
            ->setStatus(201);
    }

    public function getMethod(): string
    {
        return Method::Post;
    }
}
```

`getParam()` supports dot notation for reading nested values. `setJson()`, `setStatus()`, `setHeader()`, and `setBody()` all return the same `Response` for chaining, and `setError()` writes an error payload with a status code in one call.

## Key Concepts

- `Request` interface covers headers, parameters (with dot-notation lookup), raw body access, and the authenticated user via the `HasUser` contract from `phpnomad/auth`.
- `Response` interface covers status codes, headers, body content as string or JSON, an error helper, and fluent setters that return the response for chaining.
- `Method` enum exposes `Get`, `Post`, `Put`, `Delete`, `Patch`, and `Options` as string constants, backed by `phpnomad/enum-polyfill` so the same API works on PHP versions without native enums.
- This package ships interfaces only. Concrete `Request` and `Response` classes come from integration packages that bind them to a host runtime.
- Both `phpnomad/rest` (inbound) and `phpnomad/fetch` (outbound) depend on these contracts, which is what lets controllers and client calls stay portable across stacks.

## Documentation

Full framework documentation lives at [phpnomad.com](https://phpnomad.com), including the bootstrapping guide that shows how HTTP interfaces get wired up alongside the rest of a PHPNomad application.

## License

Released under the MIT License. See [LICENSE](LICENSE) for details.
