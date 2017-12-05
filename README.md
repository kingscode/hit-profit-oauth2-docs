# HIT Profit OAuth2 Documentation
This is the official documentation for the `hit-profit.nl` OAuth2 server.
> This documentation assumes you are already familiar with OAuth2. 
> If you do not know anything about OAuth2, 
> consider familiarizing yourself with the general terminology and features of OAuth2 before continuing.

## Requesting access to the OAuth2 server
Given that the OAuth2 server is only accessible to trusted parties, access tokens must be requested in contrary to having an open service for anyone to use.

To gain access to the HIT Profit OAuth2 server, send a mail to:
> info@documentready.nl

In this mail you must specify a Redirect URI which will be the URI that the server should redirect to after the user has authorized your application.
> For example: https://example.com/oauth/callback

## Tutorial
### Requesting tokens
#### Redirecting for authorization
Once you have requested access and received your `client_id` and `client_secret` you may start by making a redirect request
to the `https://hit-profit.nl/oauth/authorize` that has the following parameters:

- **client_id**: _Your client id._
- **redirect_uri**: _The redirect URI you have specified when requesting access to the OAuth2 server._
- **response_type**: _This should be `code`_
- **scope**: _This should be `sso`._

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id'     => 'client_id',
        'redirect_uri'  => 'http://example.com/oauth/callback',
        'response_type' => 'code',
        'scope'         => 'sso',
    ]);

    return redirect('https://hit-profit.nl/oauth/authorize?'.$query);
});
```

Which results in a redirect to: `https://hit-profit.nl/oauth/authorize?client_id=client_id&redirect_uri=http%3A%2F%2Fexample.com%2Foauth%2Fcallback&response_type=code&scope=sso`
