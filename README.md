# Ahoi Provider for OAuth 2.0 Client
[![Latest Version](https://img.shields.io/github/release/ahoicloud/oauth2-client.svg?style=flat-square)](https://github.com/ahoicloud/oauth2-client/releases)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Build Status](https://img.shields.io/travis/ahoicloud/oauth2-client/master.svg?style=flat-square)](https://travis-ci.org/ahoicloud/oauth2-client)
[![Coverage Status](https://img.shields.io/scrutinizer/coverage/g/ahoicloud/oauth2-client.svg?style=flat-square)](https://scrutinizer-ci.com/g/ahoicloud/oauth2-client/code-structure)
[![Quality Score](https://img.shields.io/scrutinizer/g/ahoicloud/oauth2-client.svg?style=flat-square)](https://scrutinizer-ci.com/g/ahoicloud/oauth2-client)
[![Total Downloads](https://img.shields.io/packagist/dt/ahoicloud/oauth2-client.svg?style=flat-square)](https://packagist.org/packages/ahoicloud/oauth2-client)

This package provides Ahoi OAuth 2.0 support for the PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

## Installation

To install, use composer:

```
composer require ahoicloud/oauth2-client
```

## Usage

Usage is the same as The League's OAuth client, using `\FVJM\OAuth2\Client\Provider\Ahoi` as the provider.

### Authorization Code Flow

```php
$provider = new FVJM\OAuth2\Client\Provider\Ahoi([
    'clientId'          => '{ahoi-client-id}',
    'clientSecret'      => '{ahoi-client-secret}',
    'ahoiInstanceUrl'      => '{ahoi-instance-url}',
    'redirectUri'       => 'https://example.com/callback-url'
]);

if (!isset($_GET['code'])) {

    // If we don't have an authorization code then get one
    $authUrl = $provider->getAuthorizationUrl();
    $_SESSION['oauth2state'] = $provider->getState();
    header('Location: '.$authUrl);
    exit;

// Check given state against previously stored one to mitigate CSRF attack
} elseif (empty($_GET['state']) || ($_GET['state'] !== $_SESSION['oauth2state'])) {

    unset($_SESSION['oauth2state']);
    exit('Invalid state');

} else {

    // Try to get an access token (using the authorization code grant)
    $token = $provider->getAccessToken('authorization_code', [
        'code' => $_GET['code']
    ]);

    // Optional: Now you have a token you can look up a users profile data
    try {

        // We got an access token, let's now get the user's details
        $user = $provider->getResourceOwner($token);

        // Use these details to create a new profile
        printf('Hello %s!', $user->getFirstname());

    } catch (Exception $e) {

        // Failed to get user details
        exit('Oh dear...');
    }

    // Use this to interact with an API on the users behalf
    echo $token->getToken();
}
```

### Managing Scopes

When creating your Ahoi authorization URL, you can specify the state and scopes your application may authorize.

```php
$options = [
    'state' => 'OPTIONAL_CUSTOM_CONFIGURED_STATE',
    'scope' => ['profile','offline_acccess'] // array or string
];

$authorizationUrl = $provider->getAuthorizationUrl($options);
```
If neither are defined, the provider will utilize internal defaults.

At the time of authoring this documentation, the following scopes are available.

- profile
- offline_acccess

### Refreshing a Token

```php
$provider = new FVJM\OAuth2\Client\Provider\Ahoi([
    'clientId'          => '{ahoi-client-id}',
    'clientSecret'      => '{ahoi-client-secret}',
    'ahoiInstanceUrl'      => '{ahoi-instance-url}',
    'redirectUri'       => 'https://example.com/callback-url'
]);

$grant = new \League\OAuth2\Client\Grant\RefreshToken();
$token = $provider->getAccessToken($grant, ['refresh_token' => $refreshToken]);
```

### Client Credentials Grant

When your application is acting on its own behalf to access resources it controls/owns in a service provider, it may use the client credentials grant type. This is best used when the credentials for your application are stored privately and never exposed (e.g. through the web browser, etc.) to end-users. This grant type functions similarly to the resource owner password credentials grant type, but it does not request a user's username or password. It uses only the client ID and secret issued to your client by the service provider.


``` php

$provider = new FVJM\OAuth2\Client\Provider\Ahoi([
    'clientId'          => '{ahoi-client-id}',
    'clientSecret'      => '{ahoi-client-secret}',
    'ahoiInstanceUrl'      => '{ahoi-instance-url}',
    'redirectUri'       => 'https://example.com/callback-url'
]);

try {

    // Try to get an access token using the client credentials grant.
    $accessToken = $provider->getAccessToken('client_credentials');

} catch (\League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {

    // Failed to get the access token
    exit($e->getMessage());

}
```

## Testing

``` bash
$ ./vendor/bin/phpunit
```


## Credits

- [Steven Maguire](https://github.com/stevenmaguire)
- [All Contributors](https://github.com/ahoicloud/oauth2-client/contributors)


## License

The MIT License (MIT). Please see [License File](https://github.com/ahoicloud/oauth2-client/blob/master/LICENSE) for more information.
