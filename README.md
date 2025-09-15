# PAYable IPG React Native SDK

A comprehensive React Native SDK for integrating Internet Payment Gateway (IPG) functionality into your mobile applications. This SDK supports one-time payments and recurring payments with a secure, user-friendly interface.

## Table of Contents
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [One-Time Payments](#one-time-payments)
- [Recurring Payments](#recurring-payments)
- [Tokenized Payments](#tokenized-payments)
- [API Usage](#api-usage)
- [Merchant Notifications](#merchant-notifications)
- [Error Handling](#error-handling)
- [Security & Compliance](#security--compliance)
- [Testing](#testing)
- [API Reference](#api-reference)
- [FAQs & Troubleshooting](#faqs--troubleshooting)

## Installation

```sh
npm install ipg-reactnative-sdk
```

## Getting Started

1. Import the SDK in your React Native application:
```js
import PayableIPG from 'ipg-reactnative-sdk';
```

2. Initialize the SDK with your merchant credentials:
```js
const data = {
  environment: 'sandbox', // 'sandbox' | 'qa' | 'dev' | 'production'
  logoUrl: 'https://your-logo-url.com/logo.png',
  returnUrl: 'https://your-return-url.com',
  webhookUrl: 'https://your-webhook-url.com',
  merchantKey: 'YOUR_MERCHANT_KEY',
  merchantToken: 'YOUR_MERCHANT_TOKEN',
};
```

## Configuration

### Required Configuration
- `environment`: SDK environment ('sandbox', 'qa', 'dev', or 'production')
- `logoUrl`: Your merchant logo URL
- `returnUrl`: URL to redirect users after payment completion
- `webhookUrl`: URL to receive payment notifications
- `merchantKey`: Your merchant key provided by PAYable
- `merchantToken`: Your merchant token provided by PAYable

## One-Time Payments

### Flow Diagram
1. Initialize payment with required parameters
2. Generate checksum value
3. Present payment screen to user
4. Process payment
5. Handle payment completion/error

### Required Parameters
```js
const paymentData = {
  paymentType: "1", // 1 for one-time payment
  invoiceId: "INV123456", // Unique invoice ID
  amount: "100.00", // Payment amount with 2 decimal places
  currencyCode: "LKR", // Currency code
  orderDescription: "Payment for order #123",
  customerFirstName: "John",
  customerLastName: "Doe",
  customerMobilePhone: "0771234567",
  customerEmail: "john.doe@example.com",
  billingAddressStreet: "123 Main St",
  billingAddressCity: "Colombo",
  billingAddressCountry: "LKA",
  checkValue: "GENERATED_CHECKSUM" // Generated using getCheckValue()
};
```

### Checksum Generation
The checksum is generated using SHA-512 hashing algorithm. Here's the process:

```js
import CryptoJS from 'crypto-js';

const getCheckValue = (
  merchantKey: string,
  merchantToken: string,
  invoiceId: string,
  amount: string,
  currencyCode: string
) => {
  // Step 1: Generate hash from merchant token
  const hash1 = CryptoJS.SHA512(merchantToken)
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  // Step 2: Generate final checksum
  const checkValue = CryptoJS.SHA512(
    `${merchantKey}|${invoiceId}|${amount}|${currencyCode}|${hash1}`
  )
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  return checkValue;
};
```

### Payment Completion Response
```js
{
  merchantKey: "YOUR_MERCHANT_KEY",
  payableOrderId: "oid-XXXXXXXX-XXX-XXXX-XXXX-XXXX",
  payableTransactionId: "XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXX",
  payableAmount: "100.00",
  payableCurrency: "LKR",
  invoiceNo: "INV123456",
  statusCode: 1,
  statusMessage: "SUCCESS",
  paymentType: 1,
  paymentMethod: 1,
  paymentScheme: "MASTERCARD",
  custom1: "optional data",
  custom2: "optional data",
  cardHolderName: "John Doe",
  cardNumber: "512345xxxxxx0008",
  checkValue: "GENERATED_CHECKSUM"
}
```

### React Native Integration Example (One-Time Payment)

```tsx
import React, { useMemo, useState } from 'react';
import { View, Button, Alert } from 'react-native';
import PayableIPG from 'ipg-reactnative-sdk';
import CryptoJS from 'crypto-js';

const getCheckValue = (
  merchantKey: string,
  merchantToken: string,
  invoiceId: string,
  amount: string,
  currencyCode: string
) => {
  const hash1 = CryptoJS.SHA512(merchantToken)
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  return CryptoJS.SHA512(
    `${merchantKey}|${invoiceId}|${amount}|${currencyCode}|${hash1}`
  )
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();
};

export default function OneTimePaymentExample() {
  const [showIpg, setShowIpg] = useState(false);

  const PAYableIPGClient = useMemo(
    () => ({
      environment: 'sandbox', // 'sandbox' | 'qa' | 'dev' | 'production'
      logoUrl: 'https://your-logo-url.com/logo.png',
      returnUrl: 'https://your-return-url.com',
      webhookUrl: 'https://your-webhook-url.com',
      merchantKey: 'YOUR_MERCHANT_KEY',
      merchantToken: 'YOUR_MERCHANT_TOKEN',
    }),
    []
  );

  const payment = useMemo(() => {
    const amount = '100.00';
    const invoiceId = `INV-${Date.now()}`;
    const currencyCode = 'LKR';

    return {
      paymentType: '1',
      invoiceId,
      amount,
      currencyCode,
      orderDescription: 'Payment for order #123',
      customerFirstName: 'John',
      customerLastName: 'Doe',
      customerMobilePhone: '0771234567',
      customerEmail: 'john.doe@example.com',
      billingAddressStreet: '123 Main St',
      billingAddressCity: 'Colombo',
      billingAddressCountry: 'LKA',
      checkValue: getCheckValue(
        PAYableIPGClient.merchantKey,
        PAYableIPGClient.merchantToken,
        invoiceId,
        amount,
        currencyCode
      ),
    };
  }, [PAYableIPGClient]);

  const handlePaymentStarted = () => {
    // Optional: track analytics
  };

  const handlePaymentCompleted = (result: any) => {
    setShowIpg(false);
    Alert.alert('Payment Success', JSON.stringify(result, null, 2));
  };

  const handlePaymentError = (error: any) => {
    setShowIpg(false);
    Alert.alert('Payment Error', JSON.stringify(error, null, 2));
  };

  const handlePaymentCancelled = () => {
    setShowIpg(false);
  };

  return (
    <View style={{ flex: 1 }}>
      {!showIpg && (
        <Button title="Pay LKR 100.00" onPress={() => setShowIpg(true)} />
      )}

      {showIpg && (
        <PayableIPG
          PAYableIPGClient={PAYableIPGClient}
          packageName={"com.yourapp"}
          paymentType={payment.paymentType}
          invoiceId={payment.invoiceId}
          amount={payment.amount}
          currencyCode={payment.currencyCode}
          orderDescription={payment.orderDescription}
          customerFirstName={payment.customerFirstName}
          customerLastName={payment.customerLastName}
          customerMobilePhone={payment.customerMobilePhone}
          customerEmail={payment.customerEmail}
          billingAddressStreet={payment.billingAddressStreet}
          billingAddressCity={payment.billingAddressCity}
          billingAddressCountry={payment.billingAddressCountry}
          checkValue={payment.checkValue}
          onPaymentStarted={handlePaymentStarted}
          onPaymentCompleted={handlePaymentCompleted}
          onPaymentError={handlePaymentError}
          onPaymentCancelled={handlePaymentCancelled}
        />
      )}
    </View>
  );
}
```

## Recurring Payments

### Flow Overview
1. Initialize recurring payment with required parameters
2. Generate checksum value
3. Present payment screen to user
4. Process initial payment
5. Set up recurring schedule
6. Handle recurring payments

### Required Parameters
```js
const recurringPaymentData = {
  paymentType: "2", // 2 for recurring payment
  invoiceId: "INV123456",
  amount: "100.00",
  currencyCode: "LKR",
  orderDescription: "Monthly subscription",
  customerFirstName: "John",
  customerLastName: "Doe",
  customerMobilePhone: "0771234567",
  customerEmail: "john.doe@example.com",
  billingAddressStreet: "123 Main St",
  billingAddressCity: "Colombo",
  billingAddressCountry: "LKA",
  startDate: "2024-01-01",
  endDate: "2024-12-31", // or "FOREVER"
  recurringAmount: "100.00",
  interval: "MONTHLY", // or "ANNUALLY"
  isRetry: "1", // 1 for true, 0 for false
  retryAttempts: "3",
  doFirstPayment: "1", // 1 for true, 0 for false
  checkValue: "GENERATED_CHECKSUM"
};
```

### Checksum Generation for Recurring Payments
```js
const getRecurringCheckValue = (
  merchantKey: string,
  merchantToken: string,
  invoiceId: string,
  amount: string,
  currencyCode: string,
  customerRefNo: string
) => {
  const hash1 = CryptoJS.SHA512(merchantToken)
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  const checkValue = CryptoJS.SHA512(
    `${merchantKey}|${invoiceId}|${amount}|${currencyCode}|${customerRefNo}|${hash1}`
  )
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  return checkValue;
};
```

## Tokenized Payments

### Overview
Tokenized payments allow merchants to securely store customer payment information for future use. This feature enhances the customer experience by eliminating the need to re-enter payment details for subsequent transactions while maintaining the highest security standards.


### Token Creation Flow
1. Customer makes initial payment and opts to save card
2. System generates a unique token for the card
3. Token is securely stored in PAYable's vault
4. Token ID is returned to merchant for future use

### Required Parameters for Tokenized Payment
```js
const tokenPaymentData = {
  paymentType: "3", // 3 for tokenized payment
  invoiceId: "INV123456",
  amount: "100.00",
  currencyCode: "LKR",
  orderDescription: "Token creation payment",
  customerFirstName: "John",
  customerLastName: "Doe",
  customerMobilePhone: "0771234567",
  customerEmail: "john.doe@example.com",
  billingAddressStreet: "123 Main St",
  billingAddressCity: "Colombo",
  billingAddressCountry: "LKA",
  isSaveCard: "1", // 1 to save card for future use
  customerRefNo: "CUST123456", // Unique customer reference
  checkValue: "GENERATED_CHECKSUM"
};
```

### Checksum Generation for Token Payments
```js
const getTokenCheckValue = (
  merchantKey: string,
  merchantToken: string,
  invoiceId: string,
  amount: string,
  currencyCode: string,
  customerRefNo: string,
  tokenId?: string
) => {
  const hash1 = CryptoJS.SHA512(merchantToken)
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  const checkValue = CryptoJS.SHA512(
    `${merchantKey}|${invoiceId}|${amount}|${currencyCode}|${customerRefNo}|${tokenId || ''}|${hash1}`
  )
    .toString(CryptoJS.enc.Hex)
    .toUpperCase();

  return checkValue;
};
```


#### Authentication Headers
```js
{
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
  'Content-Type': 'application/json',
  'X-Merchant-Key': 'YOUR_MERCHANT_KEY'
}
```

### API Endpoints

#### Token Usage
```js
POST /api/v1/payments/token
Content-Type: application/json

Request Body:
{
  "merchantKey": "YOUR_MERCHANT_KEY",
  "invoiceId": "INV123456",
  "amount": "100.00",
  "currencyCode": "LKR",
  "tokenId": "TOKEN123456",
  "customerRefNo": "CUST123456",
  "checkValue": "GENERATED_CHECKSUM"
}

Response:
{
  "status": "SUCCESS",
  "transactionId": "TXN123456",
  "amount": "100.00",
  "currency": "LKR"
}
```

### 1. List Saved Cards

Retrieve all saved cards for a customer.

**Endpoint**: `POST {{rootUrl}}/ipg/v2/tokenize/listCard`

**Headers Required**:

```
Content-Type: application/json
```

**Required Parameters**:

- `merchantId` - Your merchant ID
- `customerId` - Customer ID
- `checkValue` - Security hash

**CheckValue Generation:**

```
SHA512(merchantId|customerId|SHA512(merchantToken))
```

### 2. Delete Saved Card

Remove a saved card from customer's account.

**Endpoint**: `POST {{rootUrl}}/ipg/v2/tokenize/deleteCard`

**Headers Required**:

```
Content-Type: application/json
Authorization: Bearer {access_token}
```

**Required Parameters**:

- `merchantId` - Your merchant ID
- `customerId` - Customer ID
- `tokenId` - Token ID to delete
- `checkValue` - Security hash

**CheckValue Generation:**

```
SHA512(merchantId|customerId|tokenId|SHA512(merchantToken))
```

### 3. Edit Saved Card

Update card nickname or set as default card.

**Endpoint**: `POST {{rootUrl}}/ipg/v2/tokenize/editCard`

**Headers Required**:

```
Content-Type: application/json
Authorization: Bearer {access_token}
```

**How to Generate JWT Access Token**:

1. Create Basic Auth token: `base64(businessKey:businessToken)`
2. POST to `{{rootUrl}}/ipg/v2/auth/tokenize` with headers:
   ```
   Content-Type: application/json
   Authorization: {basicAuthToken}
   ```
   Body: `{"grant_type": "client_credentials"}`
3. Extract `accessToken` from response and use in Authorization header

**Required Parameters**:

- `merchantId` - Your merchant ID
- `customerId` - Customer ID
- `tokenId` - Token ID to edit
- `nickName` - Optional nickname for the card
- `isDefaultCard` - Set as default card (0 or 1)
- `checkValue` - Security hash

### 4. Pay with Saved Card

Process payment using a previously saved card token.

**Endpoint**: `POST {{rootUrl}}/ipg/v2/tokenize/pay`

**Headers Required**:

```
Content-Type: application/json
Authorization: Bearer {access_token}
```

**How to Generate Access Token**:

1. Create Basic Auth token: `base64(businessKey:businessToken)`
2. POST to `{{rootUrl}}/ipg/v2/auth/tokenize` with headers:
   ```
   Content-Type: application/json
   Authorization: {basicAuthToken}
   ```
   Body: `{"grant_type": "client_credentials"}`
3. Extract `accessToken` from response and use in Authorization header

**Required Parameters**:

- `merchantId` - Your merchant ID
- `customerId` - Customer ID
- `tokenId` - Token ID to use for payment
- `invoiceId` - Invoice ID
- `amount` - Payment amount
- `currencyCode` - Currency code
- `checkValue` - Security hash
- `webhookUrl` - Webhook URL for notifications

**CheckValue Generation:**

```
SHA512(merchantId|invoiceId|amount|currencyCode|customerId|tokenId|SHA512(merchantToken))
```

## Merchant Notifications

### Webhook Setup
1. Configure your webhook endpoint in the SDK initialization
2. Implement webhook handler to process notifications
3. Validate incoming notifications using checksum

### Webhook Payload Structure
```js
{
  merchantKey: "YOUR_MERCHANT_KEY",
  payableOrderId: "oid-XXXXXXXX-XXX-XXXX-XXXX-XXXX",
  payableTransactionId: "XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXX",
  payableAmount: "100.00",
  payableCurrency: "LKR",
  invoiceNo: "INV123456",
  statusCode: 1,
  statusMessage: "SUCCESS",
  paymentType: 1,
  paymentMethod: 1,
  paymentScheme: "MASTERCARD",
  custom1: "optional data",
  custom2: "optional data",
  cardHolderName: "John Doe",
  cardNumber: "512345xxxxxx0008",
  checkValue: "GENERATED_CHECKSUM"
}
```

### Webhook Response
```js
{
  "Status": 200
}
```

## Error Handling

### Common Error Codes
- 3009: Field validation error
- 400: Bad request
- 500: Server error

### Error Response Format
```js
{
  status: 3009,
  success: false,
  error: {
    fieldName: ["Error message"]
  }
}
```
## Security

### Important Security Notes

- **merchantToken** is a **secret** shared by PAYable for your merchant
- Use exact field order and string concatenation with `|` as the delimiter
- Always validate webhook responses using checkValue
- Store tokens securely in your database
- Use HTTPS for all communications
- Never expose your merchantToken in client-side code

### CheckValue Validation

```javascript
// Validate webhook response
function validateWebhook(webhookData, merchantToken) {
  const calculatedCheckValue = CryptoJS.SHA512(
    webhookData.merchantKey +
      "|" +
      webhookData.payableOrderId +
      "|" +
      webhookData.payableTransactionId +
      "|" +
      webhookData.payableAmount +
      "|" +
      webhookData.payableCurrency +
      "|" +
      webhookData.invoiceNo +
      "|" +
      webhookData.statusCode +
      "|" +
      CryptoJS.SHA512(merchantToken).toString().toUpperCase()
  )
    .toString()
    .toUpperCase();

  return calculatedCheckValue === webhookData.checkValue;
}
```


### API Reference

### PayableIPG Component
```js
<PayableIPG
  PAYableIPGClient={data}
  packageName={packageName}
  paymentType={paymentType}
  invoiceId={invoiceId}
  amount={amount}
  currencyCode={currencyCode}
  orderDescription={orderDescription}
  // ... other props
  onPaymentStarted={handlePaymentStarted}
  onPaymentCompleted={handlePaymentCompleted}
  onPaymentError={handlePaymentError}
  onPaymentCancelled={handlePaymentCancelled}
/>
```

### Event Handlers
- `onPaymentStarted`: Called when payment process begins
- `onPaymentCompleted`: Called when payment is successful
- `onPaymentError`: Called when payment fails
- `onPaymentCancelled`: Called when user cancels payment


### Support
   - Support: +94 11 777 6 777
   - Website: https://www.payable.lk

