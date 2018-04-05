# Responder

Responder provides a means to respond differently according to a request's accepted mime type.

[![GoDoc](https://godoc.org/github.com/paurudev/responder?status.svg)](https://godoc.org/github.com/paurudev/responder)

## Usage

### Register mime type

```go
import "github.com/paurudev/responder"

responder.Register("text/html", "html")
responder.Register("application/json", "json")
responder.Register("application/xml", "xml")
```

[Responder](https://github.com/paurudev/responder) has the above 3 mime types registered by default. You can register more types with the `Register` function, which accepts 2 parameters:

1. The mime type, like `text/html`
2. The format of the mime type, like `html`

### Respond to registered mime types

```go
func handler(writer http.ResponseWriter, request *http.Request) {
  responder.With("html", func() {
    writer.Write([]byte("this is a html request"))
  }).With([]string{"json", "xml"}, func() {
    writer.Write([]byte("this is a json or xml request"))
  }).Respond(request)
})
```

The first `html` in the example will be the default response type if [Responder](https://github.com/paurudev/responder) cannot find a corresponding mime type.
