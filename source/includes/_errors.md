# Errors

Classcharts doesn't return a different status code if you do something wrong. Instead, the JSON object it returns won't contain any data, and the `error` field will be populated with a description of the error.

## Example
```json
{
  "success": 0,
  "expired": 0,
  "error": "You are not logged in. [1]",
  "meta": []
}
```
