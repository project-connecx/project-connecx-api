Connecx Unified Payments
==

# GENERAL OVERVIEW

We **require all merchants** to complete a full test cycle in our sandbox environment using the information provided below before any production information can be set up and released. Once testing has been completed, contact us to enable your live account. 


Throughout this document, placeholders are used to represent variable elements.  These placeholders are enclosed with “{ }”.  Following are the explanations for these:

1. `{host}` – this refers to the host portion of a URL. Contact support team for url details
2. `{merchantId}` - A merchant ID and API key will be issued to you upon creation of your account
Your merchant ID and API key in the test environment will be different from that of your production account
3. `{hashed token}` - This is a Hex-encoded SHA-256 hash of a string consisting of concatenated values of select attributes depending on the API.
The hashing requirements are explained in a later section of this document.

# PAYMENT APIs
## API AUTHENTICATION

Payment API calls will require a `{hashed token}` to be passed as part of the request header. Talk to the support team to check how to create the hash. 

## PAY IN
Set the hash value (from API Authentication) in the request header with a header name of **`X-paymesh-authz`**.

| Description | Accepting payment from one of the supported channels|
|:--- |:--- |
| HTTP Method |POST|
| Request Headers| X-paymesh-authz={hashed token} Content-Type=application/json|
|URL | https://{host}/mxapi/{merchantId}/bdpayin |
| Request Body | In JSON format. Following are the attributes for the request body.|


**Request Body**

|Attribute Name |Required |Description|
|:-----|:-----|:--- |
| channelCode|Y|  Direct payment channel selection code. Channel Code(s) would be assigned to your solution.
accountId |Y|Assigned account ID which is a numeric, 10 digit identifier|
requestSalt |Y|Any string of any length. Should be different for every request. Suggest using current time in milliseconds. 
|txnAmount|Y|The amount of the payment. Numeric, 2 decimal places (e.g. 10.00).|
|txnCurrency|Y|Currency of the payment (ex. CNY). 
|paymentRef|Y|Your own transaction identifier
|paymentDesc|N|A description of the transaction
|dupCheck|N|Valid values are “true” or “false”. If set to true, we will check if a transaction with the same paymentRef has already been submitted. If a duplicate is found, an error will be returned. If set to false, no duplicate checking will be performed. The default is false.
|merchantTraceData|N|Custom merchant field to pass data which will be echoed back in the result, the redirect (for 3d) and in queries.|
|resultUrl|Y|Redirect Url if needed
|callbackUrl|Y|Server notification callback Url
|payInfo|Y |Conditional - depends on channel requirement. We recommend calling Channel Specification Query API in order to determine the required data object for the specific channel. This is an object with key-value pair attributes. The attributes vary depending on the channel. 
|endUserInfo|Y|User Information Object (see below)|

**`endUserInfo`  Object**

|Attribute Name|Required|Description
|---|---|---|
|identifier|N|User identifier
|ipAddress|Y|User IP address
|firstName|Y|User first name
|lastName|Y|User last name
|phoneNumber|Y|User phone number
|emailAddress|Y|User email address
|address|N|Depending on the channel -(SEE ADDRESS OBJECT)


**Address Object**
|Attribute Name|Required|Description
|---|---|---|
|addressLine1|Y|Physical Address
|cityTown|Y|City or Town
|stateProv|Y|State or Province
|country|Y|2-letter Country Code
|postalCode|Y|Postal Code

**Sample Request**

```
{
   "channelCode": "CHNL-101",
   "accountId": "1000000001",
   "requestSalt": "abcd-xyz",
   "txnAmount": 100,
   "txnCurrency": "USD",
   "paymentRef": "abc123XYZ",
   "paymentDesc": "description",
   "dupCheck":false,
   "merchantTraceData": "merchant trace data",
   "callbackUrl": "https://callback.com/call/back",
   "resultUrl": "https://result.url/result",
   "endUserInfo": {
       "identifier": "张三ABC",
       "ipAddress": "10.0.0.1",
       "firstName": "John",
       "lastName": "Doe",
       "phoneNumber": "2060987657",
       "emailAddress": "user@domain.com",
       "address": {
           "addressLine1": "123 ABC Street",
           "cityTown": "Queenstown",
           "stateProv": "Singapore",
           "country": "SG",
           "postalCode": "238858"
        }
    },
   "payInfo": {
        "bankCode": "CHTTST01",
        "bankName": "ABC",
        "bankAccountNumber": "123"
    }
}

```

**Response**

There are 2 types of responses that you may get when calling the payment API.

1. HTTP STATUS 422 - If the transaction fails authorization or validation, you will get an HTTP Status 422 (Unprocessable Entity). The response body will contain an error code and an error message. For complete error codes and description, please see Appendix. Sample error response:

```
{
          "msgCode": "E2046",
          "msgText": "Duplicate payment reference"
}
```


2. HTTP STATUS 200 - If the transaction passes authorization and validation, you will get an HTTP Status 200 (OK). The response body will contain a JSON formatted text with the following attributes:


|txnType|Fixed value of PAY-IN
|:---|:---|
|txnId|This is the transaction identifier
|status |Possible values are: <br> COMPLETED<br> FORAUTHZ<br> INPROCESS<br> CANCELLED<br> PENDING<br> FAILED
|statusTimestamp|Timestamp of the status; in long milliseconds format
|paymentRef|This is the payment reference you submitted in the request
|txnResultCode|Possible values are:<br> PENDING_SUBMIT<br> PENDING_AUTHZ<br> INPROCESS<br> TIMED_OUT<br> CANCELLED<br> INSUFFICENT_FUNDS<br> POSSIBLE_FRAUD<br> DECLINED<br> FAILED<br> SYSERR<br> SUCCESS
|txnResultMessage|A description of the result code as specified above
|amount|The amount of the requested payment
|currency|The currency of the requested payment
|processedAmount|The actual amount received when completed
|authzUrl|The URL that users should be redirected to for additional payment |actions
|authzRef|Reference number
|merchantTraceData|Merchant trace data
|resultUrl|result Url from merchant in the request


Customers should be redirected to the URL in authURL for instructions of payment transfer. 

**Sample Response**

```{
   "txnType": "PAY-IN",
   "txnId": 176272727440,
   "status": "COMPLETED",
   "statusTimeStamp": 1669868124,
   "paymentRef": "PREF-1659501526442",
   "txnResultCode": "SUCCESS",
   "txnResultMessage": "successful",
   "amount": 1000,
   "currency": "CNY",
   "authzUrl": "https://paymentredirectlink.com/paymentxyz"
}
```


## PAY OUT

Set the hash value (from API Authentication) in the request header with a header name of **`X-paymesh-authz`**

|Description|Withdrawing payment to one of the supported channels
|:---|:---|
|HTTP Method|POST
|Request Headers|X-paymesh-authz={hashed token} <br>Content-Type=application/json
|URL|https://{host}/mxapi/{merchantId}/bdpayout
|Request Body|In JSON format. Following are the attributes for the request body.

**Request Body**

|Attribute Name|Required|Description
|:---|:---|:---|
|channelCode|Y|Direct payment channel selection code. Channel Code(s) would be |assigned to your solution. Please see Data Requirements in Appendix for |reference.
|accountId|Y|Assigned account ID which is a numeric, 10 digit identifier.
|requestSalt|Y|Any string of any length. Should be different for every request. |Suggest using current time in milliseconds. 
|txnAmount|Y|The amount of the payment. Numeric, 2 decimal places (e.g. 10.00).
|txnCurrency|Y|Currency of the payment (ex. CNY). 
|paymentRef|Y|Your own transaction identifier
|paymentDesc|N|A description of the transaction
|dupCheck|N|Valid values are “true” or “false”. If set to true, we will check if a |transaction with the same paymentRef has already been submitted. If a |duplicate is found, an error will be returned. If set to false, no |duplicate checking will be performed. The default is false.|merchantTraceData|N|Custom merchant field to pass data which will be echoed back in the|result, the redirect (for 3d) and in queries.
|resultUrl|Y|Redirect Url if needed
|callbackUrl|Y|Server notification callback Url
|payInfo|see description|Conditional - depends on channel requirement. This is an object with key-value pair attributes. The attributes vary depending on the |channel. It is recommended to call the Channel Code Specification Query |API to determine the specific requirements. 
|endUserInfo|Y|User Information Object (see below)


**`endUserInfo`  Object**

|Attribute Name|Required|Description
|---|---|---|
|identifier|N|User identifier
|ipAddress|Y|User IP address
|firstName|Y|User first name
|lastName|Y|User last name
|phoneNumber|Y|User phone number
|emailAddress|Y|User email address
|address|N|Depending on the channel -(SEE ADDRESS OBJECT)


**Address Object**

|Attribute Name|Required|Description
|---|---|---|
|addressLine1|Y|Physical Address
|cityTown|Y|City or Town
|stateProv|Y|State or Province
|country|Y|2-letter Country Code
|postalCode|Y|Postal Code

**Sample Request**

```{
   "channelCode": "USDMOCK-001",
   "accountId": "1000000001",
   "requestSalt": "abcd-xyz",
   "txnAmount": 100,
   "txnCurrency": "USD",
   "paymentRef": "abc123XYZ",
   "paymentDesc": "description",
   "dupCheck":false,
   "merchantTraceData": "merchant trace data",
   "callbackUrl": "https://callback.com/call/back",
   "resultUrl": "https://result.url/result",
   "endUserInfo": {
       "identifier": "张三ABC",
       "ipAddress": "10.0.0.1",
       "firstName": "John",
       "lastName": "Doe",
       "phoneNumber": "2060987657",
       "emailAddress": "user@domain.com",
       "address": {
           "addressLine1": "123 ABC Street",
           "cityTown": "Queenstown",
           "stateProv": "Singapore",
           "country": "SG",
           "postalCode": "238858"
        }


    },
    "payInfo": {
        "bankCode": "USD-TEST01",
        "bankAccountNumber": "1234567869",
        "bankRoutingCode": "TEST0123456"
    }
}
```

**Response**

There are 2 types of responses that you may get when calling the payment API.

1. **HTTP STATUS 422** - If the transaction fails authorization or validation, you will get an HTTP Status 422 (Unprocessable Entity). The response body will contain an error code and an error message. For complete error codes and description, please see Appendix. Sample error response:

```
{
          "msgCode": "E2046",
          "msgText": "Duplicate payment reference"
}
```

2. **HTTP STATUS 200** - If the transaction passes authorization and validation, you will get an HTTP Status 200 (OK). The response body will contain a JSON formatted text with the following attributes:


|txnType|Fixed value of PAY-OUT
|:---|:---|
|txnId|This is the transaction identifier
|status|Possible values are:<br>COMPLETED<br>INPROCESS<br>FORAUTHZ<br>CANCELLED<br>PENDING<br>FAILED
|statusTimestamp|Timestamp of the status; in long milliseconds format
|paymentRef|This is the payment reference you submitted in the request
|txnResultCode|Possible values are:<br>PENDING_SUBMIT<br>PENDING_AUTHZ<br>TIMED_OUT<br>CANCELLED<br>INSUFFICENT_FUNDS<br>POSSIBLE_FRAUD<br>DECLINED<br>FAILED<br>SYSERR<br>SUCCESS
|txnResultMessage|A description of the result code as specified above
|amount|The amount of the requested payment
|currency|The currency of the requested payment
|processedAmount|The actual amount sent when completed
|authzUrl|If not blank, redirect the users to this URL for additional payment |actions. Otherwise, no further action is required.
|authzRef|Reference number
|merchantTraceData|Merchant trace data
|resultUrl|result Url from merchant in the request



Typically, the response would contain a status of INPROCESS meaning the transaction has passed basic validation and the transfer of funds has been initiated. The final result will be posted to the “callbackUrl” specified in the request. 


**Sample Response**
```
{
   "txnType": "PAY-OUT",
   "txnId": 176272727440,
   "status": "INPROCESS",
   "statusTimeStamp": 1669868124,
   "paymentRef": "PREF-1659501526442",
   "txnResultCode": "SUCCESS",
   "txnResultMessage": "successful",
   "amount": 100,
   "processedAmount": 0,
   "currency": "USD"
}
```


# CALLBACKS

Due to the asynchronous nature of the payment channels, in Pay In and Pay Out request calls, we will perform a HTTP POST to the callback URL that you specified in the request when the transaction is COMPLETED, CANCELLED or FAILED. This would return a JSON object with the format of that object being the same as the result from the original request response (Pay In Response and Pay Out Response). The callback request has `X-paymesh-authz` header for authentication verification. Contact the support team on how this header is calculated. 


# QUERY APIs

## PAYMENT STATUS

### By Transaction ID using GET Method

**Hashed Token Specifications**

The hashed token for querying a transaction using our transaction ID would consist of the following attributes:
- merchantId
- txnId
- your API key

|Description|Retrieve the payment result by the txnId
|:---|:---|
|HTTP Method|GET
|Request Headers|X-paymesh-authz={hashed token} <br>Content-Type=application/json
|URL|https://{host}/mxapi/{merchantId}/txn/{txnId}


**Response Specifications**
Similar to the Payment API, you could get an HTTP 422 or an HTTP 200 response. The format and attributes of the response body would be the same as those in the callback under Payment API.

### By Reference ID using GET Method

**Hashed Token Specifications**
The hashed token for querying a transaction using your payment reference ID would consist of the following attributes:
- merchantId
- paymentRef
- your API key


| Description| Retrieve the payment result by your payment reference ID
|:---|:---|
| HTTP Method| GET
| Request Headers| X-paymesh-authz={hashed token} <br> Content-Type=application/json
| URL| https://{host}/mxapi/{merchantId}/txn/payref/{paymentRef}


**Response Specifications**
Similar to the Payment API, you could get an HTTP 422 or an HTTP 200 response. The format and attributes of the response body for an HTTP 422 response would be the same as those for the Payment API. For an HTTP 200 response, however, the response body will be in the format of a list/array. This is because there may be several transactions associated with a single paymentRef if duplicate checking is not enabled. The format and attributes for each entry in the list/array would be the same as those in the callback under Payment API.

## RETRIEVE LIST OF BANK CODES

**Hashed Token Specifications**
The hashed token for making a query would consist of the following attributes:
- merchantId
- channelCode
- your API key


| Description| Retrieve the list of valid bank identifiers for a given channel
|:---|:---|
| HTTP Method| GET
| Request Headers| X-paymesh-authz={hashed token} <br> Content-Type=application/json
| URL| https://{host}/mxapi/{merchantId}/bankcodes/{channelCode}
| Request Parameters| requestSalt


**Sample Request**
https://{host}/mxapi/3568224497743/bankcodes/USDMOCK-001


**Sample Response**

```
{
   "channelCode": "USDMOCK-001",
   "supportedCurrency": "USD",
   "payinBankCodeRequired": true,
   "payoutBankCodeRequired": true,
   "message": null,
   "banks": [
      {
         "bankCode": "USD-TEST01",
         "bankName": "Test Bank 01"
      },
      {
         "bankCode": "USD-TEST02",
         "bankName": "Test Bank 02"
      },
      {
         "bankCode": "USD-TEST03",
         "bankName": "Test Bank 03"
      },
      {
         "bankCode": "USD-TEST04",
         "bankName": "Test Bank 04"
      },
      {
         "bankCode": "USD-TEST05",
         "bankName": "Test Bank 05"
      }
   ]
}
```

## MERCHANT BALANCE

**Hashed Token Specifications**
The hashed token for querying merchant balance would consist of the following attributes:
- merchantId
- requestSalt
- currency
- your API key

Concatenate the attributes above in that order and generate the hex-encoded SHA-256 string of the concatenated string. 

| Description| Retrieve the Merchant balance
|:---|:---|
| HTTP Method| GET
| Request Headers| X-paymesh-authz={hashed token} <br> Content-Type=application/json
| URL| https://{{host}}/mxapi/{{merchantId}}/balance/{{currency}}?requestSalt={{salt}}

**Sample Request**

https://{host}/mxapi/3568224497743/balance/USD?requestSalt=abc123


**Response**

Similar to the Payment API, you could get an HTTP 422 or an HTTP 200 response. The format and attributes of the response body for an HTTP 422 response would be the same as those for the Payment API. For an HTTP 200 response,, the response body will be in the format of a Json object.

**Sample response**
```
{
    "currency": "PHP",
    "ledgerBalance": 1198,
    "pendingPayouts": 0.0,
    "availableBalance": 1198.0,
    "asOfDate": "2024-07-18T16:48:40.707+00:00"
}
```


## CHANNEL SPECIFICATIONS using GET

**Hashed Token Specifications**
The hashed token for querying merchant balance would consist of the following attributes:
- merchantId
- requestSalt
- channelCode
- your API key

Concatenate the attributes above in that order and generate the hex-encoded SHA-256 string of the concatenated string. 


|Description|Accepting payment from one of the supported channels
|:---|:---|
|HTTP Method|GET
|Request Headers|X-paymesh-authz={hashed token}<br>Content-Type=application/json
|URL|https://{host}/mxapi/{merchantId}/channelspecs/{channelCode}?requestSalt={salt}
|Request Parameters|requestSalt


**Sample Request**
https://{host}/mxapi/3568224497743/channelspecs/mock001?requestSalt=abc


**Sample Response**
```
{
   "channelCode": "USDMOCK-001",
   "currency": "USD",
   "payinSpecs": {
       "enabled": true,
       "minTxnAmount": 1,
       "maxTxnAmount": 1000,
       "requiredEndUserInfo": [
           {
               "fieldName": "firstName",
               "description": "First Name"
           },
           {
               "fieldName": "lastName",
               "description": "Last Name"
           },
           {
               "fieldName": "emailAddress",
               "description": "Email Address"
           },
           {
               "fieldName": "phoneNumber",
               "description": "Phone Number"
           }
       ],
       "requiredPayInfo": null,
       "validateBankCode": false,
       "bankCodeRequired": false
   },
   "payoutSpecs": {
       "enabled": true,
       "minTxnAmount": 1,
       "maxTxnAmount": 1000,
       "requiredEndUserInfo": [
           {
               "fieldName": "firstName",
               "description": "First Name"
           },
           {
               "fieldName": "lastName",
               "description": "Last Name"
           },
           {
               "fieldName": "emailAddress",
               "description": "Email Address"
           },
           {
               "fieldName": "phoneNumber",
               "description": "Phone Number"
           }
       ],
       "requiredPayInfo": [
           {
               "fieldName": "bankCode",
               "description": "Bank Code"
           },
           {
               "fieldName": "bankAccountNumber",
               "description": "Bank Account Number"
           },
           {
               "fieldName": "bankAccountName",
               "description": "Bank Account Name"
           }
       ],
       "validateBankCode": true,
       "bankCodeRequired": true
   }
}

```


# APPENDIX

## Error Code Responses

| Error Code| Description
|:---|:---|
| E0000| Authentication required 
| E0001| Unauthorized access
| E0002| Authentication failed
| E0003| Account is locked
| E0004| Invalid user
| E0005| Enter current password
| E0006| Enter new password
| E0007| Entered new passwords do not match
| E0008| Invalid current password
| E0012| MFA is already enabled
| E0013| MFA code required
| E0042| Invalid Merchant ID
| E1000| Invalid merchant
| E1001| Merchant account is not active
| E1003| Merchant has no redirect URL
| E1022| Invalid account ID
| E2004| Invalid transaction authorization
| E2005| Missing transaction authorization
| E2020| Invalid transaction amount
| E2022| Unsupported currency
| E2029| Transaction not found
| E2045| Payment reference is required
| E2046| Duplicate payment reference
| E3000| Bad Request Error
| E9998| Validation Errors
| E9999| System Error






