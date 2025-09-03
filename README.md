# Schoology API Guide

A comprehensive guide explaining how to use the Schoology REST API. This guide is inspired by and serves as an in-depth extension to the tutorial by [Liamq12](https://github.com/Liamq12/Schoology-API).

---

## Overview

The Schoology API is an interface that allows applications to communicate with Schoology's server and retrieve data through HTTP requests. Unlike many modern APIs, Schoology uses the OAuth 1.0a authentication method. To follow along quickly, you can use [Postman](https://www.postman.com/) to easily make these requests.

This guide covers **two methods** for accessing Schoology data:
- **Method 1 (OAuth 1.0a)**: Full API access with proper authentication
- **Method 2 (Cookie-based)**: Simplified access using session cookies

---

## Important Considerations

### Access Levels
- **Public Access**: Using only `consumer_key` and `consumer_secret` limits you to publicly available information
- **Authenticated Access**: Requires OAuth flow or cookies to access sensitive data like grades
- **Developer Access**: For accessing the most sensitive information, you may need to apply for Schoology developer access or implement LTI (Learning Tools Interoperability)

### OAuth Callback Requirements
When accessing sensitive information through OAuth, the `oauth_callback` parameter requires Schoology developer dashboard access to create a valid callback URL for retrieving the `oauth_verifier`.

---

## Method 1: OAuth 1.0a Authentication

### Step 1: Obtain API Credentials

First, retrieve your `consumer_key` and `consumer_secret`:
1. Log in to your school's Schoology website
2. Add `/api` to the end of the URL (e.g., `https://lms.lausd.net/api`)
3. Note down your consumer key and secret

### Step 2: Request Token

Make a GET request to obtain the request token:

```
https://api.schoology.com/v1/oauth/request_token?oauth_consumer_key=YOUR_KEY&oauth_timestamp=TIMESTAMP&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=UNIQUE_STRING&oauth_signature=YOUR_SECRET%26
```

**Parameters explained:**
- `oauth_consumer_key`: Your consumer key from Step 1
- `oauth_timestamp`: Current Unix timestamp (`Math.floor(Date.now() / 1000)`)
- `oauth_signature_method`: Use `PLAINTEXT` or `SHA-256`
- `oauth_version`: Always `1.0` for Schoology
- `oauth_nonce`: A unique string for each request
- `oauth_signature`: Your `consumer_secret` + `%26`

**Response:** You'll receive `oauth_token`, `oauth_token_secret`, and `xoauth_token_ttl`.

### Step 3: User Authorization

Direct users to the authorization URL:

```
https://[YOUR_SCHOOL_DOMAIN]/oauth/authorize?oauth_consumer_key=YOUR_KEY&oauth_token=REQUEST_TOKEN&oauth_token_secret=REQUEST_TOKEN_SECRET&oauth_callback=CALLBACK_URL
```

**Parameters:**
- `oauth_consumer_key`: Your consumer key
- `oauth_token`: Token from Step 2
- `oauth_token_secret`: Token secret from Step 2
- `oauth_callback`: URL to redirect after authorization

After authorization, you'll receive an `oauth_verifier`.

### Step 4: Access Token

Make a POST request to get your access token:

```
https://api.schoology.com/v1/oauth/access_token?oauth_consumer_key=YOUR_KEY&oauth_token=REQUEST_TOKEN&oauth_signature_method=PLAINTEXT&oauth_timestamp=TIMESTAMP&oauth_nonce=NONCE&oauth_signature=SIGNATURE&oauth_verifier=VERIFIER
```

**Response:** You'll receive `oauth_token` (access token) and `oauth_token_secret` (access token secret).

### Step 5: Making API Requests

#### Public Information (Steps 1-2 only)
```
https://api.schoology.com/v1/sections/COURSE_ID?oauth_consumer_key=YOUR_KEY&oauth_timestamp=TIMESTAMP&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=NONCE&oauth_signature=YOUR_SECRET%26
```

#### Authenticated Requests (After Step 4)
```
https://api.schoology.com/v1/sections/COURSE_ID?oauth_consumer_key=YOUR_KEY&oauth_token=ACCESS_TOKEN&oauth_timestamp=TIMESTAMP&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=NONCE&oauth_signature=CONSUMER_SECRET%26ACCESS_TOKEN_SECRET
```

**Note:** The signature for authenticated requests is `consumer_secret&access_token_secret`.

---

## Method 2: Cookie-Based Access

This method is simpler and involves obtaining the user's session cookie through login and sending GET requests through the internal API (iapi) with the cookie in headers.

### Step 1: Obtain Session Cookie

You can obtain cookies manually or programmatically. Here's an example using the [schoology-wrapper npm package](https://github.com/i-nek/Schoology-Wrapper) that utilizes Puppeteer:

```javascript
import SchoologyClient from 'schoology-wrapper';

const schoologyClient = new SchoologyClient(
    'consumer_key', 
    'consumer_secret',
    'https://lms.lausd.net/' // Your domain
);

// Retrieve Cookie
const data = await schoologyClient.requestCookie("Authentication required");
// Returns: [sidName, sidValue, site]

// Use cookie for API requests
const notifications = await schoologyClient.getNotifications(data[0], data[1], data[2]);
console.log(notifications);
```

### Step 2: Making Cookie-Based Requests

Example implementation for getting notifications:

```javascript
async function getNotifications(name, value, site) {
    const result = await axios.request({
        method: "GET",
        url: `${site}/iapi2/site-navigation/notifications`,
        headers: {
            "Cookie": `${name}=${value}`
        }
    });
    return result.data;
}
```

---

## Common API Endpoints

### Courses
- `/sections/COURSE_ID` - Get course information
- `/sections/COURSE_ID/assignments` - Get course assignments
- `/sections/COURSE_ID/folders` - Get course folders
- `/sections/COURSE_ID/updates` - Get course updates
- `/sections/COURSE_ID/enrollments` - Get course members

### Groups
- `/groups/GROUP_ID` - Get group information
- `/groups/GROUP_ID/members` - Get group members
- `/groups/GROUP_ID/updates` - Get group updates
- `/groups/GROUP_ID/resources` - Get group resources

### Users
- `/users/USER_ID` - Get user information
- `/users/me` - Get current user information (authenticated requests only)

For more endpoints, refer to the [Schoology API Documentation](https://developers.schoology.com/api-documentation/example-requestsresponses/).

---

## Error Handling

Common HTTP status codes and their meanings:

| Status Code | Meaning | Likely Cause |
|-------------|---------|--------------|
| `400` | Bad Request | Syntax issues in your request |
| `401` | Unauthorized | Invalid `consumer_key` or `consumer_secret` |
| `403` | Forbidden | Missing permissions; may require `access_token` |
| `404` | Not Found | Incorrect URL endpoint or resource doesn't exist |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server-side issue |

---

## Best Practices

1. **Rate Limiting**: Respect Schoology's rate limits to avoid being blocked
2. **Error Handling**: Always implement proper error handling for failed requests
3. **Security**: Never expose your `consumer_secret` or access tokens in client-side code
4. **Caching**: Cache responses when appropriate to reduce API calls
5. **Testing**: Use Postman or similar tools to test requests before implementing

---

## Conclusion

This guide provides two approaches to accessing Schoology data:
- **OAuth 1.0a** for comprehensive, secure access
- **Cookie-based** for simpler implementation with session-based authentication

For easier implementation, consider using the [schoology-wrapper npm package](https://github.com/i-nek/Schoology-Wrapper), which simplifies the authentication process and provides a clean interface for common operations.

---

## Additional Resources

- [Schoology API Documentation](https://developers.schoology.com/)
- [OAuth 1.0a Specification](https://tools.ietf.org/html/rfc5849)
- [Postman](https://www.postman.com/) for API testing
- [Unix Timestamp Converter](https://www.unixtimestamp.com/)
