# Mail7-SDK

Mail7-SDK provides simple and reliable email validation APIs that can be easily integrated into your applications.  
This SDK supports **single** and **bulk** email validation across multiple programming languages.

---

## Features

- ✅ Validate a single email address  
- ✅ Validate multiple email addresses in bulk  
- ✅ Easy integration with JavaScript, Python, and PHP  
- ✅ JSON responses with validation details  
- ✅ RESTful endpoints, simple HTTP requests  

---

## API Endpoints

- **Single Validation:** `POST https://mail7.net/api/validate-single`  
- **Bulk Validation:** `POST https://mail7.net/api/validate-bulk`  

---

## JavaScript / Node.js

### Single Email Validation
```js
const response = await fetch('https://mail7.net/api/validate-single', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    email: 'user@example.com'
  })
});

const result = await response.json();
console.log(result);
```

### Bulk Validation
```js
const formData = new FormData();
formData.append('emails', 'user1@example.com\nuser2@example.com');

const bulkResponse = await fetch('https://mail7.net/api/validate-bulk', {
  method: 'POST',
  body: formData
});

const bulkResult = await bulkResponse.json();
console.log(bulkResult);
```

---

## Python

### Single Email Validation
```python
import requests

response = requests.post(
    'https://mail7.net/api/validate-single',
    json={'email': 'user@example.com'}
)
result = response.json()
print(result)
```

### Bulk Validation
```python
import requests

emails = "user1@example.com\nuser2@example.com"
files = {'emails': (None, emails)}

response = requests.post('https://mail7.net/api/validate-bulk', files=files)
result = response.json()
print(result)
```

---

## PHP

### Single Email Validation
```php
$data = ['email' => 'user@example.com'];
$options = [
    'http' => [
        'header'  => "Content-type: application/json\r\n",
        'method'  => 'POST',
        'content' => json_encode($data)
    ]
];

$context = stream_context_create($options);
$result = file_get_contents('https://mail7.net/api/validate-single', false, $context);
$response = json_decode($result, true);
print_r($response);
```

---

## Response Format

All responses are returned in JSON. Example:

```json
{
  "email": "user@example.com",
  "is_valid": true,
  "is_disposable": false,
  "is_role_based": false,
  "domain": "example.com",
  "mx_records": true
}
```

---

## Error Handling

Typical error responses:

```json
{
  "error": "Invalid request",
  "message": "Email address is missing or malformed"
}
```

---

## Best Practices

- Always check the `is_valid` field before using an email address.  
- For bulk validation, pass emails as newline-separated values.  
- Implement retries or error handling for network failures.  
- Cache results where appropriate to reduce repeated API calls.  

---

## License

This SDK is released under the **MIT License**.  
Feel free to use, modify, and contribute.

---

## Support

- 📧 Email: info@mail7.net  
- 🌐 Website: [https://mail7.net](https://mail7.net)  

For bugs or feature requests, please open an issue in this repository.
