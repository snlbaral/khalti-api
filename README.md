# khalti-api-laravel

[![Latest Stable Version](https://poser.pugx.org/snlbaral/khalti-api/v)](//packagist.org/packages/snlbaral/khalti-api) [![Total Downloads](https://poser.pugx.org/snlbaral/khalti-api/downloads)](//packagist.org/packages/snlbaral/khalti-api) [![Latest Unstable Version](https://poser.pugx.org/snlbaral/khalti-api/v/unstable)](//packagist.org/packages/snlbaral/khalti-api) [![License](https://poser.pugx.org/snlbaral/khalti-api/license)](//packagist.org/packages/snlbaral/khalti-api)


This is an open source library that allows PHP/Laravel applications to integrate Khalti Payment Gateway.

Requirements
------------

Using this library for PHP requires the following:

* [Composer][composer] or a manual install of the dependencies mentioned in
  `composer.json`.


Installation
------------

The recommended way to install it PHP is to install it using
[Composer][composer]:

```sh
composer require snlbaral/khalti-api
```


Quick start
-----------

Create a merchant account in Khalti, Obtain the keys. And use test_public_key and test_secret_key at first. After the first successful test, you'll get live api keys.
**Warning: *Secrets Keys* are similar to passwords or private keys by allowing an application to identify as yours: therefore, *Secret Key* should be kept private.**

### Step 1: create your payment html page

#### `index.blade.php`

```html
<html>
<head>
    <script src="https://khalti.s3.ap-south-1.amazonaws.com/KPG/dist/2020.12.17.0.0.0/khalti-checkout.iffe.js"></script>
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
</head>
<body>

    <!-- Place this where you need payment button -->
    <button id="payment-button">Pay with Khalti</button>
    <!-- Place this where you need payment button -->
    <!-- Paste this code anywhere in you body tag -->



    <script>
        var config = {
            // replace the publicKey with yours
            "publicKey": "test_public_key_YOUR_PUBLIC_KEY",
            "productIdentity": "1234567890", //Product ID
            "productName": "Dragon", //Product Name
            "productUrl": "http://gameofthrones.wikia.com/wiki/Dragons", //Product URL
            "paymentPreference": [
                "KHALTI",
                "EBANKING",
                "MOBILE_BANKING",
                "CONNECT_IPS",
                "SCT",
                ],
            "eventHandler": {
                onSuccess (payload) {
                    // hit merchant api for initiating verfication
                    //console.log(payload);
                    if(payload.status==200) {
                        $.ajaxSetup({
                            headers: {
                                'X-CSRF-Token': '{{csrf_token()}}'
                            }
                        });
                        $.ajax({
                            url: "{{ route('verification') }}", //Your backend route url, replace this with the route you'll be creating later
                            data: payload,
                            method: 'POST',
                            success: function(data) {
                                console.log('Payment is succcessfull');
                                console.log(data);
                            },
                            error: function(err) {
                                console.log(err.response);
                            },
                        });                        
                    }
                },
                onError (error) {
                    console.log(error);
                },
                onClose () {
                    console.log('widget is closing');
                }
            }
        };

        var checkout = new KhaltiCheckout(config);
        var btn = document.getElementById("payment-button");
        btn.onclick = function () {
            // minimum transaction amount must be 10, i.e 1000 in paisa.
            checkout.show({amount: 3000});
        }
    </script>
    <!-- Paste this code anywhere in you body tag -->
</body>
</html>
```


### Step 2: create route for payment verification
create route for payment verification and use this route in above ajax url

```web.php
Route::post('/verification', [App\Http\Controllers\PaymentController::class, 'verification'])->name('verification');
```



-----------
Usages
----------

**Init**

```php
use Snlbaral\Khalti\Khalti;
$khalti = new Khalti();
```

**Methods**
```php
/**
 *
 * @param string $secret your khalti merchant secret key
 * @param string $token your khalti api payment transaction token
 * @param string $idx your khalti api payment transaction idx
 * @param int $amount khalti payment transaction amount
 */

//Payment Verification
$response = $khalti->verifyPayment($secret,$token,$amount);

//List Transactions
$response = $khalti->listTransactions($secret);

//Get Transaction
$response = $khalti->getTransaction($secret,$idx);

//Transaction Status
$response = $khalti->transactionStatus($secret,$token,$amount);

```



----------
Example
----------

**PaymentController.php**

```php
use Snlbaral\Khalti\Khalti;


public function verification(Request $request) {
    $secret = "test_secret_key_YOUR_SECRET_KEY";
    $token = $reqeust->token;
    $amount = $reqeust->amount;
    $khalti = new Khalti();
    $response = $khalti->verifyPayment($secret,$token,$amount);

    //Response Array from $response
        // status_code: 200
        // data: 
            // amount: 3000
            // cashback: 0
            // created_on: "2021-05-03T17:41:16.436643+05:45"
            // fee_amount: 90
            // idx: "pBAKtpzJaQWdfdfsRN7WNtTXpcH"
            // merchant:
            //     email: "user@gmail.com"
            //     idx: "QA3rsGoGgtQHKGDfvrL9NU"
            //     mobile: "user@gmail.com"
            //     name: "Company Name"
            //     reference: null
            //     refunded: false
            //     remarks: ""
            // state:
            //     idx: "DhvMj9hdRufLqkP8ZY4d8gdfdfs"
            //     name: "Completed"
            //     template: "is complete"
            //     token: "WgnXkdfsfCNpGMcoEcojLmxCmM"
            // type:
            //     idx: "LFVwXcpfdfs3wQENxGPZWdELa"
            //     name: "Ebanking payment"
            // user:
            //     email: ""
            //     idx: "xeoLUUnskdfsfsfszikFgGLmKWhH7"
            //     mobile: "NA"
            //     name: "Nepal Clearing House"
    //Response Array Ends

     //Store into Database Here//
     // if($response['status_code']==200) {
	    // $amount = $response['data']['amount'];
	    //.
	    //..
    // }
    // //

    return $response;

}
```

License
-------

This library for PHP is licensed under the <a href="https://opensource.org/licenses/BSD-3-Clause">3-Clause
BSD License</a>

Credits
-------

This library for PHP is developed and maintained by Sunil Baral.
