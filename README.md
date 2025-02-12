# go-cerberus
Cerberus go client library, meant to be used on a11n.io Cerberus enabled application backends.

## Contents

- [Go Cerberus](#go-cerberus)
  - [Contents](#contents)
  - [Installation](#installation)
  - [Quick start](#quick-start)

### Installation

To install go-cerberus, you need to install Go and have a workspace.

1. Install [Go](http://go.dev), then install go-cerberus:
2. 
```sh
go get -u github.com/a11n-io/go-cerberus
```

2. Import it in your code:

```go
import cerberus "github.com/github.com/a11n-io/go-cerberus"
```

### Quick start

go-cerberus requires an Api key and secret from an app that you've configured on [Cerberus](cerberus.a11n.io).
NB!! Treat the API secret securely, never use in frontend code, and do not check into source repositories.

Once you've configured your project with the API key and secret (e.g. in configuration, parameters or environment variables), you can instantiate the client like this:

```go
cerberusClient := cerberus.NewClient("https://cerberus-api.a11n.io", CERBERUS_API_KEY, CERBERUS_API_SECRET)
```

Once the client has been instantiated, you can pass it down to your service layer and use it everywhere you need to interact with Cerberus,
e.g. getting a JWT token, creating resources, roles, users, assigning roles, etc.

Getting a token is the first thing you'll have to do, as all other operations require this token.
E.g. when accepting new account / user registrations, you'd want to create those locally, as well as on Cerberus, then immediately assign permissions (and optionally roles) to the user:

```go
ctx := context.Background()
cerberusToken, err := cerberusClient.GetUserToken(ctx, accountId, userId)
if err != nil {
// handle error
}

cerberusContext := context.WithValue(ctx, "cerberusToken", cerberusToken)
_, err := cerberusClient.CreateResource(cerberusContext, accountId, "", "Account")
if err != nil {
// handle error
}

_, err = cerberusClient.CreateUser(cerberusContext, userId, userEmail, userName)
if err != nil {
// handle error
}

roleId := uuid.New().String()
_, err = cerberusClient.CreateRole(cerberusContext, roleId, "AccountAdministrator")
if err != nil {
// handle error
}

err = cerberusClient.AssignRole(cerberusContext, roleId, userId)
if err != nil {
// handle error
}

err = cerberusClient.CreatePermission(cerberusContext, roleId, accountId, []string{"CanManageAccount"})
if err != nil {
// handle error
}
```

When executing multiple cerberus commands, it's better to use the 'Execute' method, which runs them all in sequence atomically.
The code block above would then become:

```go
ctx := context.Background()
cerberusToken, err := cerberusClient.GetUserToken(ctx, accountId, userId)
if err != nil {
// handle error
}

cerberusContext := context.WithValue(ctx, "cerberusToken", cerberusToken)

err := cerberusClient.Execute(cerberusContext,
    CreateResourceCmd(accountId, "", "Account"),
    CreateUserCmd(userId, userEmail, userName),
    CreateRoleCmd(roleId, "AccountAdministrator"),
    AssignRoleCmd(roleId, userId),
    CreatePermissionCmd(roleId, accountId, []string{"CanManageAccount"}))
if err != nil {
// handle error
}
```

