# patreon-php
Interact with the Patreon API via OAuth. (This is a development package that will be removed later. Please do not use this)

## Installation

Get the plugin from [Packagist](https://packagist.org/packages/patreon/patreon)

## Usage
### Step 1. Get your client_id and client_secret
Visit the [Patreon platform documentation page](https://www.patreon.com/platform/documentation)
while logged in as a Patreon creator to register your client.

This will provide you with a `client_id` and a `client_secret`.

### Step 2. Use this plugin in your code
Let's say you wanted to make a "Log In with Patreon" button.
You've read through [the directions](https://www.patreon.com/platform/documentation/oauth),
and are trying to implement "Step 2: Handling the OAuth Redirect" with your server.
The user will be arriving at one of your pages *after* you have sent them to [the authorize page](www.patreon.com/oauth2/authorize) for step 1,
so in their query parameters landing on this page,
they will have a parameter `'code'`.

_(If you are doing something other than the "Log In with Patreon" flow, please see [the examples folder](examples) for more examples)_

```php
<?php
 
/*
 * Use the following,
 * or if you installed via Composer, it should already be required via autoloader
 */

require_once __DIR__.'/vendor/autoload.php';
 
use Codebard\API;
use Codebard\OAuth;

$client_id = '';      // Replace with your data
$client_secret = '';  // Replace with your data

$oauth_client = new OAuth($client_id, $client_secret);

// There will a simple login link generator from a new class here - instead of the makeshift code below

$redirect_uri = "http://pat-php-dev.codebard.com";

$href = 'https://www.patreon.com/oauth2/authorize?response_type=code&client_id=' 
. $client_id . '&redirect_uri=' . urlencode($redirect_uri).'&scope=identity%20identity'.urlencode('[email]');

// Scopes! You must request the scopes you need to have the access token.
// In this case, we are requesting the user's identity (basic user info), user's email
// For example, if you do not request email scope while logging the user in, later you wont be able to get user's email via /identity endpoint when fetching the user details

$scope_parameters = '&scope=identity%20identity'.urlencode('[email]');

$href .= $scope_parameters;

echo '<a href="'.$href.'">Click here to login via Patreon</a>';
echo '<br>';


if ( $_GET['code'] != '' ) {
		
	$tokens = $oauth_client->get_tokens($_GET['code'], $redirect_uri);
	$access_token = $tokens['access_token'];
	$refresh_token = $tokens['refresh_token'];
	
	// There will be some advice for devs on how to save their tokens and match it to their users here

	$api_client = new API($access_token);
	
	// Return from the API can be received in either array, object or JSON formats by setting the return format. It defaults to array if not specifically set. Specifically setting return format is not necessary. Below is shown as an example of having the return parsed as an object. If there is anyone using Art4 JSON parser lib or any other parser, they can just set the API return to JSON and then have the return parsed by that parser
	
	$api_client->api_return_format = 'object';
	
	// Now get the current user:
	$patron_response = $api_client->fetch_user();
	
	echo '<pre>';
	print_r($patron_response);
	echo '</pre>';
	
}


?>
```
