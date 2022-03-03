# Errors

## Error: **401 Unauthorized**

```json
{
  "error": true,
  "status": 401,
  "type": "Unauthorized | TokenExpired",
  "message": "You are not authorized to make this request"
}
```

This error is usually raised when the access token is missing, invalid or expired. Also, it can be encountered, for example, if you are using an access token obtained in a certain scope to call API endpoints related to another scope.

## Error: **403 Forbidden**

```json
{
  "error": true,
  "status": 403,
  "type": "Forbidden",
  "message": "Wrong credentials"
}
```

This error is usually generated when a wrong set of username and/or password are supplied for an authentication endpoint. The HTTP Status code is `403` also.

## Error: **422 Unprocessable Entity**

```json
{
  "error": true,
  "status": 422,
  "type": "Validation",
  "code": 10422,
  "message": "password required"
}
```

This error is generated when the input passed in body or query params is not valid. For example, a field is missing or is empty and the API marked it as required, or if the format is invalid, like when the API expects a number or a boolean and you provide a string.

<aside class="notice">
Please note this error uses a fail-fast approach, meaning when the first invalid field is encountered, the validation process stops and returns the error. A subsequent call to the same endpoint with the correct field can yield another 422 error with another field. The validation order is hidden in the implementation and you cannot rely on the order the fields are supplied in the payload, query parameters or query string.
</aside>
