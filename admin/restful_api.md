# RESTFul API

QOR Admin generates RESTFul API for registered resources

## Basic Usage

```go
func main() {
  API := admin.New(&qor.Config{DB: db.DB})
  user := API.AddResource(&User{})

  mux := http.NewServeMux()
  API.MountTo("/api", mux)
  http.ListenAndServe(":3000", mux); err != nil {
}
```

Once you have your `user` resource defined, QOR Admin will generate RESTFul API for it:

* GET /api/users - Retrieves a list of users
* GET /api/users/12 - Retrieves a specific user
* POST /api/users - Creates a new user
* PUT /api/users/12 - Updates user #12
* DELETE /api/users/12 - Deletes user #12

Request API with a `.json` extension, like `/api/users.json`, `QOR Admin` will return JSON formated data, request with extension `.xml` (`/api/users.xml`) will return data in XML strucutre.

You could also tell `QOR Admin` the data format that you want with `Accept` header when requesting API [[1]](#transformer).

## Customize Fields

Customize Fields in the API is same as a normal resource, you could use `IndexAttrs`, `ShowAttrs` to configure visible fields to API users.

```go
user := API.AddResource(&User{})

user.IndexAttrs("MemberNumber", "Name", "Age", "Birthday")
user.ShowAttrs("MemberNumber", "Name", "Age", "Birthday", "Points", "ShippingAddress")
```

## Actions

Sometimes, it is required to expose an operation in the API that inherently is non-RESTful.

One example of such an operation is where you want to introduce a state change for a resource, you could design it with QOR Admin like:

```go
user.Action(&admin.Action{
  Name: "enable",
  Handle: func(actionArgument *admin.ActionArgument) error {
    // `FindSelectedRecords` => in bulk action mode, will return all checked records, in other mode, will return current record
    for _, record := range actionArgument.FindSelectedRecords() {
      actionArgument.Context.DB.Model(record.(*models.User)).Update("Active", true)
    }
    return nil
  },
})

user.Action(&admin.Action{
  Name: "disable",
  Handle: func(actionArgument *admin.ActionArgument) error {
    // `FindSelectedRecords` => in bulk action mode, will return all checked records, in other mode, will return current record
    for _, record := range actionArgument.FindSelectedRecords() {
      actionArgument.Context.DB.Model(record.(*models.User)).Update("Active", false)
    }
    return nil
  },
})
```

It will generate API like:

* PUT /api/users/12/enable  - enable user #12
* PUT /api/users/12/disable - disable user #12

## Nested API

You have to have valid [GORM](http://github.com/jinzhu/gorm) relationship to register nested resource, for example:

```go
type User struct {
    gorm.Model
    Name                   string `form:"name"`
    Orders                 []Order
}

type Order struct {
    gorm.Model
    UserID     uint
    User       User
    Amount     float32
    OrderItems []OrderItem
}

user := API.AddResource(&User{})
// Register nested API with User's field name
userOrders, _ := user.AddSubResource("Orders")
```

Which will generate API:

* GET /api/users/12/orders       - Retrieves a list of orders from user #12
* GET /api/users/12/orders/22    - Retrieves order #22 from user #12
* POST /api/users/12/orders      - Creates a new order for user #12
* PUT /api/users/12/orders/22    - Updates user #12's orders #22
* DELETE /api/users/12/orders/22 - Deletes users #12's orders #22

If orders #22 doesn't belong to user #12, API will return a `404 Not Found` error

## Authentication & Authorization

same as normal Admin site, you could use [Authentication & Authorization](/admin/authentication) to guard your API.

and use `Permission` to have [resource level](/admin/resources.md#resource-configuration), [field level](/admin/fields.md#customize-meta) permission control

<a id="transformer"></a>
[1] note: QOR Admin is using [Transformer](https://github.com/pauradev/admin/blob/master/transformer.go) to convert data from resource to your requested data format as API, it only support `JSON`, `XML` right now, and `XML` only has limitted support yet, you can `GET` data with `XML` format, but can't create/update data with `XML`.
