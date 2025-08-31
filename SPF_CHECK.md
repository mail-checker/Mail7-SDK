# SPF Check API Documentation

## Overview

The SPF Check API provides a comprehensive service for validating and analyzing SPF (Sender Policy Framework) records for domains. This API helps domain owners, email administrators, and developers ensure their email authentication is properly configured by detecting common SPF misconfigurations and providing actionable recommendations.

## Base URL

```
https://mail7.net
```

## Authentication

This API is free to use and does not require authentication. However, rate limiting is applied to prevent abuse.

## Rate Limiting

- **Limit**: 10 requests per minute per IP address
- **Headers**: Rate limit information is provided in response headers
- **Status**: 429 Too Many Requests when limit is exceeded

## Endpoints

### SPF Record Check

#### GET /api/spf-check/{domain}

Validates and analyzes the SPF record for a specified domain.

**Parameters:**

| Parameter | Type   | Required | Description                    | Example        |
|-----------|--------|----------|--------------------------------|----------------|
| `domain`  | string | Yes      | Domain name to check           | `example.com`  |

**Example Request:**

```bash
curl -X GET "https://mail7.net/api/spf-check/example.com" \
     -H "Accept: application/json"
```

**Response Format:**

```json
{
  "domain": "example.com",
  "is_valid": true,
  "spf_record": "v=spf1 -all",
  "dns_lookups": 0,
  "syntax_valid": true,
  "has_soft_fail": false,
  "has_hard_fail": true,
  "issues": [],
  "recommendations": [
    "Consider implementing DKIM and DMARC alongside SPF for complete email authentication",
    "Regularly monitor your email deliverability metrics",
    "Test SPF changes in a staging environment before applying to production"
  ],
  "timestamp": "2025-08-31T19:32:41.907428"
}
```

**Response Fields:**

| Field              | Type    | Description                                                                 |
|--------------------|---------|-----------------------------------------------------------------------------|
| `domain`           | string  | The domain that was checked                                                 |
| `is_valid`         | boolean | Overall validity of the SPF record                                         |
| `spf_record`       | string  | The actual SPF record found in DNS                                         |
| `dns_lookups`      | integer | Number of DNS lookups required (max 10 allowed)                            |
| `syntax_valid`     | boolean | Whether the SPF record syntax is valid                                     |
| `has_soft_fail`    | boolean | Whether the record contains a soft fail mechanism (~all)                   |
| `has_hard_fail`    | boolean | Whether the record contains a hard fail mechanism (-all)                   |
| `issues`           | array   | List of detected issues and warnings                                       |
| `recommendations`  | array   | Actionable recommendations for improvement                                 |
| `timestamp`        | string  | ISO 8601 timestamp of when the check was performed                        |

**Issue Object Structure:**

```json
{
  "type": "warning",
  "message": "Missing All Mechanism",
  "description": "SPF record does not specify what to do with unauthorized servers",
  "recommendation": "Add ~all (soft fail) or -all (hard fail) at the end of your SPF record",
  "severity": 2
}
```

**Issue Types:**

| Type      | Description                                    | Severity |
|-----------|------------------------------------------------|----------|
| `error`   | Critical issues that must be fixed            | 3        |
| `warning` | Issues that should be addressed               | 2        |

## Common SPF Issues Detected

### 1. Missing SPF Version
- **Issue**: SPF record doesn't start with `v=spf1`
- **Impact**: Record will be ignored by email servers
- **Fix**: Ensure record begins with `v=spf1`

### 2. Too Many DNS Lookups
- **Issue**: SPF record causes more than 10 DNS lookups
- **Impact**: Email servers may reject emails
- **Fix**: Consolidate include mechanisms or use subdomains

### 3. Missing All Mechanism
- **Issue**: No `~all` or `-all` mechanism specified
- **Impact**: Unclear policy for unauthorized servers
- **Fix**: Add `~all` (soft fail) or `-all` (hard fail)

### 4. Invalid Mechanisms
- **Issue**: Unknown or malformed SPF mechanisms
- **Impact**: Record may be ignored or cause errors
- **Fix**: Use only valid SPF mechanisms

### 5. Multiple SPF Records
- **Issue**: Multiple TXT records with `v=spf1`
- **Impact**: Inconsistent behavior across email servers
- **Fix**: Consolidate into single SPF record

## Supported SPF Mechanisms

| Mechanism | Description                                    | DNS Lookup |
|-----------|------------------------------------------------|------------|
| `all`     | Matches all IPs not matched by other mechanisms | No         |
| `include` | Include SPF record from another domain         | Yes        |
| `a`       | Match IPs from domain's A records              | Yes        |
| `mx`      | Match IPs from domain's MX records             | Yes        |
| `ptr`     | Match IPs from domain's PTR records            | Yes        |
| `exists`  | Match if domain exists                         | Yes        |
| `ip4`     | Match specific IPv4 address or range           | No         |
| `ip6`     | Match specific IPv6 address or range           | No         |
| `redirect`| Redirect to another domain's SPF record        | No         |

## Usage Examples

### JavaScript

```javascript
async function checkSPF(domain) {
  try {
    const response = await fetch(`https://mail7.net/api/spf-check/${domain}`);
    const data = await response.json();
    
    if (data.is_valid) {
      console.log(`✅ SPF record for ${domain} is valid`);
    } else {
      console.log(`❌ SPF record for ${domain} has issues:`);
      data.issues.forEach(issue => {
        console.log(`  - ${issue.message}: ${issue.recommendation}`);
      });
    }
    
    return data;
  } catch (error) {
    console.error('Error checking SPF:', error);
  }
}

// Usage
checkSPF('example.com');
```

### Python

```python
import requests

def check_spf(domain):
    try:
        response = requests.get(f"https://mail7.net/api/spf-check/{domain}")
        response.raise_for_status()
        
        data = response.json()
        
        if data['is_valid']:
            print(f"✅ SPF record for {domain} is valid")
        else:
            print(f"❌ SPF record for {domain} has issues:")
            for issue in data['issues']:
                print(f"  - {issue['message']}: {issue['recommendation']}")
        
        return data
        
    except requests.exceptions.RequestException as e:
        print(f"Error checking SPF: {e}")
        return None

# Usage
check_spf('example.com')
```

### PHP

```php
<?php
function checkSPF($domain) {
    $url = "https://mail7.net/api/spf-check/" . urlencode($domain);
    
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, ['Accept: application/json']);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    if ($httpCode === 200) {
        $data = json_decode($response, true);
        
        if ($data['is_valid']) {
            echo "✅ SPF record for {$domain} is valid\n";
        } else {
            echo "❌ SPF record for {$domain} has issues:\n";
            foreach ($data['issues'] as $issue) {
                echo "  - {$issue['message']}: {$issue['recommendation']}\n";
            }
        }
        
        return $data;
    } else {
        echo "Error: HTTP {$httpCode}\n";
        return null;
    }
}

// Usage
checkSPF('example.com');
?>
```

### cURL

```bash
# Basic check
curl "https://mail7.net/api/spf-check/example.com"

# Pretty-printed JSON
curl "https://mail7.net/api/spf-check/example.com" | python -m json.tool

# Save to file
curl "https://mail7.net/api/spf-check/example.com" > spf_result.json
```

## Error Responses

### 400 Bad Request
```json
{
  "detail": "Invalid domain format"
}
```

### 429 Too Many Requests
```json
{
  "detail": "Rate limit exceeded"
}
```

### 500 Internal Server Error
```json
{
  "detail": "An error occurred while processing your request"
}
```

## Best Practices

### 1. Rate Limiting
- Implement exponential backoff when hitting rate limits
- Cache results for frequently checked domains
- Batch requests when possible

### 2. Error Handling
- Always check HTTP status codes
- Handle network errors gracefully
- Provide user-friendly error messages

### 3. Caching
- Cache SPF check results for at least 1 hour
- Respect DNS TTL values when available
- Implement cache invalidation for failed checks

## Testing

### Test Domains

| Domain        | Expected Result                    | Notes                    |
|---------------|-----------------------------------|--------------------------|
| `example.com` | Valid SPF record                  | Standard test domain     |
| `gmail.com`   | Valid with redirect               | Complex SPF setup        |
| `microsoft.com`| Valid with multiple includes     | Enterprise configuration |

### Validation Checklist

- [ ] SPF record starts with `v=spf1`
- [ ] DNS lookup count ≤ 10
- [ ] Contains `~all` or `-all` mechanism
- [ ] No syntax errors
- [ ] No duplicate SPF records
- [ ] All mechanisms are valid

## Support

For questions, issues, or feature requests:

- **Website**: [https://mail7.net](https://mail7.net)
- **API Status**: Check response headers for rate limit info
- **Documentation**: [https://mail7.net/api-docs.html](https://mail7.net/api-docs.html)

## License

This API is provided as a free service. Please respect rate limits and terms of service.

---

*Last updated: August 31, 2025*
*API Version: 1.0*
