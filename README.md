# Schoology API Guide
A guide explaining how to use the Schoology REST API. This guide is inspired by and serves as an in-depth extension to the tutorial by [Liamq12](https://github.com/Liamq12/Schoology-API).

## Important Note
The method discussed here requires only your `consumer_key` and `consumer_secret`. Therefore, access will be limited to what Schoology allows as public. If you want to access more confidential information, such as grades, additional steps are required. You may either use Schoology's LTI (Learning Tools Interoperability), which is typically used to create apps available alongside your class tools (e.g., Kami), or apply for Schoology developer access, which is strict and may be unlikely for normal users.

## Info
The Schoology API is an interface that allows applications to communicate with Schoology's server and retrieve data through HTTP requests. Unlike many modern APIs, Schoology uses the OAuth 1.0a authentication method. To follow along quickly, you can use Postman to easily make these requests.
## Step 1: API Credentials
The first step is retrieving your `consumer_key` and `consumer_secret`. To obtain them, simply log in to your school's website and add `/api` to the end of the URL: `https://lms.lausd.net/api`.

## Step 2: Request Token
After obtaining your API credentials, you need to get the request token required for the Authorization URL. To get this, make a GET request to the following link:

```
https://api.schoology.com/v1/oauth/request_token?oauth_consumer_key=&oauth_timestamp=&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=&oauth_signature=
```
- `https://api.schoology.com/v1/`: This is the base URL for retrieving data.
- `oauth/request_token?`: This is the endpoint for getting the request token.
- `oauth_consumer_key=`: The `oauth_consumer_key` is the `consumer_key` you obtained in the previous step.
- `oauth_timestamp=`: The `oauth_timestamp` is simply your current date and time in Unix time format. You can get this by either doing `Math.floor(Date.now() / 1000)` or using a [website](https://www.unixtimestamp.com/#:~:text=Epoch%20and%20unix%20timestamp%20converter%20for%20developers.%20Date%20and) that provides this.
- `oauth_signature_method=PLAINTEXT`: The signature method is a unique identifier used for each request, which can be either "PLAINTEXT" or "SHA-256".
- `oauth_version=1.0`: This indicates the version of OAuth that Schoology uses (1.0a).
- `oauth_nonce=`: Similar to the signature, the nonce is a unique string needed for each request.
- `oauth_signature=`: The `oauth_signature` is your `consumer_secret` + "&26".

After making the GET request, you should receive a status 200 along with three items: `oauth_token`, `oauth_token_secret`, and `xoauth_token_ttl`.

## Step 3: Authorization URL
Once you have your OAuth tokens, you can work on authorizing. To do this, use the following link and paste it into your browser or redirect to the URL and follow the authorization process:

```
https://['your school's domain']/oauth/authorize?oauth_consumer_key=&oauth_token=&oauth_token_secret=
```
- `oauth_consumer_key`: This is your `consumer_key` from Step 1.
- `oauth_token`: This is the token you received from Step 2.
- `oauth_token_secret=`: This is the token secret you received from Step 2.

After authorizing, you can start making requests to Schoology.

## Step 4: API Requests
To send an API request, make a GET request to the following link:

```
https://api.schoology.com/v1/sections/['course_id']?oauth_consumer_key=&oauth_timestamp=&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=&oauth_signature=
```
- `sections/`: Specifies that you want to access the sections of a course.
- `['course_id']`: Replace this placeholder with the actual ID of the course you want to retrieve sections for. This ID is typically a numeric or alphanumeric string unique to that specific course.

After sending your GET request, you should receive some JSON containing all the information related to the course. You can also change the endpoint to other endpoints such as `/sections/['course_id']/assignments?`, `/groups`, and many more. You can refer to the [Example Requests/Responses](https://developers.schoology.com/api-documentation/example-requestsresponses/) on the Schoology API documentation.

## Common Errors
In the unlikely event you encounter an error, here are some common HTTP status codes and their meanings:
- `400`: Bad Request; likely due to syntax issues.
- `401`: Unauthorized; this could indicate an issue with your `consumer_key` or `consumer_secret`.
- `403`: Forbidden; you do not have permission to access this resource, which typically requires an `access_token`.
- `404`: Not Found; this usually occurs due to an incorrect URL endpoint.

## Conclusion
This guide provides a comprehensive overview of using the Schoology API. I would also like to include my [npm package](https://github.com/i-nek/Schoology-Wrapper) which makes implementing the schoology API 100% easier and only requires a few lines of code use.
