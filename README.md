# Schoology API Guide
A guide explaining how to use the Schoology REST API. This guide is inspired by and serves as an in-depth extension to the tutorial by [Liamq12](https://github.com/Liamq12/Schoology-API).

## Important Note
This guide discusses two methods in obtaining information from schoology.The first method involving taking the user's token refer and simply accessing informationn as the user refer to [Method 2](https://github.com/i-nek/Schoology-API-Guide/edit/main/README.md#method-2) while the second method requires only your `consumer_key` and `consumer_secret`. Therefore, access will be limited to what Schoology allows as public. If you want to access more confidential information, such as grades, additional steps are required if not you may skip Steps 2-5 as you are simply accessing public information or use the first method of using the user's token. You may alsoo either use Schoology's LTI (Learning Tools Interoperability), which is typically used to create apps available alongside your class tools (e.g., Kami), or apply for Schoology developer access, which is strict and may be unlikely for normal users. Lastly, when attempting for sensitive information and authorizing, the `oauth_callback` will require schoology developer as their dashboard should include creating a callback url. This is so you can retrieve the `oauth_verifer`.

## Info
The Schoology API is an interface that allows applications to communicate with Schoology's server and retrieve data through HTTP requests. Unlike many modern APIs, Schoology uses the OAuth 1.0a authentication method. To follow along quickly, you can use [Postman](https://www.postman.com/) to easily make these requests.
# Method 1
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
- `oauth_signature=`: The `oauth_signature` is your `consumer_secret` + `%26`.

After making the GET request, you should receive a status 200 along with three items: `oauth_token`, `oauth_token_secret`, and `xoauth_token_ttl`.

## Step 3: Authorization URL
Once you have your OAuth tokens, you can work on authorizing. To do this, use the following link and paste it into your browser or redirect to the URL and follow the authorization process:

```
https://['your school's domain']/oauth/authorize?oauth_consumer_key=&oauth_token=&oauth_token_secret=&oauth_callback=
```
- `oauth_consumer_key`: This is your `consumer_key` from Step 1.
- `oauth_token`: This is the token you received from Step 2.
- `oauth_token_secret=`: This is the token secret you received from Step 2.
- `oauth_callback`: This is usually a url that redircts the user back to your application with the `oauth_token` and `oauth_verifier`.

After authorizing, you can start making a request for your access_token.

## Step 5: Access_token
To retrieve your access_token make a POST request to the following link:
```
https://api.schoology.com/v1/oauth/access_token?oauth_consumer_key=consumer_key&oauth_token=request_token&oauth_signature_method=PLAINTEXT&oauth_timestamp=timestamp&oauth_nonce=nonce&oauth_signature=signature&oauth_verifier=verifier
```
- `oauth_token`: The token you got after Step 4.
- `oauth_verifier`: The verifier you got after Step 4.

Once you make the post request you should recieve two things: `oauth_token`(access_token) and `oauth_token_secret`(access_token_secret). Refer to 6.1 after

## Step 6: API Requests
To send an API request, make a GET request to the following link:

```
https://api.schoology.com/v1/sections/['course_id']?oauth_consumer_key=&oauth_timestamp=&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=&oauth_signature=
```
- `sections/`: Specifies that you want to access the sections of a course.
- `['course_id']`: Replace this placeholder with the actual ID of the course you want to retrieve sections for. This ID is typically a numeric or alphanumeric string unique to that specific course.

After sending your GET request, you should receive some JSON containing all the information related to the course. You can also change the endpoint to other endpoints such as `/sections/['course_id']/assignments?`, `/groups`, and many more. You can refer to the [Example Requests/Responses](https://developers.schoology.com/api-documentation/example-requestsresponses/) on the Schoology API documentation.

## Step 6.1 API Request with access_token
The step here is similar to Step 6 with the addition of adding the `oauth_token` and `oauth_token_secret`. Having these two items allows access to more sensitive information as schoology directly approves your application.
```
https://api.schoology.com/v1/sections/course_id?oauth_consumer_key=consumer_key&oauth_token=access_token&oauth_timestamp=timestamp&oauth_signature_method=PLAINTEXT&oauth_version=1.0&oauth_nonce=nonce&oauth_signature=consumer_secret&access_token_secret
```
- `oauth_token`: This is the `access_token` from Step 5.
- `oauth_signature`: Unlike in Step 6, the signature will be `consumer_secret&oauth_token_secret` which are from Step 1 and 5.
# Method 2
The first method is much more simpler and is the process of obtaining the user's cookie through a login and sending a GET request through the iapi with the cookie in the headers (`name=value`).
## Step 1
There are many ways to obtain one's cookie either manually or through coding. The method I will be showcasing will be through my [npm package](https://github.com/i-nek/Schoology-Wrapper) which utilizes puppeteer. The proccess in short, opens the login page and when the user logins, obtains their cookie and closes their browswer which can be used to send a request.

```javascript
import SchoologyClient from 'schoology-wrapper';

const schoologyClient = new SchoologyClient(
    'consumer_key', 
    'consumer_secret',
    'domain', // example: https://lms.lausd.net/ 
);

// Retrieve Cookie
const data = await schoologyClient.requestCookie("reason");
// The requestCookie returns 3 values: the sidName, sidValue, site,
const notifications = await schoologyClient.getNotifications(data[1], data[2], data[3]);
console.log(notifications);
);
```
getNotifications() 
```javascript
async function getNotifications(name, value, site) {
    const result = await axios.request({
        method: "GET",
        url: `${site}/iapi2/site-navigation/notifications`,
        headers: {
            "Cookie": `${name}=${value}`
        }
    })
    return result.data
}
```

## Common Errors
In the unlikely event you encounter an error, here are some common HTTP status codes and their meanings:
- `400`: Bad Request; likely due to syntax issues.
- `401`: Unauthorized; this could indicate an issue with your `consumer_key` or `consumer_secret`.
- `403`: Forbidden; you do not have permission to access this resource, which typically requires an `access_token`.
- `404`: Not Found; this usually occurs due to an incorrect URL endpoint.

## Conclusion
This guide provides a comprehensive overview of using the Schoology API. I would also like to include my [npm package](https://github.com/i-nek/Schoology-Wrapper) which makes implementing the schoology API 100% easier and only requires a few lines of code use.
