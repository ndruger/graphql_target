# GraphQLのテスト

## 参考

- チートシート
  - [GraphQL Schema Language Cheat Sheet – wehavefaces by hafiz ismail](https://wehavefaces.net/graphql-shorthand-notation-cheatsheet-17cd715861b6)

- 仕様
  - [GraphQL](https://facebook.github.io/graphql/)

## シンプルなクエリ

Query
```graphql
query { 
  viewer {
    name,
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
      "email": "",
      "repository": {
        "id": "MDEwOlJlcG9zaXRvcnkxMDE2Njg3MzQ=",
        "name": "graphql_target"
      }
    }
  }
}
```

- 欲しいフィールドだけ取得できる
- 自分の情報と、自分のリポジトリの情報を一度のクエリで取得できる

## viewerのフィールドやrepositoryの引数はどこからわかるのか？

[Introduction to GraphQL | GitHub Developer Guide](https://developer.github.com/v4/guides/intro-to-graphql/#discovering-the-graphql-api)でcurlで取得した[./github_graphql.json](./github_graphql.json)の下記からわかる。

ただ、[Reference | GitHub Developer Guide](https://developer.github.com/v4/reference/)みたほうが早い。

`viewer`フィールドの定義
```json
            {
              "name": "viewer",
              "description": "The currently authenticated user.",
              "args": [],
              "type": {
                "kind": "NON_NULL",
                "name": null,
                "ofType": {
                  "kind": "OBJECT",
                  "name": "User",
                  "ofType": null
                }
              },
              "isDeprecated": false,
              "deprecationReason": null
            }
```

- `viewer`フィールドは`User`型である。

`User`型の定義
```json

        {
          "kind": "OBJECT",
          "name": "User",
          "description": "A user is an individual's account on GitHub that owns repositories and can make new content.",
          "fields": [
...省略...
            {
              "name": "name",
              "description": "The user's public profile name.",
              "args": [],
              "type": {
                "kind": "SCALAR",
                "name": "String",
                "ofType": null
              },
              "isDeprecated": false,
              "deprecationReason": null
            },
...省略...
            {
              "name": "email",
              "description": "The user's publicly visible profile email.",
              "args": [],
              "type": {
                "kind": "NON_NULL",
                "name": null,
                "ofType": {
                  "kind": "SCALAR",
                  "name": "String",
                  "ofType": null
                }
              },
              "isDeprecated": false,
              "deprecationReason": null
            },
...省略...
            {
              "name": "repository",
              "description": "Find Repository.",
              "args": [
                {
                  "name": "name",
                  "description": "Name of Repository to find.",
                  "type": {
                    "kind": "NON_NULL",
                    "name": null,
                    "ofType": {
                      "kind": "SCALAR",
                      "name": "String",
                      "ofType": null
                    }
                  },
                  "defaultValue": null
                }
              ],
              "type": {
                "kind": "OBJECT",
                "name": "Repository",
                "ofType": null
              },
              "isDeprecated": false,
              "deprecationReason": null
            },
...省略...

```

- `User`型には`name`や`email`フィールドや引数付きの`repository`フィールドがある。
- 引数付きの`repository`フィールドの必須の引数は`name`である。

## データの更新

issueを10件まで取得
```graphql
query { 
  viewer { 
    repository(name: "graphql_target") {
      issues(first: 10) {
        edges {
          node {
            id
          }
        }
      },
    }
  }
}
```

issueの10件まで取得結果
```graphql
{
  "data": {
    "viewer": {
      "repository": {
        "issues": {
          "edges": [
            {
              "node": {
                "id": "MDU6SXNzdWUyNTM0MDE1NzQ="
              }
            }
          ]
        }
      }
    }
  }
}
```

LAUGHマークとコメントを追加
```graphql
mutation {
  addReaction(input:{subjectId:"MDU6SXNzdWUyNTM0MDE1NzQ=",content:LAUGH}) {
    reaction {
      content
    }
    subject {
      id
    }
  }
  addComment(input:{subjectId:"MDU6SXNzdWUyNTM0MDE1NzQ=",body:"テストコメント"}) {
    subject {
      id
    }
  }
}
```

- 複数の更新操作を一度のリクエストで行える。

## Variables

Query
```graphql
query ($limit: Int) { 
  viewer { 
    repository(name: "graphql_target") {
      issues(first: $limit) {
        edges {
          node {
            id
          }
        }
      },
    }
  }
```

Query Variables
```graphql
{
  "limit": 10
}
```

Result
```graphql
{
  "data": {
    "viewer": {
      "repository": {
        "issues": {
          "edges": [
            {
              "node": {
                "id": "MDU6SXNzdWUyNTM0MDE1NzQ="
              }
            }
          ]
        }
      }
    }
  }
}
```

- Chrome Developer Toolsを見るとわかるが、送信前に$limitを解決しているのではなく、queryとvariablesを両方サーバに送ってサーバで解決をしている。

## Fragments

Query
```graphql
query { 
  viewer {
    ...viewerFields
    repository(name: "graphql_target") {
      id,
      name
    }
  }
}

fragment viewerFields on User {
  name,
  email
}
```

Result
```graphql
{
  "data": {
    "viewer": {
      "repository": {
        "id": "MDEwOlJlcG9zaXRvcnkxMDE2Njg3MzQ=",
        "name": "graphql_target"
      },
      "name": "ndruger",
      "email": ""
    }
  }
}
```

## Introspection

Query
```graphql
query {
  __type(name: "User") {
    name
    fields {
      name
      type {
        name
      }
    }
  }
}
```

Result
```graphql
{
  "data": {
    "__type": {
      "name": "User",
      "fields": [
        {
          "name": "avatarUrl",
          "type": {
            "name": null
          }
        },
        {
          "name": "bio",
          "type": {
            "name": "String"
          }
        },
        {
          "name": "bioHTML",
          "type": {
            "name": null
          }
        },
...省略...
      ]
    }
  }
}
```
