# HIT Profit OAuth2 Documentation
This is the official documentation for the `https://hitsolution.nl/api` OAuth2 server.
> This documentation assumes you are already familiar with OAuth2.
> If you do not know anything about OAuth2,
> consider familiarizing yourself with the general terminology and features of OAuth2 before continuing.

## Requesting access to the OAuth2 server
Given that the OAuth2 server is only accessible to trusted parties, access tokens must be requested in contrary to having an open service for anyone to use.

To gain access to the HIT Profit OAuth2 server, send a mail to:
> info@kingscode.nl

In this mail you must specify a Redirect URI which will be the URI that the server should redirect to after the user has authorized your application.
> For example: https://example.com/oauth/callback

And the name of your application, this will be visible to the user.
> For example: HIT My Administration

# Usage
## Requesting tokens
### Redirecting for authorization
Once you have requested access and received your `client_id` and `client_secret` you may start by making a redirect request
to the `https://hitsolution.nl/api/oauth/authorize` URI that has the following parameters:

- **client_id**: Your client id.
- **redirect_uri**: The redirect URI you have specified when requesting access to the OAuth2 server.
- **response_type**: This should be `code`.
- **scope**: This should be `sso`.

```php
Route::get('/oauth/redirect', function () {
    $query = http_build_query([
        'client_id'     => 'client-id',
        'redirect_uri'  => 'https://example.com/oauth/callback',
        'response_type' => 'code',
        'scope'         => 'sso',
    ]);

    return redirect('https://hitsolution.nl/api/oauth/authorize?'.$query);
});
```

Which results in a redirect to: `https://hitsolution.nl/api/oauth/authorize?client_id=client_id&redirect_uri=http%3A%2F%2Fexample.com%2Foauth%2Fcallback&response_type=code&scope=sso`

### Converting authorization codes to access tokens
If the user approves the authorization request, they will be redirected back to your application.
You should then issue a `POST` request to the `https://hitsolution.nl/api/oauth/token` URI to request an access token.
The request should include the following parameters:

- **grant_type**: This should be `authorization_code`.
- **client_id**: Your client id.
- **client_secret**: Your client secret.
- **redirect_uri**: The redirect URI you have specified when requesting access to the OAuth2 server.
- **code**: The code that is sent back to your application via a `GET` parameter.

```php
Route::get('/oauth/callback', function (Request $request) {
    $http = new GuzzleHttp\Client;

    $response = $http->post('https://hitsolution.nl/api/oauth/token', [
        'form_params' => [
            'grant_type'    => 'authorization_code',
            'client_id'     => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri'  => 'https://example.com/oauth/callback',
            'code'          => $request->code,
        ],
    ]);

    return json_decode((string) $response->getBody(), true);
});
```

This will return a JSON response which contains the following attributes:

- **access_token**: The OAuth2 access token. (used to identify the user)
- **refresh_token**: The OAuth2 refresh token. (used for refreshing the access token)
- **expires_in**: The time in seconds when the token will expire.

## Implicit grant tokens
The implicit grant is similar to the authorization code grant; however, the token is returned to the client without exchanging an authorization code.
This grant is most commonly used for JavaScript or mobile applications where the client credentials can't be securely stored.

For this you should make a request to `https://hitsolution.nl/api/oauth/authorize` with the following parameters:

- **client_id**: Your client id.
- **redirect_uri**: The redirect URI you have specified when requesting access to the OAuth2 server.
- **response_type**: This should be `token`.
- **scope**: This should be `sso`.

```php
Route::get('/oauth/redirect', function () {
    $query = http_build_query([
        'client_id'     => 'client-id',
        'redirect_uri'  => 'http://example.com/oauth/callback',
        'response_type' => 'token',
        'scope'         => 'sso',
    ]);

    return redirect('http://hitsolution.nl/api/oauth/authorize?'.$query);
});
```

## Refreshing tokens
It is possible to refresh the `access_token` via the `https://hitsolution.nl/api/oauth/token` uri by using the `refresh_token` that was issued when authorizing.
The request should include the following parameters:

- **grant_type**: This should be `refresh_token`.
- **refresh_token**: The refresh token of the user.
- **client_id**: Your client id.
- **client_secret**: Your client secret.
- **scope**: This should be `sso`.

```php
$http = new GuzzleHttp\Client;

$response = $http->post('https://hitsolution.nl/api/oauth/token', [
    'form_params' => [
        'grant_type'    => 'refresh_token',
        'refresh_token' => 'your-refresh-token',
        'client_id'     => 'client-id',
        'client_secret' => 'client-secret',
        'scope'         => 'sso',
    ],
]);

return json_decode((string) $response->getBody(), true);
```

This will return a JSON response which contains the following attributes:

- **access_token**: The OAuth2 access token. (used to identify the user)
- **refresh_token**: The OAuth2 refresh token. (used for refreshing the access token)
- **expires_in**: The time in seconds when the token will expire.

## Getting user data
### Me
After the whole OAuth2 dance has been completed, you can make a `GET` request to `https://hitsolution.nl/api/v1/me` which enables you to get some of the user's data.

You should specify the access token as a `Bearer` token in the `Authorization` header of your request, like so:
```text
Authorization: Bearer access_token
```

But this also enables you to check if the token is still valid and thus if the user is still logged in, if that's not the case you'll get a proper `401 Unauthorized` that includes a json response like this:

```json
{
    "message": "Unauthenticated."
}
```

## Revoking tokens
It is possible to revoke the granted access token, for example, if the user wants to logout and revoke access to their account.

For this you can make a `DELETE` (or a `POST` with a parameter `_method=DELETE`) request to `https://hitsolution.nl/api/oauth/token`. (keep in mind that you need to have the `Authorization` header set to the correct value).
