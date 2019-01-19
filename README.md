# Laravel Mpesa Package

This package help you in integrating your laravel application with Mpesa daraja APIS. The package eliminate all the hastles and let you concetrate with what is important.

The package will help you integrating the following APIs, available in mpesa daraja

- C2B (consumer to business)
- B2C (business to cunsumers)
- Lipa na mpesa online
- Reversal
- Transaction status
- Account balance

 
## Installation
You can install this awesome package via composer

```
composer require gathuku\laravel_mpesa

```
The package will install automatically to your application. After the package has been istalled in your project run

```
php artisan vendor:publish
```
This will help in publishing mpesa_config in your application config. From the mpesa config file this where you will define if your application is running in sandbox or production. If your application is running on sandbox you  will define `'mpesa_status' => 'sandbox',`
You will continue filling your details from your application in [developers portal ](developers.safaricom.co.ke)

When you publish mpesa config it will come with test credentials which can be used in sandbox.For production you will need to change all this credentials.


## Usage

### Register C2B Urls
This API enables you to register the callback URLs via which you shall receive payment notifications for payments to your paybill/till number. Read more on [official safaricom documentation](https://developer.safaricom.co.ke/docs) or on [ a simplified documentation by Peter Njeru](https://peternjeru.co.ke/safdaraja/ui/)

To register Urls call this method using Mpesa facade from your controller

```php
$registerUrlsResponse=Mpesa::c2bRegisterUrls()
```

Upon successful registering of urls you should get the following response


### C2B API
You can use C2B API to simulate payment from clients and safaricom API

To use C2B you need to pass three parameters to simulate C2B function

1. The amount being simulated
2. Test MSISDN- form API test credentials
3. Bill reference

```php
$simulateResponse=Mpesa::simulateC2B(100, "254708374149", "Testing");

```
Upon successful simulation you will receive this responce

```json
       {
            "ConversationID": "AG_20190115_00007fc37fc3db6e9562",
            "OriginatorCoversationID": "10028-4198443-1",
            "ResponseDescription": "Accept the service request successfully."
        }

```
For you to receive payment confirmation you need to register a route in routes/api.php according to the confirmation you registered.Then you can use laravel log facade to log the request content.
For example the following urls you can get validation and confirmation requests using the code below
```
//Validation url
https://b2d7e6a4.ngrok.io/api/validate?key=ertyuiowwws

//Confirmation url
https://b2d7e6a4.ngrok.io/api/confirm?key=ertyuiowwws
```
```php
Route::post('validate/confirm', function(Request $request){
    Log::info($request->getContent());
});
```

You will also receive a  confirmation from your the confirmation you registered
```json
       {
            "TransactionType": "Pay Bill",
            "TransID": "NAF81H7Y72",
            "TransTime": "20190115234802",
            "TransAmount": "100.00",
            "BusinessShortCode": "601380",
            "BillRefNumber": "Testing",
            "InvoiceNumber": "",
            "OrgAccountBalance": "900.00",
            "ThirdPartyTransID": "",
            "MSISDN": "254708374149",
            "FirstName": "John", 
            "MiddleName": "J.",
            "LastName": "Doe"
        }
```


### B2C API
To use this API you need to call `b2c()` on `Mpesa` facade. This function accept the following parameters
1. `Amount` - Amount of money sent to a consumer
2. `MSISDN` - Phone number of consumer
3. `command_id` -This specifies the type of transaction.There are three allowed values on the API:      `SalaryPayment`, `BusinessPayment` or `PromotionPayment`
4. `Remarks` - small decription on the payment being made.

```php
$b2cResponse=Mpesa::b2c(100,'254708374149','PromotionPayment','testing');
```
Upon success you should receive below response
```json

       {
            "ConversationID": "AG_20190117_00004636fb3ac56655df",
            "OriginatorConversationID": "17503-13504109-1",
            "ResponseCode":"0",
            "ResponseDescription": "Accept the service request successfully."
        }
```
After a successdull transaction you will get get a callback via `b2c_result` you specifies in `mpesa  config`, A sample success callback is below.
```json
 {
     "Result":{"ResultType":0,"ResultCode":0,"ResultDesc":"The service request is processed successfully.","OriginatorConversationID":"17492-14303897-1",
     "ConversationID":"AG_20190119_00005912734ad99a27e8",
     "TransactionID":"NAJ51H7YJ7",
     "ResultParameters":{"ResultParameter":[{
     "Key":"TransactionReceipt","Value":"NAJ51H7YJ7"},
     {"Key":"TransactionAmount","Value":100},
     {"Key":"B2CChargesPaidAccountAvailableFunds","Value":-495.00},
     {"Key":"B2CRecipientIsRegisteredCustomer","Value":"Y"},
     {"Key":"TransactionCompletedDateTime","Value":"19.01.2019 10:46:22"},
     {"Key":"ReceiverPartyPublicName","Value":"254708374149 - John Doe"},{"Key":"B2CWorkingAccountAvailableFunds","Value":600000.00},
     {"Key":"B2CUtilityAccountAvailableFunds","Value":767.00}]},
     "ReferenceData":{"ReferenceItem":{"Key":"QueueTimeoutURL","Value":"https:\/\/internalsandbox.safaricom.co.ke\/mpesa\/b2cresults\/v1\/submit"}}}}  
```

### Mpesa Express
To run this api call `express` function on `Mpesa` Facade. The function requires the following parameters
1. `amount` - Amount of money in the transaction
2. `msisdn` - Phone Number of debit party 
3. `AccountReference` - Payment reference
4. `TransactionDesc`  - a small description on the payment

```php
$expressResponse=Mpesa::express();
```


Upon success `$expressREsponse` will return below response
```json
{
  "MerchantRequestID": "10029-6178310-1"
  "CheckoutRequestID": "ws_CO_DMZ_291417540_19012019145720246"
  "ResponseCode": "0"
  "ResponseDescription": "Success. Request accepted for processing"
  "CustomerMessage": "Success. Request accepted for processing"
}
```

