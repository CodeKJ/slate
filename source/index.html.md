---
title: DISIFY API Docs

language_tabs: # must be one of https://git.io/vQNgJ
  - php
  - csharp
  - java

search: true
---

# Introduction

Welcome to the [DISIFY](https://disify.com) API documentation!

DISIFY is free email validation API which also includes validation for disposable (DEA) or temporary email addresses.

Here you will find some basic request and response examples, tips, best practices and use cases.

Documentation is still in development process and will be updated with more features and examples. Check back soon.

* Website: [disify.com](https://disify.com)
* GitHub: [https://github.com/CodeKJ/DISIFY](https://github.com/CodeKJ/DISIFY)

# Quick Tips

* If you request only domain validation for your emails we strongly recommend to make some general whitelist on your side before making request to our API.
For example: gmail.com, hotmail.com, yahoo.com and other well known email providers will be always considered as valid and non-disposable, so you don't really need to check them on our API - save your and our resources.
* Large amount of requests in short time interval are throttled. Suspicious requests might even lead to permanent IP ban.
* If you are having issues feel free to contact us and open issue in our [GitHub page](https://github.com/CodeKJ/DISIFY).
* This is free service single-handedly made just for passion. No terms, no obligations, no cost.

# Single Email Address

> Example POST request

```php
<?php
$curl = curl_init();

$data = [
    'email' => 'your@example.com',
];

$post_data = http_build_query($data);

curl_setopt_array($curl, array(
    CURLOPT_URL => "https://disify.com/api/email",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CUSTOMREQUEST => "POST",
    CURLOPT_POSTFIELDS => $post_data,
));

$response = curl_exec($curl);
$err = curl_error($curl);

curl_close($curl);

if ($err) {
    echo "cURL Error #:" . $err;
} else {
    echo $response;
}
```
```csharp
var client = new RestClient("https://disify.com/api/email");
var request = new RestRequest(Method.POST);
request.AddParameter("email", "your@example.com");
IRestResponse response = client.Execute(request);
```
```java
OkHttpClient client = new OkHttpClient();

MediaType mediaType = MediaType.parse("application/x-www-form-urlencoded");
RequestBody body = RequestBody.create(mediaType, "email=your@example.com");
Request request = new Request.Builder()
  .url("https://disify.com/api/email")
  .post(body)
  .build();

Response response = client.newCall(request).execute();
```

> Response example

```json
{
    "format": true,
    "alias": true,
    "domain": "example.com",
    "disposable": false,
    "dns": true
}
```

### Description

Validates single email address or domain.

### GET Request

Requires just <code>email</code> parameter and email address after slash (or domain).

`GET https://disify.com/api/email/your@example.com`

`GET https://disify.com/api/domain/example.com`

### POST Request

`POST https://disify.com/api/email`

`POST https://disify.com/api/domain`

### POST Parameters

Parameter | Type | Description
--------- | --------- | -----------
email | string | Email address to check
domain | string | Domain name to check

<aside class="notice">Remember to use either <code>email</code> OR <code>domain</code> parameter accordingly, no need to use both parameters at same time.</aside>

### JSON Response Parameters

Parameter | Type | Description
--------- | --------- | -----------
format | boolean | Check if email has valid format
alias | boolean | Check if email contains "+" symbol
domain | string | Returns top-level domain (TLD) of given email
disposable | boolean | Check if email has been detected as disposable
dns | boolean | Check if email MX records are live

<aside class="notice">If parameter is non-existent it's considered as <code>false</code>. For example, <code>alias</code> parameter won't be provided if email doesn't contain alias symbol.</aside>

# Mass Emails & Domains

> Example POST request

```php
<?php
$curl = curl_init();

$emails = [
    'your@example.com',
    'another@mail.com'
];

$data = [
    'bulk' => true,
    'email' => implode(',', $emails),
];

$post_data = http_build_query($data);

curl_setopt_array($curl, array(
    CURLOPT_URL => "https://disify.com/api/email",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CUSTOMREQUEST => "POST",
    CURLOPT_POSTFIELDS => $post_data,
));

$response = curl_exec($curl);
$err = curl_error($curl);

curl_close($curl);

if ($err) {
    echo "cURL Error #:" . $err;
} else {
    echo $response;
}
```

> Response example

```json
{
    "total": 2,
    "invalid_format": 0,
    "invalid_dns": 0,
    "disposable": 0,
    "unique": 2,
    "valid": 1,
    "session": "d117271ce938bf91bc718f6cfb7954de"
}
```

### Description

Validates bulk of email addresses or domain names.

### GET Request

Insert all emails/domains as single string right after <code>email</code> parameter (after slash). Each email/domain will be automatically recognized by comma, space or new line.
Then simply attach parameter <code>mass</code> (or <code>bulk</code> alias) after email list slash.

If you want to mass check domains, simply replace parameter <code>email</code> with <code>domain</code> inside URL.

`GET https://disify.com/api/email/your@example.com,another@mail.com/mass`

### POST Request

Same request structure as single email address, except you will need to provide additional parameter <code>bulk</code>

`POST https://disify.com/api/email`

### POST Parameters

Parameter | Type | Description
--------- | --------- | -----------
bulk | boolean | Required in order to make mass/bulk email validation
email | string | Bulk of email address to check (seperated by comma, whitespace or new-line)
domain | string | Bulk of domain names to check (seperated by comma, whitespace or new-line)

<aside class="notice">Remember to use either <code>email</code> OR <code>domain</code> parameter accordingly, no need to use both parameters at same time.</aside>

### JSON Response Parameters

Parameter | Type | Description
--------- | --------- | -----------
total | integer | Total number of provided emails or domains
invalid_format | integer | Number of emails/domains with invalid format
invalid_dns | integer | Number of emails/domains with unresolvable MX records
disposable | integer | Number of emails/domains detected in our internal blacklists
unique | integer | Number of unique and valid emails/domains
valid | integer | Number of valid emails/domains
session | string | Given hash string to temporary access request results with valid emails/domains

# View Valid Mass Results

> Response example #1

```text
your@example.com
another@example.com
```
> Response example #2

```text
your@example.com,another@example.com
```

### Description

Returns valid email list of previously requested bulk emails/domains.

### GET Request

Use previously returned session value (from mass email/domain check) after URL parameter view.

Optional parameters after session value:
<code>download</code> will trigger force download.
<code>separate</code> (or <code>comma</code>, <code>separator</code> aliases) will display emails in single string separated by comma instead of new-line. 

`GET https://disify.com/api/view/d117271ce938bf91bc718f6cfb7954de`

### Response

Plain-text list of emails/domains seperated by comma or new-line.

# Blacklists

You can check our internal domain and MX blacklists here.

* [Domain blacklist](https://disify.com/blacklist/domains)
* [MX blacklist](https://disify.com/blacklist/mx)
