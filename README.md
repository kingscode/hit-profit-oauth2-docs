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
> For example: https://your-application.com/oauth/callback
