## Apigee Edge UI Authentication

This tool allows you to authenticate to the Apigee UI, and save the security HTTP headers
to disk. What's the point of this you ask? By doing this, you can use cURL to call the same
APIs that the Apigee UI uses. 

This is sometimes useful for automation purposes if you want to call Apigee Edge APIs for features that are still in alpha, or beta stage.

For all other Apigee Edge APIs, see the [Apigee docs](https://docs.apigee.com/reference/apis).

Note that this tool works with [Apigee Edge](https://docs.apigee.com/api-platform/). It does not work with Apigee Hybrid.

## Disclaimer
This tool is for experimental purposes. The undocumented Apigee Edge APIs, or even the login process can change at any time. Use it at your own risk.

## Prerequisites
 * Node.js (tested with v12.18.0)

## How to use the tool
* Install the tool from this repo
  ```shell script
  npm install -g https://github.com/micovery/apigee-edge-auth.git
  ```

* Login to the Apigee UI using your GCP credentials,
  
  ```shell script
  apigee-edge-auth.js -u yourusername@google.com -p SuperSecret123
  ```

  (this outputs a file called `auth.txt` which contains the Apigee security headers)
  
* Then, make a call to the Apigee APIs using curl

  This sample call retrieves the list of Apigee Developer portals.

  ```shell script
  curl -X GET https://apigee.com/portals/api/sites?orgname=YOUR_APIGEE_ORG \
    -H 'Accept: application/json' \
    -H '@auth.txt'
  ```

## How the tool works

The main script is module is  `apigee-edge-auth.js`. The script  works by using the [Puppeteer](https://pptr.dev/) library to automate the end-user login process. This is a library lets you 
navigate websites within a Chrome headless browser.

The login workflow is as follows:

  1. Visit https://login.apigee.com
  2. Enter the username
  3. Enter the password
  4. Click Sign In
  5. Wait for Apigee page to load
  
If all five steps are successful, then the script goes ahead and dumps the following HTTP headers to a file called `auth.txt`:

```shell script
x-requested-with: XMLHttpRequest
x-apigee-csrf: ... CSRF token scraped from the login.apigee.com site ...
cookie: ...all cookies from the login.apigee.com site ...
```
 

## API Call Samples

### Create API Product
```shell script
curl -X POST 'https://apigee.com/organizations/YOUR_APIGEE_ORG/apiproducts' \
  -H 'Content-Type: application/json' \
  -H 'Accept:application/json' \
  -H '@auth.txt' \
  -d '
{
  "name":"Product2",
  "displayName":"Product2",
  "description":"",
  "environments":["test"],
  "apiResources":[],
  "attributes":[{"name":"access","value":"internal"}],
  "approvalType":"auto",
  "proxies":[],
  "scopes":[]
}'
```

### List developer portals

```shell script
  curl -X GET https://apigee.com/portals/api/sites?orgname=YOUR_APIGEE_ORG \
    -H 'Accept: application/json' \
    -H '@auth.txt'
```
### Create a developer portal

```shell script
curl -X POST 'https://apigee.com/portals/api/sites' \
  -H 'Content-Type: application/json' \
  -H 'Accept:application/json' \
  -H '@auth.txt' \
  -d '
{
  "name":"test-portal",
  "description":"",
  "orgName":"YOUR_APIGEE_ORG",
  "portalVersion":2
}' 
```

### List API Products within Developer Portal
```shell script
curl -X GET 'https://apigee.com/portals/api/sites/SITE_ID/apidocs' \
  -H 'Accept:application/json' \
  -H '@auth.txt'
```

### Add API Product to Developer Portal

```shell script
curl -X POST 'https://apigee.com/portals/api/sites/SITE_ID/apidocs' \
  -H 'Content-Type: application/json' \
  -H 'Accept:application/json' \
  -H '@auth.txt' \
  -d '
{
  "title":"API Product2",
  "description":"",
  "edgeAPIProductName":"Product2",
  "visibility":true,
  "anonAllowed":true,
  "requireCallbackUrl":false,
  "specId":"petstore"
}'
```
### Add OpenAPI Spec snapshot to API Product in Developer Portal

```shell script
curl -X POST 'https://apigee.com/portals/api/sites/SITE_ID/apidocs/DOC_ID/snapshot' \
  -H 'content-type: text/plain' \
  -H 'Accept:application/json' \
  -H '@auth.txt' \
  --data-binary '@./petstore.yml'
```

## Not Google Product Clause

This is not an officially supported Google product.