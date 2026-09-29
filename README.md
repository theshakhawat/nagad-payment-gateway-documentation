# Nagad Online Payment Gateway Integration Guide (Native / Package-Free)

[![PHP Version](https://img.shields.io/badge/PHP-8.1%2B%20%7C%208.2%20%7C%208.3%20%7C%208.5-777BB4?logo=php&logoColor=white)](https://php.net)
[![Laravel](https://img.shields.io/badge/Laravel-10.x%20%7C%2011.x%20%7C%2012.x-FF2D20?logo=laravel&logoColor=white)](https://laravel.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Author](https://img.shields.io/badge/Developed%20By-Shakhawat%20Hossain-E83823?style=flat&logo=globe)](https://shakhawatdev.com)

একটি সম্পূর্ণ, প্রফেশনাল এবং প্যাকেজ-মুক্ত (No Third-Party Package Required) গাইড—যা যেকোনো প্রোগ্রামিং ল্যাঙ্গুয়েজ (PHP/Laravel, Node.js, Python ইত্যাদি)-এ সরাসরি নগদের অফিশিয়াল REST API, RSA পাবলিক-প্রাইভেট কী এনক্রিপশন এবং SHA256 ডিজিটাল সিগনেচার দিয়ে পেমেন্ট গেটওয়ে ইন্টিগ্রেট করার নিয়ম ব্যাখ্যা করে।

---

## 📑 সূচিপত্র (Table of Contents)
1. [ভূমিকা ও পেমেন্ট গেটওয়ে মেন্টাল মডেল](#1-ভূমিকা-ও-পেমেন্ট-গেটওয়ে-মেন্টাল-মডেল)
2. [পেমেন্ট লাইফসাইকেল (Full Lifecycle)](#2-পেমেন্ট-লাইফসাইকেল-full-lifecycle)
3. [ক্রেডেনশিয়াল ও ৩টি ক্রিপ্টোগ্রাফিক চাবি (The 3 Keys)](#3-ক্রেডেনশিয়াল-ও-৩টি-ক্রিপ্টোগ্রাফিক-চাবি-the-3-keys)
4. [কী জেনারেশন ও ডাউনলোড গাইড](#4-কী-জেনারেশন-ও-ডাউনলোড-গাইড)
5. [এনক্রিপশন বনাম ডিজিটাল সিগনেচার](#5-এনক্রিপশন-বনাম-ডিজিটাল-সিগনেচার)
6. [Laravel ইমপ্লিমেন্টেশন (Step-by-Step Code)](#6-laravel-ইমপ্লিমেন্টেশন-step-by-step-code)
7. [API রেফারেন্স ও পেলোড ফরম্যাট](#7-api-রেফারেন্স-ও-পেলোড-ফরম্যাট)
8. [Node.js এবং Python কোড উদাহরণ](#8-nodejs-এবং-python-কোড-উদাহরণ)
9. [বাস্তব এরর সমাধান ও ট্রাবলশুটিং](#9-বাস্তব-এরর-সমাধান-ও-ট্রাবলশুটিং)
10. [প্রোডাকশন সিকিউরিটি চেকলিস্ট](#10-প্রোডাকশন-সিকিউরিটি-চেকলিস্ট)
11. [ক্রেডিট ও লাইসেন্স](#11-ক্রেডিট-ও-লাইসেন্স)

---

## 1. ভূমিকা ও পেমেন্ট গেটওয়ে মেন্টাল মডেল

একজন কাস্টমার যখন আপনার ওয়েবসাইট বা অ্যাপে কোনো পেমেন্ট করতে চায়, আপনার সার্ভার কখনোই কাস্টমারের নগদ পিন (PIN) বা ওটিপি (OTP) সরাসরি নেয় না।

```
+----------------+          +---------------+          +----------------------+
|  Your Website  | -------> |   Nagad API   | -------> |  Nagad Checkout UI   |
| (Order Placed) |          | (Init & Order)|          |  (PIN & OTP Auth)    |
+----------------+          +---------------+          +----------------------+
```

### কেন কোনো প্যাকেজ ছাড়া করবেন?
* **সম্পূর্ণ কন্ট্রোল:** কোনো থার্ড-পার্টি প্যাকেজের ডিপেন্ডেন্সি এরর বা আন-মেইনটেইনড কোডের ওপর নির্ভর করতে হয় না।
* **সিকিউর ও লাইটওয়েট:** লারাভেলের বিল্ট-ইন `Http` ক্লায়েন্ট এবং পিএইচপির নেটিভ `openssl` লাইব্রেরি ব্যবহার করেই সম্পূর্ণ কাজ করা যায়।

---

## 2. পেমেন্ট লাইফসাইকেল (Full Lifecycle)

```
[CUSTOMER]
   │
   │ 1. ক্লিক করে "Pay with Nagad"
   ▼
[YOUR SERVER]
   │
   │ 2. Initialize API কল করে (Encrypt + Sign)
   ▼
[NAGAD API]
   │
   │ 3. এনক্রিপ্টেড Response পাঠায়
   ▼
[YOUR SERVER]
   │
   │ 4. Decrypt করে paymentReferenceId ও challenge বের করে
   │ 5. Complete API কল করে (Amount + Order Details)
   ▼
[NAGAD API]
   │
   │ 6. পেমেন্ট পেজের URL (callBackUrl) পাঠায়
   ▼
[CUSTOMER BROWSER]
   │
   │ 7. রিডাইরেক্ট হয় Nagad Payment Page-এ
   │ 8. নগদ একাউন্ট নম্বর, OTP ও PIN দিয়ে পেমেন্ট করে
   ▼
[NAGAD]
   │
   │ 9. রিডাইরেক্ট করে আপনার Callback URL-এ (GET Request)
   ▼
[YOUR SERVER]
   │
   │ 10. Verify Payment API কল করে সার্ভার-টু-সার্ভার যাচাই করে
   │ 11. Amount, Order ID ও Status === 'Success' নিশ্চিত করে
   ▼
[DATABASE]
   │
   │ 12. Order Status আপডেট করে: 'PAID'
```

---

## 3. ক্রেডেনশিয়াল ও ৩টি ক্রিপ্টোগ্রাফিক চাবি (The 3 Keys)

নগদ ইন্টিগ্রেশনে ৩টি আলাদা কী (Key) জড়িত থাকে। এদের কাজ স্পষ্টভাবে বোঝা জরুরি:

| Key Name | কার চাবি? | কোথায় থাকে? | মূল কাজ |
| :--- | :--- | :--- | :--- |
| **Merchant Private Key** | আপনার নিজের সিক্রেট চাবি | আপনার সার্ভারের `storage/` এ | রিকোয়েস্টে ডিজিটাল সিগনেচার তৈরি করা এবং নগদ থেকে আসা রেসপন্স ডিক্রিপ্ট (Decrypt) করা। |
| **Merchant Public Key** | আপনার কী-পেয়ারের পাবলিক অংশ | নগদ মার্চেন্ট পোর্টালে আপলোড করা থাকে | নগদ সার্ভার আপনার পাঠানো ডিজিটাল সিগনেচার ভেরিফাই করতে এটি ব্যবহার করে। |
| **Nagad Gateway Public Key** | নগদ কর্তৃক প্রদত্ত চাবি | নগদ পোর্টাল থেকে ডাউনলোড করে আপনার সার্ভারে রাখা | নগদ সার্ভারে ডেটা পাঠানোর আগে এনক্রিপ্ট করা এবং নগদের রেসপন্স সিগনেচার ভেরিফাই করা। |

---

## 4. কী জেনারেশন ও ডাউনলোড গাইড

### ধাপ ১: টার্মিনালে আপনার নিজস্ব RSA Key Pair জেনারেট করুন
```bash
# ১. আপনার Merchant Private Key তৈরি করুন (2048-bit)
openssl genrsa -out nagad_private_key.pem 2048

# ২. প্রাইভেট কী থেকে Merchant Public Key এক্সট্রাক্ট করুন
openssl rsa -in nagad_private_key.pem -pubout -out nagad_merchant_public_key.pem
```

### ধাপ ২: নগদ পোর্টালে আপলোড ও নাগাদ পাবলিক কী সংগ্রহ
1. নগদ মার্চেন্ট পোর্টালে লগইন করুন।
2. **Merchant Integration** মেন্যুতে যান।
3. আপনার তৈরি করা `nagad_merchant_public_key.pem` ফাইলের সম্পূর্ণ টেক্সট আপলোড করুন।
4. পোর্টাল থেকে নগদের নিজস্ব **Nagad Gateway Public Key** টি ডাউনলোড/কপি করে `nagad_public_key.pem` নামে সেভ করুন।

### ধাপ ৩: প্রজেক্টে ফাইলগুলো রাখুন
```text
storage/
└── keys/
    └── nagad/
        ├── nagad_private_key.pem  <-- আপনার Merchant Private Key
        └── nagad_public_key.pem   <-- পোর্টাল থেকে পাওয়া Nagad Public Key
```

---

## 5. এনক্রিপশন বনাম ডিজিটাল সিগনেচার

```
                                 PLAIN SENSITIVE DATA (JSON)
                                              │
                    ┌─────────────────────────┴─────────────────────────┐
                    │                                                   │
                    ▼                                                   ▼
            [ ENCRYPTION ]                                      [ SIGNATURE ]
    🔒 ডেটা গোপন রাখার জন্য                             ✍️ সত্যতা ও অখণ্ডতা নিশ্চিত করতে
                    │                                                   │
         Nagad Gateway Public Key                            Merchant Private Key
         RSA (OPENSSL_PKCS1_PADDING)                         RSA (OPENSSL_ALGO_SHA256)
                    │                                                   │
                    ▼                                                   ▼
         sensitiveData (Base64)                              signature (Base64)
```

---

## 6. Laravel ইমপ্লিমেন্টেশন (Step-by-Step Code)

### ১. `.env` কনফিগারেশন
```env
NAGAD_BASE_URL="https://api.mynagad.com/api/dfs"
NAGAD_MERCHANT_ID="YOUR_MERCHANT_ID"
NAGAD_ACCOUNT_NUMBER="017XXXXXXXX"
NAGAD_CALLBACK_URL="${APP_URL}/api/nagad/callback"
NAGAD_PUBLIC_KEY="keys/nagad/nagad_public_key.pem"
NAGAD_PRIVATE_KEY="keys/nagad/nagad_private_key.pem"
NAGAD_API_VERSION="v-0.2.0"
NAGAD_CLIENT_TYPE="PC_WEB"
NAGAD_SERVER_IP="YOUR_SERVER_PUBLIC_IP"
```

### ২. `config/nagad.php`
```php
<?php

return [
    'base_url'       => env('NAGAD_BASE_URL'),
    'merchant_id'    => env('NAGAD_MERCHANT_ID'),
    'account_number' => env('NAGAD_ACCOUNT_NUMBER'),
    'callback_url'   => env('NAGAD_CALLBACK_URL'),
    'public_key'     => env('NAGAD_PUBLIC_KEY'),
    'private_key'    => env('NAGAD_PRIVATE_KEY'),
    'api_version'    => env('NAGAD_API_VERSION', 'v-0.2.0'),
    'client_type'    => env('NAGAD_CLIENT_TYPE', 'PC_WEB'),
    'server_ip'      => env('NAGAD_SERVER_IP'),
];
```

### ৩. `app/Services/NagadService.php`
```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use RuntimeException;

class NagadService
{
    protected string $publicKey;
    protected string $privateKey;

    public function __construct()
    {
        $publicKeyPath = storage_path(config('nagad.public_key'));
        $privateKeyPath = storage_path(config('nagad.private_key'));

        if (! file_exists($publicKeyPath)) {
            throw new RuntimeException("Nagad public key not found: {$publicKeyPath}");
        }

        if (! file_exists($privateKeyPath)) {
            throw new RuntimeException("Nagad private key not found: {$privateKeyPath}");
        }

        $this->publicKey = file_get_contents($publicKeyPath);
        $this->privateKey = file_get_contents($privateKeyPath);
    }

    /**
     * 1. Initialize Payment
     */
    public function initialize(string $orderId): object
    {
        $merchantId = config('nagad.merchant_id');
        $url = rtrim(config('nagad.base_url')) . "/check-out/initialize/{$merchantId}/{$orderId}";
        $dateTime = now('Asia/Dhaka')->format('YmdHis');

        $sensitiveData = [
            'merchantId' => $merchantId,
            'datetime'   => $dateTime,
            'orderId'    => $orderId,
            'challenge'  => $this->generateChallenge(),
        ];

        $encryptedData = $this->encrypt($sensitiveData);
        $signature = $this->sign($sensitiveData);

        $payload = [
            'accountNumber' => config('nagad.account_number'),
            'dateTime'      => $dateTime,
            'sensitiveData' => $encryptedData,
            'signature'     => $signature,
        ];

        $response = Http::withHeaders($this->getHeaders())
            ->acceptJson()
            ->asJson()
            ->post($url, $payload);

        if ($response->failed()) {
            throw new RuntimeException("Nagad initialize request failed: {$response->body()}");
        }

        $responseData = $response->json();

        if (! is_array($responseData) || empty($responseData['sensitiveData']) || empty($responseData['signature'])) {
            throw new RuntimeException('Invalid or missing sensitiveData in Nagad initialize response.');
        }

        $plainData = $this->decryptRaw($responseData['sensitiveData']);
        $isValid = $this->verifySignature($plainData, $responseData['signature']);

        if (! $isValid) {
            throw new RuntimeException('Nagad initialize response signature verification failed.');
        }

        $decryptedData = json_decode($plainData, true);

        return (object) [
            'raw'  => $responseData,
            'data' => $decryptedData,
        ];
    }

    /**
     * 2. Complete / Place Order
     */
    public function complete(string $orderId, string $amount, array $initializeData): object
    {
        $paymentReferenceId = $initializeData['paymentReferenceId']
            ?? throw new RuntimeException('Payment reference ID not found.');

        $challenge = $initializeData['challenge']
            ?? $initializeData['random']
            ?? throw new RuntimeException('Challenge not found in Initialize response.');

        $sensitiveData = [
            'merchantId'   => config('nagad.merchant_id'),
            'orderId'      => $orderId,
            'amount'       => number_format((float) $amount, 2, '.', ''),
            'currencyCode' => '050',
            'challenge'    => $challenge,
        ];

        $encryptedData = $this->encrypt($sensitiveData);
        $signature = $this->sign($sensitiveData);

        $url = rtrim(config('nagad.base_url')) . "/check-out/complete/{$paymentReferenceId}";

        $payload = [
            'sensitiveData'       => $encryptedData,
            'signature'           => $signature,
            'merchantCallbackURL' => config('nagad.callback_url'),
        ];

        $response = Http::withHeaders($this->getHeaders())
            ->acceptJson()
            ->asJson()
            ->post($url, $payload);

        if ($response->failed()) {
            throw new RuntimeException("Nagad complete request failed: {$response->body()}");
        }

        $responseData = $response->json();

        return (object) $responseData;
    }

    /**
     * 3. Verify Payment
     */
    public function verify(string $paymentReferenceId): object
    {
        $url = rtrim(config('nagad.base_url'), '/') . "/verify/payment/{$paymentReferenceId}";

        $response = Http::withHeaders($this->getHeaders())
            ->acceptJson()
            ->get($url);

        if ($response->failed()) {
            throw new RuntimeException("Nagad verify request failed: {$response->body()}");
        }

        $responseData = $response->json();

        return (object) $responseData;
    }

    public function generateChallenge(int $length = 20): string
    {
        return bin2hex(random_bytes($length));
    }

    /* Crypto Helpers */

    protected function encrypt(array $data): string
    {
        $jsonData = json_encode($data, JSON_UNESCAPED_SLASHES);
        $publicKey = openssl_pkey_get_public($this->publicKey);

        if ($publicKey === false) {
            throw new RuntimeException('Invalid Nagad gateway public key.');
        }

        $encrypted = null;
        openssl_public_encrypt($jsonData, $encrypted, $publicKey, OPENSSL_PKCS1_PADDING);

        return base64_encode($encrypted);
    }

    protected function sign(array $data): string
    {
        $jsonData = json_encode($data, JSON_UNESCAPED_SLASHES);
        $privateKey = openssl_pkey_get_private($this->privateKey);

        if ($privateKey === false) {
            throw new RuntimeException('Invalid Nagad merchant private key.');
        }

        $signature = '';
        openssl_sign($jsonData, $signature, $privateKey, OPENSSL_ALGO_SHA256);

        return base64_encode($signature);
    }

    protected function decryptRaw(string $encryptedData): string
    {
        $decodedData = base64_decode($encryptedData, true);
        $privateKey = openssl_pkey_get_private($this->privateKey);

        if ($privateKey === false) {
            throw new RuntimeException('Invalid Nagad merchant private key.');
        }

        $decrypted = '';
        openssl_private_decrypt($decodedData, $decrypted, $privateKey, OPENSSL_PKCS1_PADDING);

        return $decrypted;
    }

    protected function verifySignature(string $plainData, string $signature): bool
    {
        $decodedSignature = base64_decode($signature, true);
        $publicKey = openssl_pkey_get_public($this->publicKey);

        if ($publicKey === false) {
            throw new RuntimeException('Invalid Nagad gateway public key.');
        }

        $result = openssl_verify($plainData, $decodedSignature, $publicKey, OPENSSL_ALGO_SHA256);

        return $result === 1;
    }

    protected function getHeaders(): array
    {
        return [
            'Content-Type'     => 'application/json',
            'X-KM-Api-Version' => config('nagad.api_version'),
            'X-KM-Client-Type' => config('nagad.client_type'),
            'X-KM-IP-V4'       => config('nagad.server_ip'),
        ];
    }
}
```

### ৪. `routes/web.php`
```php
use App\Http\Controllers\AdminController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth')->prefix('admin')->group(function () {
    Route::get('/test-nagad', [AdminController::class, 'nagadPayment'])->name('nagad.payment');
});

Route::get('/api/nagad/callback', [AdminController::class, 'nagadCallback'])->name('nagad.callback');
```

### ৫. `app/Http/Controllers/AdminController.php`
```php
namespace App\Http\Controllers;

use App\Services\NagadService;
use Illuminate\Http\Request;

class AdminController extends Controller
{
    public function __construct(protected NagadService $nagadService) {}

    public function nagadPayment()
    {
        // ১. ইউনিক ডায়নামিক অর্ডার আইডি তৈরি
        $orderId = 'ORD' . date('YmdHis') . rand(100, 999);
        $amount = '100.00';

        // ২. Initialize API
        $initializeData = $this->nagadService->initialize($orderId);

        // ৩. Complete API
        $completeResult = $this->nagadService->complete($orderId, $amount, (array) $initializeData->data);

        // ৪. কাস্টমারকে পেমেন্ট পেজে রিডাইরেক্ট করা
        return redirect()->away($completeResult->callBackUrl);
    }

    public function nagadCallback(Request $request)
    {
        $paymentReferenceId = $request->query('payment_ref_id');

        if (! $paymentReferenceId) {
            return response()->json(['status' => 'failed', 'message' => 'Missing payment reference ID'], 400);
        }

        // ৫. সার্ভার-টু-সার্ভার পেমেন্ট ভেরিফাই
        $result = $this->nagadService->verify($paymentReferenceId);

        if ($result->status === 'Success' && $result->merchantId === config('nagad.merchant_id')) {
            // ডাটাবেজে পেমেন্ট পেইড মার্ক করুন
            return response()->json([
                'status'  => 'success',
                'message' => 'Payment verified successfully!',
                'data'    => $result
            ]);
        }

        return response()->json(['status' => 'failed', 'message' => 'Payment verification failed.'], 400);
    }
}
```

---

## 7. API রেফারেন্স ও পেলোড ফরম্যাট

### রিকোয়েস্ট হেডার্স
```http
Content-Type: application/json
X-KM-Api-Version: v-0.2.0
X-KM-Client-Type: PC_WEB
X-KM-IP-V4: 103.112.54.40
```

### ১. Initialize API
* **Method:** `POST`
* **URL:** `/check-out/initialize/{merchantId}/{orderId}`
* **Plain Data:**
```json
{
    "merchantId": "YOUR_MERCHANT_ID",
    "datetime": "20260929112358",
    "orderId": "ORD20260929052355164",
    "challenge": "8f4abbef61aa8b8a88ba"
}
```

### ২. Complete / Place Order API
* **Method:** `POST`
* **URL:** `/check-out/complete/{paymentReferenceId}`
* **Plain Data:**
```json
{
    "merchantId": "YOUR_MERCHANT_ID",
    "orderId": "ORD20260929052355164",
    "amount": "100.00",
    "currencyCode": "050",
    "challenge": "8f4abbef61aa8b8a88ba"
}
```

### ৩. Verify Payment API
* **Method:** `GET`
* **URL:** `/verify/payment/{paymentReferenceId}`

---

## 8. Node.js এবং Python কোড উদাহরণ

### Node.js (Express & Native Crypto)
```javascript
const crypto = require('crypto');
const fs = require('fs');

const merchantPrivateKey = fs.readFileSync('./keys/nagad_private_key.pem', 'utf8');
const nagadGatewayPublicKey = fs.readFileSync('./keys/nagad_public_key.pem', 'utf8');

function encrypt(data) {
    const buffer = Buffer.from(JSON.stringify(data));
    const encrypted = crypto.publicEncrypt({
        key: nagadGatewayPublicKey,
        padding: crypto.constants.RSA_PKCS1_PADDING
    }, buffer);
    return encrypted.toString('base64');
}

function sign(data) {
    const signer = crypto.createSign('RSA-SHA256');
    signer.update(JSON.stringify(data));
    signer.end();
    return signer.sign(merchantPrivateKey, 'base64');
}
```

### Python (FastAPI/Flask + `cryptography`)
```python
import base64
import json
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding

with open("keys/nagad_private_key.pem", "rb") as f:
    private_key = serialization.load_pem_private_key(f.read(), password=None)

with open("keys/nagad_public_key.pem", "rb") as f:
    public_key = serialization.load_pem_public_key(f.read())

def encrypt(data: dict) -> str:
    json_bytes = json.dumps(data, separators=(',', ':')).encode('utf-8')
    encrypted = public_key.encrypt(json_bytes, padding.PKCS1v15())
    return base64.b64encode(encrypted).decode('utf-8')

def sign(data: dict) -> str:
    json_bytes = json.dumps(data, separators=(',', ':')).encode('utf-8')
    signature = private_key.sign(json_bytes, padding.PKCS1v15(), hashes.SHA256())
    return base64.b64encode(signature).decode('utf-8')
```

---

## 9. বাস্তব এরর সমাধান ও ট্রাবলশুটিং

| Error Code | Error Message | Cause & How to Fix |
| :--- | :--- | :--- |
| `16_0006_058` | **Failed to verify signature** | সিগনেচার মেথডে `OPENSSL_ALGO_SHA256` ব্যবহার করুন এবং পোর্টালে আপনার সঠিক পাবলিক কী আপলোড হয়েছে কিনা নিশ্চিত করুন। |
| `16_0006_083` | **Duplicate Order ID in same day** | টেস্ট করার সময় একই হার্ডকোডেড Order ID দেওয়া হয়েছে। প্রতিবার ডায়নামিক ও ইউনিক Order ID ব্যবহার করুন। |
| `16_0006_059` | **Invalid Sensitive Data** | ফিল্ডের নামে ভুল (যেমন `datetime` এর জায়গায় `dateTime` দিলে)। ছোট হাতের `datetime` ব্যবহার করুন। |
| `16_0006_068` | **Invalid Order Id** | Order ID তে স্পেশাল ক্যারেক্টার (যেমন আন্ডারস্কোর `_` বা স্পেস) থাকলে এই এরর আসে। শুধুমাত্র `A-Z, 0-9` ব্যবহার করুন। |
| `16_0006_064` | **Mandatory Header Missing** | `X-KM-Api-Version` বা `X-KM-IP-V4` হেডার মিসিং থাকলে। |

---

## 10. প্রোডাকশন সিকিউরিটি চেকলিস্ট

- [x] **Private Keys Secret রাখুন:** `.pem` কী ফাইল ও `.env` কখনো গিটহাবে পুশ করবেন না। `.gitignore`-এ যুক্ত রাখুন।
- [x] **কলব্যাক ডাটা কখনোই অন্ধভাবে বিশ্বাস করবেন না:** কলব্যাক আসার পর অবশ্যই `Verify Payment API` দিয়ে স্ট্যাটাস নিশ্চিত করুন।
- [x] **Amount ও Order ID মিলান:** Verify রেসপন্সের `amount` এবং `orderId` ডাটাবেজের সাথে হুবহু মিলতে হবে।
- [x] **Idempotency নিশ্চিত করুন:** একই অর্ডারের জন্য বারবার কলব্যাক এলেও যাতে ডাটাবেজে ডাবল পেমেন্ট প্রসেস না হয়।

---

## 11. ক্রেডিট ও লাইসেন্স

* **Author:** [Shakhawat Hossain](https://shakhawatdev.com)
* **Portfolio & Contact:** [shakhawatdev.com](https://shakhawatdev.com)
* **GitHub Profile:** [@theshakhawat](https://github.com/theshakhawat)
* **License:** [MIT License](LICENSE)

> যদি এই ডকুমেন্টেশনটি আপনার কাজে লেগে থাকে, তবে গিটহাবে একটি ⭐️ Star দিতে ভুলবেন না!
