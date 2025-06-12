# naFakturuj API Documentation

Modern REST API for invoice management system. Create and manage invoices, clients, and more programmatically.

## Base URL
```
https://api.nafakturuj.cz/v1
```

## Authentication

The API uses Bearer Authentication. You need to include your API key in the Authorization header:

```bash
Authorization: Bearer YOUR_API_KEY
```

## Endpoints

### Invoices

#### List Invoices

```http
GET /v1/invoices
```

Query Parameters:
- `limit` (integer, optional): Maximum number of invoices to return. Default is 10, max is 100.
- `offset` (integer, optional): Number of invoices to skip. Default is 0.
- `status` (string, optional): Filter by status (paid, unpaid, overdue)

Example Response:
```json
{
  "success": true,
  "data": {
    "invoices": [
      {
        "id": 123,
        "invoice_number": "FAK-2023-0001",
        "client_id": 456,
        "client_name": "ACME Corp",
        "issue_date": "2023-06-15",
        "due_date": "2023-06-30",
        "total_amount": 1250.00,
        "currency": "CZK",
        "status": "unpaid"
      }
    ],
    "total": 15,
    "limit": 5,
    "offset": 0
  },
  "error": null
}
```

#### Create Invoice

```http
POST /v1/invoices
```

Request Body:
```json
{
  "client_id": 456,
  "issue_date": "2023-06-15",
  "due_date": "2023-06-30",
  "invoice_type": "regular",
  "payment_method": "bank_transfer",
  "currency": "CZK",
  "items": [
    {
      "name": "Web Services",
      "quantity": 1,
      "unit_price": 10000,
      "vat_rate": 21
    }
  ],
  "note": "Thank you for your business"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "invoice_id": 124,
    "invoice_number": "FAK-2023-0002",
    "message": "Invoice created successfully"
  },
  "error": null
}
```

#### Get Invoice

```http
GET /v1/invoices/{id}
```

Retrieves detailed information about a specific invoice including public access information if the invoice is public.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| id | integer | Invoice ID |

**Example Request:**

```bash
curl -X GET \
  https://api.nafakturuj.cz/v1/invoices/102 \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "id": 102,
    "invoice_number": "FAK202503-0001",
    "invoice_type": "proforma",
    "client": {
      "id": 1,
      "name": "ACME Corp",
      "ico": "12345678",
      "dic": "CZ12345678",
      "address": "Testovací 123, Praha",
      "email": "client@example.com"
    },
    "company": {
      "id": 1,
      "name": "Má Firma",
      "ico": "87654321",
      "dic": "CZ87654321",
      "address": "Firemní 456, Praha",
      "bank_account": "1234567890/0800"
    },
    "dates": {
      "issue_date": "2025-05-01",
      "due_date": "2025-05-15"
    },
    "payment": {
      "method": "paypal",
      "status": "unpaid",
      "variabilni_symbol": "2025030001"
    },
    "amounts": {
      "total_without_vat": 5000,
      "total_vat": 1050,
      "total_with_vat": 6050,
      "currency": "CZK"
    },
    "note": "",
    "items": [
      {
        "id": 144,
        "name": "Webové služby",
        "quantity": 1,
        "unit_price": 5000,
        "vat_rate": 21,
        "total_without_vat": 5000,
        "vat_amount": 1050,
        "total_with_vat": 6050,
        "currency": "CZK"
      }
    ],
    "public_access": {
      "is_public": true,
      "public_token": "7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
      "public_url": "https://cab.nafakturuj.cz/public-invoice?token=7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
      "return_url": "https://example.com/thank-you",
      "paypal_available": true,
      "payment_url": "https://www.sandbox.paypal.com/checkoutnow?token=6W971358GJ269693H"
    }
  },
  "error": null
}
```

The `public_access` field is only included when the invoice is set to public.

#### Get Invoice PDF

```http
GET /v1/invoices/{id}/pdf
```

Parameters:
- `template` (string, optional): PDF template (default, modern, classic). Default is "default"
- `color` (string, optional): Color theme for modern template (blue, green, red, yellow). Default is "blue"

#### Set Invoice as Public

```http
POST /v1/invoices/{id}/setpublic
```

Sets an invoice as public and generates a public link for PayPal payment. This works only for invoices with PayPal payment method.

This endpoint is ideal for e-commerce stores or online services where you want to allow customers to pay invoices online via PayPal.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| id | integer | Invoice ID |

**Request Body Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| return_url | string | (Optional) URL where the customer will be redirected after successful payment |

**Example Request:**

```bash
curl -X POST \
  https://api.nafakturuj.cz/v1/invoices/102/setpublic \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"return_url": "https://example.com/thank-you"}'
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "invoice_id": 102,
    "is_public": true,
    "public_token": "7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
    "public_url": "https://cab.nafakturuj.cz/public-invoice?token=7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
    "return_url": "https://example.com/thank-you",
    "payment_method": "paypal",
    "paypal_available": true,
    "payment_url": "https://www.sandbox.paypal.com/checkoutnow?token=6W971358GJ269693H",
    "message": "Invoice set to public successfully"
  },
  "error": null
}
```

**New Features:**
- `payment_url`: Direct link to PayPal payment page that you can use to redirect customers directly
- `return_url`: Custom URL where the customer will be redirected after successful payment

**Possible Errors:**

| Code | Description |
|------|-------------|
| 400 | Public link is available only for invoices with PayPal payment method |
| 400 | PayPal is not configured for this user |
| 400 | Invalid return URL format |
| 404 | Invoice not found |


### Clients

#### List Clients

```http
GET /v1/clients
```

#### Create Client

```http
POST /v1/clients
```

### User Profile

#### Get User Profile

```http
GET /v1/users/profile
```

Response:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "username": "exider",
      "email": "user@example.com",
      "role": "admin",
      "created_at": "2024-03-27 18:46:04"
    },
    "subscription": {
      "plan_name": "VIP Business",
      "plan_code": "vip",
      "status": "active",
      "start_date": "2024-03-28",
      "end_date": "2025-03-28",
      "max_companies": 5,
      "used_companies": 1
    },
    "stats": {
      "total_invoices": 10,
      "paid_invoices": 8,
      "unpaid_invoices": 2,
      "total_amount": 25000,
      "total_clients": 5
    },
    "companies": {
      "total": 1,
      "remaining": 4
    }
  },
  "error": null
}
```

## Code Examples

### PHP

```php
<?php
$api_key = 'YOUR_API_KEY';
$base_url = 'https://api.nafakturuj.cz/v1';

// List invoices
function getInvoices($limit = 10, $offset = 0) {
    global $api_key, $base_url;
    
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $base_url . "/invoices?limit=$limit&offset=$offset");
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Authorization: Bearer ' . $api_key,
        'Content-Type: application/json'
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}

// Create invoice
function createInvoice($data) {
    global $api_key, $base_url;
    
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $base_url . '/invoices');
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Authorization: Bearer ' . $api_key,
        'Content-Type: application/json'
    ]);
    
    $response = curl_exec($ch);
    curl_close($ch);
    
    return json_decode($response, true);
}
```

### Python

```python
import requests

class nafakturujAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = 'https://api.nafakturuj.cz/v1'
        self.headers = {
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        }
    
    def get_invoices(self, limit=10, offset=0):
        response = requests.get(
            f'{self.base_url}/invoices',
            params={'limit': limit, 'offset': offset},
            headers=self.headers
        )
        return response.json()
    
    def create_invoice(self, data):
        response = requests.post(
            f'{self.base_url}/invoices',
            json=data,
            headers=self.headers
        )
        return response.json()
    
    def get_invoice_pdf(self, invoice_id, template='default', color='blue'):
        response = requests.get(
            f'{self.base_url}/invoices/{invoice_id}/pdf',
            params={'template': template, 'color': color},
            headers=self.headers
        )
        return response.content

# Usage example
api = nafakturujAPI('YOUR_API_KEY')

# List invoices
invoices = api.get_invoices(limit=5)
print(invoices)

# Create invoice
invoice_data = {
    "client_id": 456,
    "issue_date": "2023-06-15",
    "due_date": "2023-06-30",
    "invoice_type": "regular",
    "payment_method": "bank_transfer",
    "currency": "CZK",
    "items": [
        {
            "name": "Web Services",
            "quantity": 1,
            "unit_price": 10000,
            "vat_rate": 21
        }
    ]
}
result = api.create_invoice(invoice_data)
print(result)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

class nafakturujAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.nafakturuj.cz/v1';
        this.client = axios.create({
            baseURL: this.baseURL,
            headers: {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            }
        });
    }

    async getInvoices(limit = 10, offset = 0) {
        try {
            const response = await this.client.get('/invoices', {
                params: { limit, offset }
            });
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    async createInvoice(data) {
        try {
            const response = await this.client.post('/invoices', data);
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    async getInvoicePDF(invoiceId, template = 'default', color = 'blue') {
        try {
            const response = await this.client.get(`/invoices/${invoiceId}/pdf`, {
                params: { template, color },
                responseType: 'arraybuffer'
            });
            return response.data;
        } catch (error) {
            throw this.handleError(error);
        }
    }

    handleError(error) {
        if (error.response) {
            return new Error(error.response.data.error?.message || 'API Error');
        }
        return error;
    }
}

// Usage example
const api = new nafakturujAPI('YOUR_API_KEY');

// List invoices
api.getInvoices(5)
    .then(result => console.log(result))
    .catch(error => console.error(error));

// Create invoice
const invoiceData = {
    client_id: 456,
    issue_date: "2023-06-15",
    due_date: "2023-06-30",
    invoice_type: "regular",
    payment_method: "bank_transfer",
    currency: "CZK",
    items: [
        {
            name: "Web Services",
            quantity: 1,
            unit_price: 10000,
            vat_rate: 21
        }
    ]
};

api.createInvoice(invoiceData)
    .then(result => console.log(result))
    .catch(error => console.error(error));

// E-commerce PayPal Integration Example
class FakturacePayPalIntegration {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.nafakturuj.cz/v1';
        this.client = axios.create({
            baseURL: this.baseURL,
            headers: {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            }
        });
    }

    // Create an invoice with PayPal payment method
    async createPayPalInvoice(customerData, items, returnUrl) {
        try {
            // 1. Create the invoice with PayPal payment method
            const invoiceData = {
                client_id: customerData.clientId,
                issue_date: new Date().toISOString().split('T')[0],
                due_date: this.getDueDate(7), // 7 days from now
                invoice_type: "regular",
                payment_method: "paypal", // Important: Set payment method to PayPal
                currency: "CZK",
                items: items
            };
            
            const invoiceResult = await this.client.post('/invoices', invoiceData);
            const invoiceId = invoiceResult.data.data.invoice_id;
            
            // 2. Set the invoice as public and get payment link
            const publicResult = await this.client.post(`/invoices/${invoiceId}/setpublic`, {
                return_url: returnUrl // URL where customer will be redirected after payment
            });
            
            // 3. Return the payment info including the direct PayPal payment URL
            return {
                invoiceId: invoiceId,
                invoiceNumber: invoiceResult.data.data.invoice_number,
                paymentUrl: publicResult.data.data.payment_url, // Direct link to PayPal payment
                publicUrl: publicResult.data.data.public_url    // Link to our invoice page
            };
        } catch (error) {
            console.error('Error in PayPal integration:', error);
            throw error;
        }
    }
    
    // Helper to calculate due date (n days from now)
    getDueDate(days) {
        const date = new Date();
        date.setDate(date.getDate() + days);
        return date.toISOString().split('T')[0];
    }
}

// Usage example in an e-commerce checkout process
async function processCheckout(cart, customer) {
    const paypalIntegration = new FakturacePayPalIntegration('YOUR_API_KEY');
    
    // Map cart items to invoice items
    const invoiceItems = cart.items.map(item => ({
        name: item.name,
        quantity: item.quantity,
        unit_price: item.price,
        vat_rate: 21
    }));
    
    // Create invoice and get payment URL
    const paymentInfo = await paypalIntegration.createPayPalInvoice(
        { clientId: customer.id },
        invoiceItems,
        'https://your-shop.com/thank-you'
    );
    
    // Redirect customer to PayPal payment
    window.location.href = paymentInfo.paymentUrl;
}
```

## Error Handling

The API uses standard HTTP status codes:

| Code | Description |
|------|-------------|
| 200  | OK - Request successful |
| 201  | Created - Resource successfully created |
| 400  | Bad Request - Invalid request |
| 401  | Unauthorized - Authentication failed |
| 404  | Not Found - Resource doesn't exist |
| 429  | Too Many Requests - Rate limit exceeded |
| 500  | Server Error - Something went wrong on our end |

Error Response Format:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": 404,
    "message": "Invoice not found"
  }
}
```

## Rate Limits

60 requests per minute

Response Headers:
- `X-RateLimit-Limit`: Maximum allowed requests per minute
- `X-RateLimit-Remaining`: Remaining requests in current minute
- `X-RateLimit-Reset`: Time when current limit resets

## Support

For any questions or issues, please contact Exider or open an issue in this repository. 