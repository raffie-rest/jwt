# JWT
Google® oAuth2 JWT Generator Class

- [Using OAuth 2.0 to Access Google APIs](https://developers.google.com/accounts/docs/OAuth2)
- [Using OAuth 2.0 for Server to Server Applications](https://developers.google.com/accounts/docs/OAuth2ServiceAccount)

## Introduction

This class is already hooked up in another proprietary Remote REST abstraction, but I aim to implement the Google oAuth2 service account auth method by virtue of this class in my [Adapter](https://github.com/raffie-rest/adapter) project also.

For the moment, you might be able to incorporate it in your own Remote REST abstraction.

It is exactly intended for that purpose; those who find the PHP google client libraries too bloated, but have a hard time figuring out how to generate a JWT / negotiating for a token whilst dodging cryptic `bad_request` / `invalid_grant` responses. Mostly this is due to the system clock being out of sync with NTP, but this class accounts for this.

## Generating a JWT

Feed it a config array, like so:

	$jwtConfig = [
	  'key'    => [
	    'pass'  => 'notasecret',
	    // Converted from the .p12 off of the Developer console
	    'path'  => 'file://' . storage_path() . '/certs/google.pem'
	  ],
	  'header'  => [
	    'alg'  => 'RS256',
	    'typ'  => 'JWT'
	  ],
	  'claim_set' => [
	    'iss'     => '',  // service account e-mail
	    'scope'   => 'https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/prediction',
	    'aud'     => 'https://www.googleapis.com/oauth2/v3/token',
	    'exp'     => '+30 minutes',
	    'sub'     => ''
	  ]
	];
	
	$generator = new Generator($jwtConfig);
	
	$returnedJwt = $generator->generate();

Make sure that the relevant API's are enabled, and the user has read/write access. 

## Negotiate for an access token

The JWT can be used to negotiate an access token via the pseudocode below:

	POST https://www.googleapis.com/oauth2/v3/token HTTP/1.1
	Content-Type: application/x-www-form-urlencoded
	grant_type=urlencode(JWT::$grant_type)&assertion=urlencode($returnedJwt)

In the case of Google oAuth2, the `grant_type` is virtually always `urn:ietf:params:oauth:grant-type:jwt-bearer`.

### If stuff goes wrong

The class is aimed at preventing the cryptic `bad_request` stuff by converting your time to UTC, but always make sure your system time is in sync with NTP and that you use your service account e-mail address (not your ID).

### If all is well...

If everything goes well, you get something like this you can cache for further usage:

	{
	  "access_token" : "1/8xbJqaOZXSUZbHLl5EOtu1pxz3fmmetKx9W8CV4t79M",
	  "token_type" : "Bearer",
	  "expires_in" : 3600
	}

After the access token is expired, you can simply rebuild the JWT and request a new access token.

## Authenticate with the access token

Put this header on each Google API request:

	GET https://www.googleapis.com/calendar/v3/users/me/calendarlist HTTP/1.1
	Authorization: Bearer 1/8xbJqaOZXSUZbHLl5EOtu1pxz3fmmetKx9W8CV4t79M
