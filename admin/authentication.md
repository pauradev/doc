# Authentication & Authorization

## Authentication

[QOR Admin](/admin/README.md) allows you to integrate your current authentication methods by providing an interface for common Authentication related tasks.

What you need to do is implement an `Auth` interface like below, and set it in the [QOR Admin](/admin/README.md) value.

```go
type Auth interface {
  GetCurrentUser(*Context) qor.CurrentUser // get current user, if don't have permission, then return nil
  LoginURL(*Context) string // get login url, if don't have permission, will redirect to this url
  LogoutURL(*Context) string // get logout url, if click logout link from admin interface, will visit this page
}
```

When setting it when initializing QOR Admin, like:

```go
func main() {
  // Set Auth interface when initialize QOR Admin
  Admin := admin.New(&admin.AdminConfig{
    Auth: yourAuthInterface,
  })
}
```

Here is an example integrated with [QOR Auth](https://github.com/pauradev/auth) & [QOR Auth Themes](https://github.com/pauradev/auth_themes):

```go
import "github.com/pauradev/auth_themes/clean"

var Auth = clean.New(&auth.Config{
  DB:         DB,
  // User model needs to implement qor.CurrentUser interface (https://godoc.org/github.com/pauradev/qor#CurrentUser) to use it in QOR Admin
  UserModel:  models.User{},
})

type AdminAuth struct {}

func (AdminAuth) LoginURL(c *admin.Context) string {
    return "/auth/login"
}

func (AdminAuth) LogoutURL(c *admin.Context) string {
    return "/auth/logout"
}

func (AdminAuth) GetCurrentUser(c *admin.Context) qor.CurrentUser {
    currentUser := Auth.GetCurrentUser(c.Request)
    if currentUser != nil {
      qorCurrentUser, ok := currentUser.(qor.CurrentUser)
      if !ok {
        fmt.Printf("User %#v haven't implement qor.CurrentUser interface\n", currentUser)
      }
      return qorCurrentUser
    }
    return nil
}

func main() {
  // Set Auth interface when initialize QOR Admin
  Admin := admin.New(&admin.AdminConfig{
    Auth: &AdminAuth{},
  })
}
```

## Authorization

QOR Admin rely on [QOR Roles](https://github.com/pauradev/roles) for [Authorization](https://en.wikipedia.org/wiki/Authorization), Check it out for details.

### Authorization For Resource

```go
Admin.AddResource(&Product{}, &admin.Config{
  Permission: roles.Deny(roles.Delete, roles.Anyone).Allow(roles.Delete, "admin")
})
```

### Authorization For Fields

```go
product := Admin.AddResource(&Product{})

product.Meta(&admin.Meta{Name: "Price", Permission: roles.Allow(roles.Update, "admin")})
```

### Authorization For Actions

QOR Admin will check permission mode `roles.Update` when checking if the current user has the ability to call an action, other modes will ignore.

```go
user.Action(&admin.Action{
  Name: "enable",
  Permission: roles.Allow(roles.Update, "admin"),
  Handle: func(actionArgument *admin.ActionArgument) error {
    // `FindSelectedRecords` => in bulk action mode, will return all checked records, in other mode, will return current record
    for _, record := range actionArgument.FindSelectedRecords() {
      actionArgument.Context.DB.Model(record.(*models.User)).Update("Active", true)
    }
    return nil
  },
})
```

### Authorization For Menus

QOR Admin will check permission mode `roles.Read` when checking if the current user has the ability to view the menu, other modes will ignore.

```go
Admin.AddMenu(&admin.Menu{Name: "Report", Link: "/admin", Permission: roles.Allow(roles.Read, "admin")})
```
