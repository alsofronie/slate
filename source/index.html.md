---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - http

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slates</a>

includes:
  - models
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Awesome API
---

# Introduction

Welcome to our awesome API.

This API documentation page was created with [Slate](https://github.com/slatedocs/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Authentication

## With credentials

```http
POST /api/v1/authenticate HTTP/1.1
Content-Type: application/json
Accept: application/json
{
  "username": "root",
  "password": "secret",
  "grant_type": "token",
  "scope": "root"
}

200 OK HTTP/1.1
{
  "access_token": "<your_access_token>",
  "refresh_token": "<your_refresh_token>",
  "expires": 123155100
  "type": "bearer"
}
```

`POST /api/v1/authenticate`

This endpoint provides you with an `access_token` that you can use for all subsequent requests.

### Request Body

| Name         | Required | Description                                     |
| ------------ | -------- | ----------------------------------------------- |
| "username"   | yes      | Your username or `"root"`                       |
| "password"   | yes      | Your password or `"secret"` for a fresh install |
| "grant_type" | yes      | Use `"token"` to get an access token            |
| "scope"      | yes      | Can be `root`, `admin` or `user`                |

<aside class="warning">
With a fresh install, you will be using a default set of credentials in order to make your initial setup. Please use `root` as your username (this will not change), and `secret` as your password. You won't be able to call other API endpoints with an access token generated with the `secret` as the password. You **must** change your `root` password.
</aside>

## Refresh tokens

```http
PUT /api/v1/authenticate HTTP/1.1
Content-Type: application/json
Accept: application/json
{
  "refresh_token": "<your_refresh_token>
  "grant_type": "token",
  "scope": "root"
}

200 OK HTTP/1.1
{
  "access_token": "<your_access_token>",
  "refresh_token": "<your_refresh_token>",
  "expires": 123155100
  "type": "bearer"
}
```

`PUT /api/v1/authenticate`

While the access token is pretty short lived (about 8 hours), the refresh token is valid for 30 days. When the access token expires, you need to refresh it based on the refresh token obtained in authentication flow. If the refresh token is also expired, you need to authenticate with the username and the password.

### Request Body

| Name            | Required | Description                                       |
| --------------- | -------- | ------------------------------------------------- |
| "refresh_token" | yes      | The refresh token obtained in authentication flow |
| "grant_type"    | yes      | Use `"token"` to get an access token              |
| "scope"         | yes      | Can be `root`, `admin` or `user`                  |

<aside class="notice">If the refresh token is past 10 days, you will also get a new refresh token. Otherwise, the old refresh token will be returned unchanged.
</aside>

## Impersonate an user

```http
POST /api/v1/impersonate HTTP/1.1
Authorization: <your_access_token>
Content-Type: application/json
Accept: application/json
{
  "user_id": "<identifier>",
  "grant_type": "token",
  "scope": "user"
}

200 OK HTTP/1.1
{
  "access_token": "<access_token>",
  "refresh_token": "<refresh_token>",
  "expires": 123155100
  "type": "bearer"
}
```

`POST /api/v1/impersonate`

Please note this endpoint is available only for access tokens scoped to `root` or `admin`.

You might need to obtain an access token for a different user. In order to do that, you can use the described endpoint to get an access token for a certain user

### Request body

| Name       | Type   | Description                                                                             |
| ---------- | ------ | --------------------------------------------------------------------------------------- |
| user_id    | string | Identify the user you wish to impersonate. It can be either the user id or the username |
| grant_type | string | Always `"token"`                                                                        |
| scope      | string | Can be `"admin"` or `"user"`                                                            |

<aside class="notice">You cannot impersonate root accounts. You cannot impersonate you own account. If your access token is generated with a root scope, you can impersonate admins and users. If your access token is admin scoped, you can only impersonate users.</aside>

# Profile

## Change own profile

```http
PUT /api/v1/profile HTTP/1.1
Content-Type: application/json
Authorization: Bearer _your_access_token_
Accept: application/json
{
  "email": "your@email",
  "password": "_your_supersecret_password_",
  "first_name": "Your first name",
  "last_name": "Your last name"
}

204 No Content HTTP/1.1
```

`PUT /api/v1/profile`

This endpoint allows you to change your Personal data. This data is never shared with anyone and it is solely used to be able to communicate with you, informing about the status of your server, errors, downtimes, new features and so on.

### Body

| Name         | Required | Description               |
| ------------ | -------- | ------------------------- |
| "email"      | no \*    | Use your email            |
| "username"   | no \*    | Username                  |
| "password"   | no \*    | For changing the password |
| "first_name" | no \*    | Update your first name    |
| "last_name"  | no \*    | Update your last name     |

<aside class="warning">* With a fresh install, you need to supply all the fields.</aside>

<aside class="notice">* All fields are optional, but it makes no sense to call this endpoint without any fields, right? So, at least one of them are required.
</aside>

<aside class="notice">Please note the username must be unique at platform level.</aside>

## Get own profile

```http
GET /api/v1/profile HTTP/1.1
Content-Type: application/json
Authorization: Bearer _your_access_token_
Accept: application/json

200 OK HTTP/1.1
{
  "id": "_root_",
  "username": "root",
  "first_name": "Root",
  "last_name": "User",
  "email": "root@api.com",
  "role": "root"
}
```

`GET /api/v1/profile`

This endpoint allows you to read your own profile. The returned response is an [user](#user-model).

## Delete profile

```http
DELETE /api/v1/profile HTTP/1.1
Content-Type: application/json
Authorization: Bearer _your_access_token_
Accept: application/json

204 No Content HTTP/1.1
```

`DELETE /api/v1/profile`

This endpoint allows you to delete your own profile. The stored data will be anonymized and your account blocked.

<aside class="warning">You cannot delete your profile if you are the last root account.</aside>

# SSO

For server-to-server communication, it is not feasible to store an username and a password for accessing the API. By using SSO, you can obtain access tokens for roles that are not linked to a specific user.

The authentication (obtaining an access token) is accomplished using a `client_id` and a `client_secret` instead of a username and a password.

## List

```http
GET /api/v1/sso HTTP/1.1
Authorization: Bearer <access_token>
Accept: application/json

200 OK HTTP/1.1
{
  "total": 1,
  "page": 0,
  "chunk": 24,
  "navigation": {
    "prev": null,
    "next": null,
  }
  "type": "sso",
  "data": [
    {
      "client_id": "UfRn2uAe6qRCJ492ARHCfeCy5Mro1SLS",
      "client_secret": "T`;***;vB",
      "role": "admin"
    }
  ]
}
```

This endpoint gets a list with all your generated pairs of client id and client secrets, with obfuscated client secret (only first three and last three characters shown, with three `*` in the middle).

## Create

```http
POST /api/v1/sso HTTP/1.1
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
{
  "role": "root|admin|user"
}

200 OK HTTP/1.1
{
  "client_id": "UfRn2uAe6qRCJ492ARHCfeCy5Mro1SLS",
  "client_secret": "T`;$dn%JUlX#_}sd@gAc%[anZf}I6{2w;'?~*?1fW-?h(whZb9RgUWI~1/7d;vB",
  "role": "<provided_role>"
}
```

`POST /api/v1/sso`

<aside class="danger">Please note that once generated, the client secret is not accessible anymore. You must store it on your part.</aside>

## Update

```http
PATCH /api/v1/sso/UfRn2uAe6qRCJ492ARHCfeCy5Mro1SLS HTTP/1.1
Authorization: Bearer <access_token>
Accept: application/json

200 OK HTTP/1.1
{
  "client_id": "UfRn2uAe6qRCJ492ARHCfeCy5Mro1SLS",
  "client_secret": "T`;$dn%JUlX#_}sd@gAc%[anKf}I6{2w;'?~*?1fW-?h(whZb9RgUWI~1/7d;vB",
  "role": "<existent_role>"
}
```

`PATCH /api/v1/sso/<client_id>`

In case you forgot your client secret, you think it got leaked or just as a precaution, you can regenerate the client secret for a certain client id. Once re-generated, the old client secret will not work anymore.

## Delete

```http
DELETE /api/v1/sso/UfRn2uAe6qRCJ492ARHCfeCy5Mro1SLS HTTP/1.1
Authorization: Bearer <access_token>
Accept: application/json

204 No Content HTTP/1.1
```

`DELETE /api/v1/sso/<client_id>`

This will immediately delete the client id and client secret pair from the system.

## Use

# Users

There are three levels on which you can use these endpoints, depending on your obtained `access_token`'s scope:

| Scope | Access                                               |
| ----- | ---------------------------------------------------- |
| root  | Unrestricted access                                  |
| admin | You cannot access any user of type `root` or `admin` |
| user  | You have no access to any of these endpoints         |

## Get a list of users

```http
GET /api/v1/users HTTP/1.1
Authorization: Bearer _your_access_token
Content-Type: application/json
Accept: application/json

200 OK HTTP/1.1
{
  "total": 5,
  "page": 0,
  "chunk": 3,
  "navigation": {
    "prev": null,
    "next": "/api/v1/users?page=1&chunk=3"
  }
  "type": "users",
  "data": [
    {
      "id": "_root_",
      "username": "root",
      "first_name": "Root",
      "last_name": "User",
      "email": "root@api.com",
      "role": "root"
    },
    {
      "id": "b52fa022-65f7-4a85-be35-f0d70d6436cb",
      "username": "jon",
      "first_name": "Jon",
      "last_name": "Appleseed",
      "email": "jon@appleseed.com",
      "role": "admin"
    },
    {
      "id": "b52fa022-65f7-4a85-be35-f0d70d6436cc",
      "username": "james",
      "first_name": "James",
      "last_name": "May",
      "email": "james@may.com",
      "role": "user"
    }
  ]
}
```

`GET /api/v1/users?page=0&chunk=1000`

### Query parameters

| Name  | Required | Type    | Default value | Description                           |
| ----- | -------- | ------- | ------------- | ------------------------------------- |
| page  | no       | integer | 0             | The page number, starting with zero   |
| chunk | no       | integer | 24            | The chunk size between `1` and `1000` |

### Response

| Name            | Type             | Description                                                                                                                                    |
| --------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| total           | integer          | The total number of records available                                                                                                          |
| page            | integer          | The current page                                                                                                                               |
| chunk           | integer          | The chunk size used                                                                                                                            |
| type            | string           | The data type (in this case, `"users"`)                                                                                                        |
| navigation.prev | string or `null` | A link for getting the previous page, using the same chunk size as you provided (or default value), or `null` if no previous page is available |
| navigation.next | string or `null` | A link for getting the next page, using the same chunk size as you provided (or default value), or `null` if no data is available on next page |
| data            | array            | An array of [user objects](#user-model)                                                                                                        |

## Get a single user

```http
GET /api/v1/users/<identifier> HTTP/1.1
Authorization: Bearer _your_access_token
Content-Type: application/json
Accept: application/json

200 OK HTTP/1.1
{
  "id": "_root_",
  "username": "root",
  "first_name": "Root",
  "last_name": "User",
  "email": "root@api.com",
  "role": "root"
}
```

`GET /api/v1/users/<identifier>`

Gets a single user identified by `<identifier>`. This value can be either the user's `id` or the `username`.

The return value is a [user model](#user-model).

## Update an user

```http
PUT /api/v1/users/<identifier> HTTP/1.1
Authorization: Bearer _your_access_token
Content-Type: application/json
Accept: application/json
{
  "username": "admin",
  "first_name": "Admin",
  "last_name": "User",
  "email": "admin@api.com",
  "role": "admin"
}

200 OK HTTP/1.1
{
  "id": "_root_"
  "username": "admin",
  "first_name": "Admin",
  "last_name": "User",
  "email": "admin@api.com",
  "role": "admin"
}
```

`PUT /api/v1/users/<identifier>`

Updates a single user identified by `<identifier>`. This value can be either the user's `id` or the `username`.

All fields are optional.

<aside class="notice">Please note the username must be unique at platform level.</aside>

The return value is a [user model](#user-model).

## Delete an user

```http
DELETE /api/v1/users/<identifier> HTTP/1.1
Authorization: Bearer _your_access_token
Accept: application/json

204 No Content HTTP/1.1
```

`DELETE /api/v1/users/<identifier>`

Updates a single user identified by `<identifier>`. This value can be either the user's `id` or the `username`.

<aside class="notice">Please note you cannot delete your own user or the last root.</aside>
