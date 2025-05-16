# Fakturace API Documentation

Modern REST API for invoice management system. Create and manage invoices, clients, and more programmatically.

## Base URL
```
https://api.fakturace.site/v1
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

**Example Request:**

```bash
curl -X POST \
  https://api.fakturace.site/v1/invoices/102/setpublic \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "invoice_id": 102,
    "is_public": true,
    "public_token": "7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
    "public_url": "https://api.fakturace.site/public-invoice.php?token=7e5c8aaf3414c9b535dd62a078df408241306bf3f67b2628079e42d0e482c94",
    "payment_method": "paypal",
    "paypal_available": true,
    "message": "Invoice set to public successfully"
  },
  "error": null
}
```

**Possible Errors:**

| Code | Description |
|------|-------------|
| 400 | Public link is available only for invoices with PayPal payment method |
| 400 | PayPal is not configured for this user |
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
$base_url = 'https://api.fakturace.site/v1';

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

class FakturaceAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = 'https://api.fakturace.site/v1'
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
api = FakturaceAPI('YOUR_API_KEY')

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

class FakturaceAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.fakturace.site/v1';
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
const api = new FakturaceAPI('YOUR_API_KEY');

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