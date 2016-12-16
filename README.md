Genesis PHP
===========

Overview
--------

Client Library for processing payments through Genesis Payment Processing Gateway. Its highly recommended to checkout "Genesis Payment Gateway API Documentation" first, in order to get an overview of Genesis's Payment Gateway API and functionality.

Requirements
------------

* PHP version 5.3.2 or newer
* PHP Extensions:
    * [BCMath](https://php.net/bcmath)
    * [CURL](https://php.net/curl) (required, only if you use the curl network interface)
    * [Filter](https://php.net/filter)
    * [Hash](https://php.net/hash)
    * [XMLReader](https://php.net/xmlreader)
    * [XMLWriter](https://php.net/xmlwriter)
* Composer (optional)

Note: Most of the extension are part of PHP and enabled by default, however some distributions are using custom configuration that might have some of them removed/disabled.

Installation
------------

* clone / [download](https://github.com/GenesisGateway/genesis_php/archive/master.zip) this repo

```sh
git clone http://github.com/GenesisGateway/genesis_php genesis_php && cd genesis_php
```

Getting Started
------------------

```php
<?php
require 'vendor/autoload.php';

// Use the Genesis Namespace
use \Genesis;

// Load the pre-configured ini file...
Config::loadSettings('/path/to/config.ini');

// ...OR, optionally, you can set the credentials manually
Config::setEndpoint('<set_your_endpoint>');
Config::setUsername('<enter_your_username>');
Config::setPassword('<enter_your_password>');
Config::setToken('<enter_your_token>');

// Create a new Genesis instance with desired API request
$genesis = new \Genesis\Genesis('Financial\Cards\Authorize');

// Set request parameters
$genesis
    ->request()
        ->setTransactionId('43671')
        ->setUsage('40208 concert tickets')
        ->setRemoteIp('245.253.2.12')
        ->setAmount('50')
        ->setCurrency('USD')
        // Customer Details
        ->setCustomerEmail('emil@example.com')
        ->setCustomerPhone('1987987987987')
        // Credit Card Details
        ->setCardHolder('Emil Example')
        ->setCardNumber('4200000000000000')
        ->setExpirationMonth('01')
        ->setExpirationYear('2020')
        ->setCvv('123')
        // Billing/Invoice Details
        ->setBillingFirstName('Travis')
        ->setBillingLastName('Pastrana')
        ->setBillingAddress1('Muster Str. 12')
        ->setBillingZipCode('10178')
        ->setBillingCity('Los Angeles')
        ->setBillingState('CA')
        ->setBillingCountry('US');

try {
    // Send the request
    $genesis->execute();

    // Successfully completed the transaction - display the gateway unique id
    echo $genesis->response()->getResponseObject()->unique_id;
}
// Log/handle API errors
// Example: Declined transaction, Invalid data, Invalid configuration
catch (\Genesis\Exceptions\ErrorAPI $api) {
    echo $genesis->response()->getResponseObject()->technical_message;
}
// Log/handle invalid parameters
// Example: Empty (required) parameter
catch (\Genesis\Exceptions\ErrorParameter $parameter) {
    error_log($parameter->getMessage());
}
// Log/handle network (transport) errors
// Example: SSL verification errors, Timeouts
catch (\Genesis\Exceptions\ErrorNetwork $network) {
    error_log($network->getMessage());
}

?>
```

Note: the file ```vendor/autoload.php``` is located inside the directory where you cloned the repo and it is auto-generated by [Composer]. If the file is missing, just run ```php composer.phar update``` inside the root directory


Notifications
-------------

When using an Asynchronous workflow, you need to parse the incoming extension in order to ensure its authenticity and verify it against Genesis server.

Example:

```php
<?php
require 'vendor/autoload.php';

// Use the Genesis Namespace
use \Genesis;

try {
    $notification = new API\Notification($_POST);

    // Reconciliation is generally optional, but
    // its a recommended practice to ensure
    // that you have the latest information
    $notification->initReconciliation();

    // Application Logic
    // ...
    // for example, process the transaction status
    // $status = $notification->getReconciliationObject()->status;

    // Respond to Genesis
    $notification->renderResponse();
}
catch (\Exception $e) {
    error_log($e->getMessage());
}

?>
```

Endpoints
---------

The current versions supports two separate endpoints: ```E-ComProcessing``` and ```eMerchantPay```

For example:

- You can set the Endpoint to ```E-ComProcessing```, thus all the requests will go to ```E-ComProcessing```s Genesis instance:
```php
\Genesis\Config::setEndpoint('e-comprocessing');
```

- You can set the Endpoint to ```eMerchantPay```, thus all the requests will go to ```eMerchantPay```s Genesis instance:
```php
\Genesis\Config::setEndpoint('emerchantpay');
```

Request types
-------------

You can use the following request types to initialize the Genesis client:

```text
// Generic transaction operations
Financial\Capture
Financial\Refund
Financial\Void

// Alternative Payment Methods transactions
Financial\Alternatives\ABNiDEAL
Financial\Alternatives\CashU
Financial\Alternatives\Paysafecard
Financial\Alternatives\POLi
Financial\Alternatives\PPRO
Financial\Alternatives\Sofort

// PayByVouchers transactions
Financial\PayByVouchers\oBeP
Financial\PayByVouchers\Sale

// Credit Cards transactions
Financial\Cards\Authorize
Financial\Cards\Authorize3D
Financial\Cards\Credit
Financial\Cards\Payout
Financial\Cards\Sale
Financial\Cards\Sale3D
Financial\Cards\Recurring\InitRecurringSale
Financial\Cards\Recurring\InitRecurringSale3D
Financial\Cards\Recurring\RecurringSale

// Electronic Wallets transactions
Financial\Wallets\eZeeWallet
Financial\Wallets\Neteller
Financial\Wallets\WebMoney

// Generic (Non-Financial) requests
NonFinancial\AVS
NonFinancial\AccountVerification
NonFinancial\Blacklist

// Chargeback information request
NonFinancial\Fraud\Chargeback\DateRange
NonFinancial\Fraud\Chargeback\Transaction

// SAFE/TC40 Report
NonFinancial\Fraud\Reports\DateRange
NonFinancial\Fraud\Reports\Transaction

// Retrieval request
NonFinancial\Fraud\Retrieval\DateRange
NonFinancial\Fraud\Retrieval\Transaction

// Reconcile requests
NonFinancial\Reconcile\DateRange
NonFinancial\Reconcile\Transaction

// Get ABN iDEAL supported banks
NonFinancial\Retrieve\AbniDealBanks

// Web Payment Form (Checkout) requests
WPF\Create
WPF\Reconcile
```


More information about each one of the request types can be found in the Genesis API Documentation and the Wiki

Running Specs
--------------

The following step are optional, however its recommended to run specs at least once, in order to ensure that everything is working as intended on your setup

* install [Composer] (if you don't have it already)
```sh
curl -sS https://getcomposer.org/installer | php
```

* fetch all required packages
```sh
php composer.phar install
```

* run phpspec
```sh
php vendor/bin/phpspec run
```

[Composer]: https://getcomposer.org/
