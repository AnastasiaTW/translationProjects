


# [Xsolla PHP SDK](#php_sdk)





## [Overview](#overview)

Xsolla PHP SDK is an open source library for interacting with [Xsolla API](https://developers.xsolla.com/api_v2.html). [Here](https://github.com/xsolla/xsolla-sdk-php) you can find the link to this project on Github.





## [Features](#features)

1.  Full customization of Payment UI with the help of different methods of getting token.
2.  Client for all API methods, making your integration easy and convenient. You can use it for setting up and updating virtual currency, items and subscription plans, for managing the users balance, for checking the finance information with the help of Report API and so on.
3.  Convenient webhook server:
    1.  To start you need only one callback function.
    2.  All security checking already implemented: signature authentication and IP whitelisting.
    3.  Full customization of notification processing logic, if standard server class doesn't suit you.
4.  SDK is built on Guzzle v3, and utilizes many of its features, including persistent connections, parallel requests, events and plugins (via Symfony2 EventDispatcher), service descriptions, over-the-wire logging, caching, flexible batching, and request retrying with truncated exponential back off.





## [Getting started](#getting_started)

Please register your [Publisher Account](https://publisher.xsolla.com/signup) and create the project. In order to use the PHP SDK Library you'll need:

1.  MERCHANT_ID
2.  API_KEY
3.  PROJECT_ID
4.  PROJECT_KEY

You can obtain these parameters using the information in your [Company Profile](https://publisher.xsolla.com/company) and [Project Settings](https://publisher.xsolla.com/projects).





## [System Requirements](#system_requirements)

1.  PHP 5.3.9+
2.  The following PHP extensions are required:
    1.  curl
    2.  json





## [Installing](#installing)

The recommended way to install Xsolla SDK for PHP is through Composer.

    $ cd /path/to/your/project
    $ composer require xsolla/xsolla-sdk-php

Please visit our [Github project site](https://github.com/xsolla/xsolla-sdk-php#installation) for another ways of installing.





## [Usage](#usage)

##### [Get Token](#usage_get-token)

To integrate Payment UI into your game you should obtain an access token. An access token is a string that identifies game, user and purchase parameters.

There are number of ways for getting token. The easiest one is to use the _createCommonPaymentUIToken_ method, you will need to pass only ID of project in Xsolla system and ID of the user in your game:

    use Xsolla\SDK\API\XsollaClient;

    $client = XsollaClient::factory(array(
        'merchant_id' => MERCHANT_ID,
        'api_key' => API_KEY
    ));
    $paymentUIToken = $client->createCommonPaymentUIToken(PROJECT_ID, USER_ID,
    $sandboxMode = true);

You can pass more parameters in JSON for token:

    <?php

    use Xsolla\SDK\API\XsollaClient;
    use Xsolla\SDK\API\PaymentUI\TokenRequest;

    $tokenRequest = new TokenRequest($projectId, $userId);
    $tokenRequest->setUserEmail('email@example.com')
        ->setExternalPaymentId('12345')
        ->setSandboxMode(true)
        ->setUserName('USER_NAME')
        ->setCustomParameters(array('key1' => 'value1', 'key2' => 'value2'));

    $xsollaClient = XsollaClient::factory(array(
        'merchant_id' => MERCHANT_ID,
        'api_key' => API_KEY
    ));
    $token = $xsollaClient->createPaymentUITokenFromRequest($tokenRequest);

If you want to use some custom parameters for opening Payment UI (for example, "settings.ui.theme"), you can use this example:

    <?php

    use Xsolla\SDK\API\XsollaClient;

    $tokenContent = array (
        'user' =>
            array (
                'id' =>
                    array (
                        'value' => '1234567',
                        'hidden' => true,
                    )
            ),
        'settings' =>
            array (
                'project_id' => 14004,
                'ui' =>
                    array(
                        'theme' => 'dark'
                    )
            )
    );

    $xsollaClient = XsollaClient::factory(array(
        'merchant_id' => MERCHANT_ID,
        'api_key' => API_KEY
    ));
    $response = $xsollaClient->CreatePaymentUIToken(array('request' => $tokenContent));
    $token = $response['token'];

##### [Integrate Payment UI (Pay Station)](#usage_integrate-payment-ui-pay-station)

You can use the following code to add the payment UI on your page:

    <html>
    <head lang="en">
        <meta charset="UTF-8">
    </head>
    <body>
        <button data-xpaystation-widget-open>Buy Credits</button>

        <?php \Xsolla\SDK\API\PaymentUI\PaymentUIScriptRenderer::send($paymentUIToken, $isSandbox = true); ?>
    </body>
    </html>

For more information and examples about Payment UI integration please follow the [link](http://developers.xsolla.com/api_v2.html#paystation_ui).

##### [Receive Webhooks](#usage_receive-webhooks)

There is a build in server class to help you to handle the webhooks.

    <?php

    use Xsolla\SDK\Webhook\WebhookServer;
    use Xsolla\SDK\Webhook\Message\Message;
    use Xsolla\SDK\Exception\Webhook\XsollaWebhookException;

    $callback = function (Message $message) {
        switch ($message->getNotificationType()) {
            case Message::USER_VALIDATION:
                /** @var Xsolla\SDK\Webhook\Message\UserValidationMessage $message */
                // TODO if user not found, you should throw Xsolla\SDK\Exception\Webhook\InvalidUserException
                break;
            case Message::PAYMENT:
                /** @var Xsolla\SDK\Webhook\Message\PaymentMessage $message */
                // TODO if the payment delivery fails for some reason, you should throw Xsolla\SDK\Exception\Webhook\XsollaWebhookException
                break;
            case Message::REFUND:
                /** @var Xsolla\SDK\Webhook\Message\RefundMessage $message */
                // TODO if you cannot handle the refund, you should throw Xsolla\SDK\Exception\Webhook\XsollaWebhookException
                break;
            default:
                throw new XsollaWebhookException('Notification type not implemented');
        }
    };

    $webhookServer = WebhookServer::create($callback, PROJECT_KEY);
    $webhookServer->start();

If standard server class does not suit you, you can create your own:

    <?php

    use Xsolla\SDK\Webhook\WebhookRequest;
    use Xsolla\SDK\Webhook\Message\Message;
    use Xsolla\SDK\Webhook\WebhookResponse;
    use Xsolla\SDK\Exception\Webhook\XsollaWebhookException;

    $httpHeaders = array();//TODO fetch HTTP request headers from $_SERVER or apache_request_headers()
    $httpRequestBody = file_get_contents('php://input');
    $clientIPv4 = $_SERVER['REMOTE_ADDR'];
    $request = new WebhookRequest($httpHeaders, $httpRequestBody, $clientIPv4);
    //or $request = WebhookRequest::fromGlobals();

    $this->webhookAuthenticator->authenticate($request, $authenticateClientIp = true);// throws Xsolla\SDK\Exception\Webhook\XsollaWebhookException

    $requestArray = $request->toArray();

    $message = Message::fromArray($requestArray);
    switch ($message->getNotificationType()) {
        case Message::USER_VALIDATION:
            /** @var Xsolla\SDK\Webhook\Message\UserValidationMessage $message */
            $userArray = $message->getUser();
            $userId = $message->getUserId();
            $messageArray = $message->toArray();
            // TODO if user not found, you should throw Xsolla\SDK\Exception\Webhook\InvalidUserException
            break;
        case Message::PAYMENT:
            /** @var Xsolla\SDK\Webhook\Message\PaymentMessage $message */
            $userArray = $message->getUser();
            $paymentArray = $message->getTransaction();
            $paymentId = $message->getPaymentId();
            $externalPaymentId = $message->getExternalPaymentId();
            $paymentDetailsArray = $message->getPaymentDetails();
            $customParametersArray = $message->getCustomParameters();
            $isDryRun = $message->isDryRun();
            $messageArray = $message->toArray();
            // TODO if the payment delivery fails for some reason, you should throw Xsolla\SDK\Exception\Webhook\XsollaWebhookException
            break;
        case Message::REFUND:
            /** @var Xsolla\SDK\Webhook\Message\RefundMessage $message */
            $userArray = $message->getUser();
            $paymentArray = $message->getTransaction();
            $paymentId = $message->getPaymentId();
            $externalPaymentId = $message->getExternalPaymentId();
            $paymentDetailsArray = $message->getPaymentDetails();
            $customParametersArray = $message->getCustomParameters();
            $isDryRun = $message->isDryRun();
            $refundArray = $message->getRefundDetails();
            $messageArray = $message->toArray();
            // TODO if you cannot handle the refund, you should throw Xsolla\SDK\Exception\Webhook\XsollaWebhookException
            break;
        default:
            throw new XsollaWebhookException('Notification type not implemented');
    }

Once you've finished the handling of notifications on your server, please set up the URL that will receive all webhook notifications on the Settings page for your project.

After passing the tests in Testing tab, you're ready to go live. Please don't forget to remove the sandbox parameters from the code, if you have used them.





## [Troubleshooting](#php_troubleshooting)

Here you can find some tips for handling and preventing the most frequently encountered errors returned by the Xsolla PHP SDK.

##### [[curl] 77: error setting certificate verify locations: CAfile](#php_troubleshooting_[curl]-77:error-setting-certificate-verify-locations:-cafile)

You may see this kind of error when you send the request to our server using Xsolla PHP SDK, for example when getting token.

By default we enable SSL certificate verification and use the default CA bundle provided by operating system. However not all system's have a known CA bundle on disk. For example, Windows and OS X do not have a single common location for CA bundles. There are several ways, how this problem can be resolved.

1) **Development**

You can disable the certificate verification, when you're in development mode. Please mention, that you will need additional testing of this issue on Production environment.

    use Xsolla\SDK\API\XsollaClient;

    $client = XsollaClient::factory(array(
        'merchant_id' => MERCHANT_ID,
        'api_key' => API_KEY
    ));
    $client->setDefaultOption('ssl.certificate_authority', false);

2) **Production**

More secure and reliable way is to provide the correct CA bundle. You can specify the CA path to the file using the following code:

    use Xsolla\SDK\API\XsollaClient;

    $client = XsollaClient::factory(array(
        'merchant_id' => MERCHANT_ID,
        'api_key' => API_KEY
    ));
    $сlient->setDefaultOption('ssl.certificate_authority', '/path/to/file');

In Windows this file can be located in the following paths:

1.  C:\windows\system32\curl-ca-bundle.crt
2.  C:\windows\curl-ca-bundle.crt

Please check the existence of these files, and set the appropriate CA path.

If you don't have this certificate located in the mentioned places, you can try to use the certificate provided by mozilla, which can be downloaded [here](https://raw.githubusercontent.com/bagder/ca-bundle/master/ca-bundle.crt) (provided by the maintainer of cURL).

In some versions of php for Windows there is a problem with programmatic configuration of certificate paths. In order to solve this problem, download the file [https://curl.haxx.se/ca/cacert.pem](https://curl.haxx.se/ca/cacert.pem) and specify the path to this file directly in php.ini: curl.cainfo=c:/cacert.pem.

* * *

##### ["INVALID_SIGNATURE" error code with message "Authorization header not found in Xsolla webhook request"](#php_troubleshooting_invalid_signature-error-code-with-message-authorization-header-not-found-in-xsolla-webhook-request)

php-cgi under Apache does not pass HTTP Basic user/pass to PHP by default.

In order to get this working you need to add the following line to .htaccess or httpd.conf Apache file:

    RewriteEngine On
    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

* * *

##### ["INVALID_CLIENT_IP" error code in your Webhook server](#php_troubleshooting_invalid_client_ip-error-code-in-your-webhook-server)

By default Xsolla PHP SDK is checking the IPs from which the webhook was send for security reasons. The error code "INVALID_CLIENT_IP" can be returned if you test your webhook server from a localhost in development environment, or your application server works behind some sort of proxy - like a load balancer - on production. If you are behind a proxy, you should manually whitelist your proxy.

If you are in development environment, you can disable the IP checking using the following code:

    use Xsolla\SDK\Webhook\WebhookServer;

    $webhookServer = WebhookServer::create($callback, PROJECT_KEY);
    $webhookServer->start($webhookRequest = null, $authenticateClientIp = false);

More secure and reliable way is to set your reverse proxy IP address to webhook server:

    use Xsolla\SDK\Webhook\WebhookServer;
    use Symfony\Component\HttpFoundation\Request;

    $request = Request::createFromGlobals();
    $request->setTrustedProxies(array('YOUR_PROXY_SERVER_IP_OR_CIDR'));

    $webhookServer = WebhookServer::create($callback, PROJECT_KEY);
    $webhookServer->start();

More information is available in [Symfony Documentation](http://symfony.com/doc/current/components/http_foundation/trusting_proxies.html).





# [Xsolla Android SDK](#android_sdk)





## [Overview](#overview)

Xsolla created Android Client SDK for accepting payments from your application. You can check its work by downloading [this apk](http://livedemo.xsolla.com/sdk/android/2.2.2/app.apk).

Before start, please choose one of the modules listed [here](https://developers.xsolla.com/#getting-started), implement the Webhook handling, create an [access token](https://developers.xsolla.com/api_v2.html#token).





## [System Requirements](#system_requirements)

1.  Minimum required Android OS version: 4.0
2.  Internet Connection is essential for the Xsolla Android SDK





## [Download](#download)

  Download via Jcentral:  

     <dependency> 
    <groupId>com.xsolla.android</groupId> 
    <artifactId>xsollasdk</artifactId> 
    <version>2.2.2</version>
     </dependency>

 or Gradle:

 `compile 'com.xsolla.android:xsollasdk:2.2.2'` 





## [Installing](#installing)

You can add our Xsolla Android SDK in Android Studio. Please follow this steps:

1.  import module - File > Import Module > xsollasdk
2.  add dependency - File > Project Structure > Your App Module > Dependency > + > Module Dependency > xsollasdk
3.  add to your build gradlew in android section the following:

         packagingOptions {
            exclude 'META-INF/LICENSE'
            exclude 'META-INF/NOTICE'
         }

[Here](https://github.com/xsolla/xsolla-sdk-android) you can find the link to this project on Github.





## [Make a Payment](#make_payment)

For the proper work of the SDK, please make sure that you have an access token. More information about getting token is available [here](https://developers.xsolla.com/api_v2.html#token).

##### [Xsolla UI](#make_payment_xsolla-ui)

Let's take as an example the simple application that has a payment button, when clicked our Shop UI is opened.

    public class MainActivity extends Activity {

     public void openShop(String token, boolean isTestPayment) {
        XsollaSDK.createPaymentForm(this, token, isTestPayment);
     }

     protected void onActivityResult(int requestCode, int resultCode, Intent data) {
       switch (requestCode){
         case XsollaActivity.REQUEST_CODE:
           if(data != null) {
             long objectId = data.getExtras().getLong(XsollaActivity.EXTRA_OBJECT_ID);
             if (resultCode == RESULT_OK) {
               XStatus statusData = (XStatus) XsollaObject.getRegisteredObject(objectId);
               Toast.makeText(this, statusData.toString(), Toast.LENGTH_LONG).show();
             } else {
               XError error = (XError) XsollaObject.getRegisteredObject(objectId);
               Toast.makeText(this, error.toString(), Toast.LENGTH_LONG).show();
             }
           }
       }
     }
    }

Once the payment has been processed, your application can get a result in OnActivityResult. Also we will send a webhook on your server, even if the application has been closed.





# [Xsolla Unity SDK](#unity_sdk)





## [Overview](#overview)

Xsolla created Unity SDK for accepting payments in desktop, web or mobile applications.

**Download the latest release of Xsolla Unity SDK from [GitHub.](https://github.com/xsolla/xsolla-unity-sdk)**





## [System Requirements](#system_requirements)

Xsolla Unity SDK works with Unity 5.0 and above.





## [Integration](#integration)

Main Features:

*   Saved payment methods;

*   Purchase of virtual currency;

*   Purchase of virtual items;

*   Purchase of subscriptions;

*   Promotions;

*   Redeem coupon;

*   User's payment history.

If you would like to accept payments through Xsolla's payment UI, follow these steps:

1.  Set up webhook processing:[http://developers.xsolla.com/api_v2.html#webhooks](http://developers.xsolla.com/api_v2.html#webhooks)
2.  Create an Xsolla Access Token to conduct payments with maximum security. You can find documentation on creating a token here: [http://developers.xsolla.com/api_v2.html#payment_ui](http://developers.xsolla.com/api_v2.html#payment_ui)
3.  Add XsollaSDK script to any object or use prefab from 'Resources -> Prefabs' folder;
4.  Call _XsollaSDK(instance)_ to generate ready to use payment form.
5.  Call the method _CreatePaymentForm(token, actionOk(XsollaStatusData), actionError(XsollaError), actionMore(string))_

<table class="different-table">

<thead>

<tr>

<th>Parameter</th>

<th>Description</th>

</tr>

</thead>

<tbody>

<tr>

<td>token</td>

<td>Your purchase token received using [Getting token method](https://developers.xsolla.com/api_v2.html#token)</td>

</tr>

<tr>

<td>actionOk</td>

<td>Is called when the payment process is completed. Insert your payment accepting function here.</td>

</tr>

<tr>

<td>actionError</td>

<td>Is called when the payment process is canceled or failed for any reason. Insert your event handling function here.</td>

</tr>

</tbody>

</table>

1.  You can also use _XsollaSDK.InitPaystation(string token)_ to launch the payment UI in the native browser.

If you want to have your own payment UI, you should write own class which extends XsollaPaystation Class.

As example you can use XsollaPaystationController.

SDK Response Objects:

    public class XsollaResult {
        public string invoice{ get; set;}
        public Status status{ get; set;}
        public Dictionary<string, object> purchases;
    }





## [Try it!](#try_it)

Please take a look at our [demo](https://livedemo.xsolla.com/sdk/unity/).

We also have test scenes in "XsollaUnitySDK" -> "Resources" -> "_Scenes" folder:

1.  XsollaFarmFreshScene - emulates item shop.
2.  XollaTokenTestScene - you can test your token here.



</div>