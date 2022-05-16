---
title: Integrate Messenger with Okta for identity management (SSO)
indextype: blueprint
icon: blueprint
image: images/Messenger_Okta_Integration.png
category: 6
summary: |
  This Genesys Cloud Developer Blueprint demonstrates how to integrate Messenger with the identity provider, Okta, in order to use SSO to authenthicate Messenger across multiple domains. This blueprint also includes the steps to set up an Okta account to obtain authorization credentials.
---

  This Genesys Cloud Developer Blueprint demonstrates how to integrate Messenger with the identity provider, Okta, in order to use SSO to authenthicate Messenger across multiple domains. This blueprint also includes the steps to set up an Okta account to obtain authorization credentials.

![Flow diagram](./images/Messenger_Okta_Integration.png "Integrate Messenger with Okta")

## Solution components

**Genesys Cloud** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. In this solution, a Genesys Cloud user account is required for your application to be authorized to integrate with Messenger.

## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud

## Genesys Cloud account

* A Genesys Cloud license. For more information, see [Genesys Cloud pricing](https://www.genesys.com/pricing "Opens the Genesys Cloud pricing page") on the Genesys website.
* (Recommended) The Master Admin role in Genesys Cloud. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.

## Okta Developer Edition account

* An Okta Developer Edition account. For more information, see [the Okta Developer page](https://developer.okta.com/signup/ "Goes to the Okta Developer page on the Okta website").

## Implementation steps

### Set up Okta

1. Log in to your Okta Developer edition account.
2. In the left panel, click **Applications > Applications**. 
3. Click **Create App Integration**.

  ![Okta Applications page](./images/Application.png "Okta Applications page")

4. On the Create a new app integration page, in the Sign-in method section, click **OpenID Connect**.
5. For the application type, select **Web Application** and then click **Next**.
6. On the New Web App Integration** page, in the **App integration name** box, type a name for your app integration.
7. Select the required **Grant type**. 

![Okta Application page](./images/OKTA.png "OKTA Application page")

8. Add the **Sign-in redirect URIs**. 
	* For local use while you are developing, add http://{local_domain_name}. Example: http://localhost:8080/. 
	* For production use, add the URL where Messenger will initialize after login.
9. Add the **Sign-out redirect URIs**.
	* For local use while you are developing, add http://{local_domain_name}. Example: http://localhost:8080/. 
	* For production use, add the appropriate URL.
10. Under **Security**, click **API** > **Trusted Origin** and add your web page origin in the Okta account.
11. In the **Assignment** section, select the option that best suites your organization and click **Save**. This creates the client credentials.
12. To find the Okta URL, navigate to **Security** > **API** and click **Default**. The Okta URL appears in the **Issuer** section.
14. Make a note of these items, which you will need later: 
 	* Okta client credentials 
	* Okta URL  

### Update the integration in Genesys Cloud

1. Log in to your Genesys Cloud account and click **Admin** > **Integrations** > **+Integrations**.

  ![Install an integration](./images/Install_Integration.png "Install an integration")

3. Install the **OpenID Connect Messenger Configuration**.

  ![Install the OpenID Connect Messenger configuration](./images/integration.png "Install the OpenID Connect Messenger configuration")

4. In the **Configuration** section, paste the Discovery Uri: `https://<okta-user-domain>/oauth2/default/.well-known/openid-configuration`.
	
  ![Genesys Cloud Integration page](./images/Integration_properties.png "Genesys Cloud Integration page")

5. In **Credentials** section, click **Configure** and provide the Okta client credentials.
6. Click **Save**.
7. On the main Integrations page, set your integration to **Active**.

  ![Activate your integration](./images/Active.png "Activate your integration")

## Update the Messenger configuration

1. In Genesys Cloud > **Admin**, under **Message**, click **Messenger Configurations**. 
2. Enable Authentication.
3. From the list, select the **OAuth integration** created from [Genesys Cloud Integration](#integration "Goes to Genesys Cloud Integration step").

  ![Genesys Cloud Integration page](./images/Messenger-Okta-configuration.png "Genesys Cloud Integration page")

## Update the Messenger deployment

1. In **Genesys Cloud > Admin > Messenger Deployment**, select the **Configuration** created in the [Messenger Configuration](#messenger-configuration "Goes to Messenger Configuration step")

 ![Genesys Cloud Messenger Deployment](./images/Configuration.png "Genesys Cloud Messenger Deployment")

3. Save the deployment. This creates a single Snippet.
4. Use this single snippet on the website you would like the Messenger to appear.

 ![Genesys Cloud Messenger Deployment](./images/Snippet.png "Genesys Cloud Messenger Deployment")

:::info
 Note: Note the deployment Id and environment from snippet to play with sample authentication app .
:::

## Configure authenticated messenger

Write necessary code to configure messenger for authenticated web messaging in your webpage.

1. To enable OKTA Sign-In experience, using JavaScript there are two approaches.
  A. Including [Okta Auth JavaScript SDK](https://global.oktacdn.com/okta-auth-js/5.2.2/okta-auth-js.min.js 'Okta Auth JavaScript SDK') in your page.
  B. Using OAuth 2.0 Endpoint.

	**A.Okta Sign-In using SDK:**

	- Include the okta-auth-js library in your web page.

	```{"title":"OKTA SDK","language":"html"}
	<script src="https://global.oktacdn.com/okta-auth-js/5.2.2/okta-auth-js.min.js" type="text/javascript"></script>
	```

	- Create an instance of the OktaAuth object to configure the OKTA authorization options.

	```{"title":"OktaAuth Object","language":"JavaScript"}
	const oktaConfig = {
		redirectUri: <signInRedirectUri>,
		postLogoutRedirectUri: <signOutRedirectUri>,
		clientId: <ClientId>,
		issuer: <Okta URL>,
		scopes: ['openid', 'email', 'profile', 'offline_access'],
		pkce: true,
		responseType: 'code',
		maxAge : <MaxAge>
	};
	const authClient = new OktaAuth(oktaConfig)		
	```

	- The OktaAuth object contains the following parameters:
	| Parameter | Description |
	| --------- |----------|
	| redirectUri | Specify the url where the browser should be redirected after signIn. This must be your full page URL which is listed in your Okta application's Sign-in redirect URIs.|
	| postLogoutRedirectUri | Specify the url where the browser should be redirected after signOut. This must be your full page URL which is listed in your Okta application's Sign-out redirect URIs. If not specified, your application's origin (window.location.origin) will be used.|
	| clientId | Specify the clientID which was generated when the Okta app was created. Refer [OKTA Set-Up](#okta-set-up)|
	| issuer | Specify the Okta URL present in your OKTA-Developer account under Security --> API --> default|
	| scopes| Set auth scopes to specify what access privileges are being requested as part of the authorization, if required.|
	| pkce | Default value is true which enables the PKCE OAuth Flow. To use the Implicit Flow or Authorization Code Flow, set pkce to false. *Note : PKCE flow works only with secure domain.* |
	| responseType| Set to **code** indicating that we are using the Authorization Code grant type.|
	| maxAge | Allowable elapsed time, in seconds, since the last time the end user was actively authenticated by Okta.|
	| nonce | A random string value generated by OKTA SDK. Nonce can also be passed as a paramater to OktaAuth object which rewrites the generated nonce value by OKTA SDK.|
	{: class="table-striped table-bordered"}

	- Trigger the signIn action by calling the OKTA SDK's method **signInWithRedirect** with the request parameters. The **originalUri** parameter in options to track the route before the user signIn, and the addtional parameters are mapped to the Authorize options. This could be triggered with the help of link, button, etc.,

	```{"title":"OktaAuth signInWithRedirect method","language":"JavaScript"}
	authClient.signInWithRedirect({
		originalUri: <your current page url here>,
		...oktaConfig
	});
	```

	**B. Okta Sign-In using Endpoint:**

	- Generate **Auth URL** and trigger the login action. This could be triggered with the help of link, button, etc.,

	```{"title":"Auth url Example","language":"html"}
	//Your request URL would look something like this

	authURL = `<DomainURL>client_id=<ClientId>&scope=openid%20email%20profile%20offline_access&response_type=code&redirect_uri=<RedirectURL>&state=eyJiYWNrVG9QYXRoIjoiL3ByaXZhdGUiLCJpc3N1ZXIiOiJva3RhIiwiYnl0ZXMiOiItSEhlWEV3YmNRak5fQWl3a0NkanVDNEZpQ1VPRV81emkzeFlKa1BQaWcwIn0%3D`

	//Your request URL would look something like this if pkce flow is enabled

	authURL = `<DomainURL>client_id=<ClientId>&scope=openid%20email%20profile%20offline_access&response_type=code&redirect_uri=<RedirectURL>&state=eyJiYWNrVG9QYXRoIjoiL3ByaXZhdGUiLCJpc3N1ZXIiOiJva3RhIiwiYnl0ZXMiOiItSEhlWEV3YmNRak5fQWl3a0NkanVDNEZpQ1VPRV81emkzeFlKa1BQaWcwIn0%3D&code_challenge_method=S256&code_challenge=<codeChallenge>`

	//Your request URL would look something like this if optional parameter(nonce & maxAge) is added

	authURL = `<DomainURL>client_id=<ClientId>&scope=openid%20email%20profile%20offline_access&response_type=code&redirect_uri=<RedirectURL>&state=eyJiYWNrVG9QYXRoIjoiL3ByaXZhdGUiLCJpc3N1ZXIiOiJva3RhIiwiYnl0ZXMiOiItSEhlWEV3YmNRak5fQWl3a0NkanVDNEZpQ1VPRV81emkzeFlKa1BQaWcwIn0%3D&nonce=<nonce>&max_age=<maxAge>`
	```

	- The Auth URL contains the following parameters:
	| Parameter | Description |
	| --------- |----------|
	| DomainURL | Authorization server's endpoint(https://{yourOktaDomain}/oauth2/default/v1/authorize)|
	| ClientId  | Specify the clientID which was generated when the Okta app was created. Refer [OKTA Set-Up](#okta-set-up)|
	| RedirectURL| Specify the url where the browser should be redirected after signIn. This must be your full page URL which is listed in your Okta application's Sign-in redirect URIs.|
	| Auth scopes| Set auth scopes to specify what access privileges are being requested as part of the authorization, if required.|
	| response_type| Set to **code** indicating that we are using the Authorization Code grant type|
	| max_age | Allowable elapsed time, in seconds, since the last time the end user was actively authenticated by Okta.|
	| nonce | A random string value preferably an uuid format which is returned in the ID token. |
	| codeChallenge | A string value generated from code verifier to support PKCE flow in OAuth. The code challenge is verified in the access token request. *Note: Code verifier is a random string value between 43 and 128 characters long. Okta uses it to recompute the code_challenge and verify if it matches the original code_challenge in the authorization request.* |
	| codeChallengeMethod | Method used to derive the code challenge for PKCE flow. Valid value: S256. |
		{: class="table-striped table-bordered"}

2. If the user doesn't have an existing session, making this request opens the **OKTA** sign-in page.
3. If they have an existing session, they arrive at the specified redirect_uri along with a code as shown in the code snippet.

```{"title":"Redirect url appended with code and state Example","language":"javascript"}
https://mypureclloud.com/?code=P5I7mdxxdv13_JfXrCSq&state=state-296bc9a0-a2a2-4a57-be1a-d0e2fd9bb601 // Code specifies OKTA authcode
```

4. Page reload takes place when redirection happens from **OKTA**. This initializes [Auth plugin](https://developer.genesys.cloud/api/digital/webmessaging/messengersdk/SDKCommandsEvents#auth-plugin 'Goes to the SDK Commands and Events page') and calls its command [getTokens](https://developer.genesys.cloud/api/digital/webmessaging/messengersdk/SDKCommandsEvents#auth-plugin 'Goes to the SDK Commands and Events page') for Authentication.
5. Split the **OKTA** authcode from the redirect url.
6. Create your own authprovider plugin and register the command [getAuthCode](https://developer.genesys.cloud/api/digital/webmessaging/messengersdk/SDKCommandsEvents#authprovider-plugin 'Goes to the SDK Commands and Events page').

```{"title":"Prepare the AuthProvider plugin","language":"javascript"}
Genesys('registerPlugin', 'AuthProvider', (AuthProvider) => {

  // COMMAND
  // *********
  // getAuthCode

  const urlParams = new URLSearchParams(window.location.search); // Get the authorization response which is added as a query string from the redirect URL
  const authCode = urlParams.has('code') ? urlParams.get('code'); // Get code from the query string

  /* Register Command - mandatory */

  AuthProvider.registerCommand('getAuthCode', (e) => {

  //Messenger will call this command to get the the tokens.

  e.resolve({
      authCode: <authCode>,			// Pass your authCode here
      redirectUri: <your redirect uri>,	// Pass the redirection URI configured in your Authentication provider here
      nonce: <nonce>,				// For Sign-In using SDK approach, pass the random string value stored in session storage. For endpoint approach, generate a random string value
      maxAge: <maxAge>				// Pass elapsed time in seconds and it is an optional parameter
      codeVerifier: <codeVerifier>		// For PKCE flow : If SDK approach is used, get code verifier from session storage. If endpoint approach is used, pass a cryptographically random string which you used to generate codeChallenge
    });
  });
});
```

7. Trigger the signOut action by calling the OKTA SDK's method **signOut**. This method should be called after [Auth.logout](https://developer.genesys.cloud/api/digital/webmessaging/messengersdk/SDKCommandsEvents#auth-logout 'Goes to Auth provider plugin') command. This could be triggered with the help of link, button, etc.,

```{"title":"OktaAuth signOut method","language":"JavaScript"}
AuthProvider.command('Auth.logout').finally(() => {
	authClient.signOut();
});
```

### Run the sample app

You can run the sample app locally or from the blueprint repo.

:::primary
**Note**: Regardless of where you run the sample app from, you need a Genesys Cloud user account in order for it to work. Our sample app uses [Okta Sign-In using SDK](#configure-authenticated-messenger) approach.
:::

To run the sample app from the blueprint repo:

1. Click [here](https://genesyscloudblueprints.github.io/messenger-authentication-okta-integration-blueprint/oauth.html "Goes to the sample app").

2. Enter the environment and deployment id configured With Okta. Click Submit.

3. Enter the client credentials for authenticated web messaging.

:::info
 Refer the source code for sample app [here](https://github.com/GenesysCloudBlueprints/messenger-authentication-okta-integration-blueprint/blob/main/docs/oauth.html "Goes to source code of the sample app")
:::

  For more information, see [Platform API](/api/digital/webmessaging/authenticate "Goes to the Authenticated WebMessaging page in the Genesys Cloud Developer Center").
