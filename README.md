
# Simple Query

Query
```graphql
query { 
  viewer {
    name,
    isDeveloperProgramMember,
    email,
    repository(name: "graphql_target") {
      id,
      name
    }
  }
}
```

Result
```graphql
{
  "data": {
    "viewer": {
      "name": "ndruger",
      "isDeveloperProgramMember": false,
      "email": "",
      "repository": {
        "id": "MDEwOlJlcG9zaXRvcnkxMDE2Njg3MzQ=",
        "name": "graphql_target"
      }
    }
  }
}
```
