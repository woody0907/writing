> in go, you simply make ever file in the same directory of main package,and they'll be included
> 
> template file
    {{define "layout"}}
    {{template "navebar" .}}
    {{end}}

### Handling requests
#### The Go net/http library
>cargo cult programming

the net/http library id divided into two parts:
Client: Client, Response, Header, Request, Cookie
Server: Server, ServerMux, Handler/HandleFun, ResponseWriter, Header,Request,Cookie

#### The Go web Server
Creating a server is trivial and can be done with a call to ListenAndServe, with the network address as the first parameter and the handler that take care of the request the second parameter.
If the first parameter is an empty string, the defautl is all network interface at port 80. If the handler parameter is nil, the default multiplexer, DefualtServeMux, is used.

Go also provide a Server Struct that's essentially a server configuration.
```go
type Server struct {
Addr string
Handler Handler
ReadTimeout time.Duration
WriteTimeout time.Duration
MaxHeaderBytes int
TLSConfig *tls.Config
TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
ConnState func(net.Conn, ConnState)
ErrorLog *log.Logger
}
```

#### Serving through HTTPs
HTTPS is nothing more than layering HTTP on top of SSL[security socket layer](actually, Transport Security Layer[TLS]). To server our web application through HTTPs,we'll use ListenAndServeTLS funtion.

```go
func main() {
server := http.Server{
Addr: "127.0.0.1:8080",
Handler: nil,
}
server.ListenAndServeTLS("cert.pem", "key.pem")
}
```

the cert.pem is the SSL certificate whereas key.pem is the privat key for the server.

#### Generate your own SSL certificate and private key


#### Handling requests
In Go, a handler is an interface that has a method named ServeHTTP with two parameters: an HTTPResponseWriter interface and a pointer to a Request struct.

```go
ackage main
import (
"fmt"
"net/http"
)
type HelloHandler struct{}
func (h *HelloHandler) ServeHTTP (w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "Hello!")
}
type WorldHandler struct{}
func (h *WorldHandler) ServeHTTP (w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "World!")
}
func main() {
hello := HelloHandler{}
world := WorldHandler{}
server := http.Server{
Addr: "127.0.0.1:8080",
}
http.Handle("/hello", &hello)
http.Handle("/world", &world)
server.ListenAndServe()
}
```

#### Handler functions

```go
func hello(w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "Hello!")
}
func world(w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "World!")
}
func main() {
server := http.Server{
Addr: "127.0.0.1:8080",
}
http.HandleFunc("/hello", hello)
http.HandleFunc("/world", world)
server.ListenAndServe()
}
```
the HandleFunc function converts the hello function into a Handler
and registers it to DefaultServeMux. In other words, handler functions are merely convenient ways of creating handlers.

#### Chaining handlers and handler functions

A common way of cleanly separating cross-cutting conserns away from your other logic is chaining.

    
For any registered URLs that don’t end with a slash (/), ServeMux will try to
match the exact URL pattern. If the URL ends with a slash (/), ServeMux will see if the requested URL starts with any registered URL

#### using http2

```go
package main
import (
"fmt"
"golang.org/x/net/http2"
"net/http"
)
type MyHandler struct{}
func (h *MyHandler) ServeHTTP (w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "Hello World!")
}
func main() {
handler := MyHandler{}
server := http.Server{
Addr: "127.0.0.1:8080",
Handler: &handler,
}
http2.ConfigureServer(&server, &http2.Server{})
server.ListenAndServeTLS("cert.pem", "key.pem")
}
```

### Processing requests

#### Request
Some important parts of Resqeust are:
* URL
* Header
* Body
* Form,PostForm,and MultipartForm

#### Request Url

> scheme://[userinfo@]host/path[?query][#fragment]
> URLs that don’t start with a slash after the scheme are interpreted as
scheme:opaque[?query][#fragment]

#### Request header
Request and response headers are described in the Header type, which is a map representing the key-value pairs in an HTTP header. There are four basic methods on Header, which allow you to add, delete, get, and set values, given a key. Both the key and the value are strings.

> a header is a map with keys that are strings and values are slices of strings


#### Reqeust body

Both request and response bodies are represented by the Body field, which is an
io.ReadCloser interface. In turn the Body field consists of a Reader interface and a Closer interface

#### Reqeust form
```go
r.ParseForm()
fmt.Fprintln(w, r.Form)
```
As mentioned earlier, you need to first parse the request using ParseForm, and then access the Form field

```go
r.ParseMultipartForm(1024)
fmt.Fprintln(w, r.MultipartForm)
```
You need to tell the ParseMultipartForm method how much data you want to extract from the multipart form, in bytes

#### files

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Go Web Programming</title>
</head>
<body>
<form action="http://localhost:8080/process?hello=world&thread=123"
method="post" enctype="multipart/form-data">
<input type="text" name="hello" value="sau sheong"/>
<input type="text" name="post" value="456"/>
<input type="file" name="uploaded">
<input type="submit">
</form>
</body>
</html>
```
```go
package main
import (
"fmt"
"io/ioutil"
"net/http"
)
func process(w http.ResponseWriter, r *http.Request) {
r.ParseMultipartForm(1024)
fileHeader := r.MultipartForm.File["uploaded"][0]
file, err := fileHeader.Open()
if err == nil {
data, err := ioutil.ReadAll(file)
if err == nil {
fmt.Fprintln(w, string(data))
}
}
}
func main() {
server := http.Server{
Addr: "127.0.0.1:8080",
}
http.HandleFunc("/process", process)
server.ListenAndServe()
}
```

As with the FormValue and PostFormValue methods, there’s a shorter way of getting an uploaded file using the FormFile method

#### Processing POST requests with JSON body

#### ResponseWriter

sending a response from the server to client

the responsewriter interface has three methond:
* Write
* WriteHeader
* Header

the writeheader methos is pretty usefule if you want return error code

header method returns a map of headers that you can modify.

```go
func headerExample(w http.ResponseWriter, r *http.Request) {
w.Header().Set("Location", "http://google.com")
w.WriteHeader(302)
}
```

### Cookies

there are two classes of cookies: session cookies and persistent cookies

#### sending cookies to the browser
You use the Set method to add the first cookie and then the Add method to add a second cookie
```go
func setCookie(w http.ResponseWriter, r *http.Request) {
c1 := http.Cookie{
Name: "first_cookie",
Value: "Go Web Programming",
HttpOnly: true,
}
c2 := http.Cookie{
Name: "second_cookie",
Value: "Manning Publications Co",
HttpOnly: true,
}
w.Header().Set("Set-Cookie", c1.String())
w.Header().Add("Set-Cookie", c2.String())
}
```

#### get cookies from the header
```go
h := r.Header["Cookie"]
```

Go provides a couple of easy ways to get cookies
```go
func getCookie(w http.ResponseWriter, r *http.Request) {
c1, err := r.Cookie("first_cookie")
if err != nil {
fmt.Fprintln(w, "Cannot get the first cookie")
}
cs := r.Cookies()
fmt.Fprintln(w, c1)
fmt.Fprintln(w, cs)
}
```

#### using cookies for flash messages

```go
func setMessage(w http.ResponseWriter, r *http.Request) {
msg := []byte("Hello World!")
c := http.Cookie{
Name: "flash",
Value: base64.URLEncoding.EncodeToString(msg),
}
http.SetCookie(w, &c)
}
```

```go
rc := http.Cookie{
Name: "flash",
MaxAge: -1,
Expires: time.Unix(1, 0),
}
http.SetCookie(w, &rc)
val, _ := base64.URLEncoding.DecodeString(c.Value)
fmt.Fprintln(w, string)
```

### Display content

there are roughly two ideal type of template engines,
* Logic-less templage
* Embedded logic template engine
Go's template engin is a hybrid

Using the go template engine requires two steps:
* parse the text-formatted template source.
* execute the parsed template

```go
func process(w http.ResponseWriter, r *http.Request) {
t, _ := template.ParseFiles("tmpl.html")
t.Execute(w, "Hello World!")
}
```
    
another way to parse file is to use the ParseGlob function, which use pattern matching instead of specific files.

```go
t, _ := template.ParseGlob("*.html")
```

The usual Go practice is to handle the error, but Go provides another mechanism to handle errors returned by parsing templates:

```go
t := template.Must(template.ParseFiles("tmpl.html"))
```

The Must function wraps around a function that returns a pointer to a template and an error, and panics if the error is not a nil. (In Go, panicking refers to a situation where the normal flow of execution is stopped, and if it’s within a function, the function returns to its caller.

If you call the Execute method on a template set, it’ll take the first template in the set. But if you want to execute a different template in the template set and not the first one, you need to use the ExecuteTemplate method
```go
t, _ := template.ParseFiles("t1.html", "t2.html")
t.Execute(w, "Hello World!")
t.ExecuteTemplate(w, "t2.html", "Hello World!")
```
#### Actions

* Conditional actions
* Iterator actions
* Include actions
* Set actions

It might come as a surprise, but the dot (.) is an action, and it’s the most important one. The dot is the evaluation of the data that’s passed to the template

#### Conditional actions

```go
{{ if arg }}
some content
{{ end }}
```

```go
{{ if arg }}
some content
{{ else }}
other content
{{ end }}
```

#### Iterator actions
```go
{{ range array }}
Dot is set to the element {{ . }}
{{ end }}
```

#### Set actions

```go
{{ with arg }}
Dot is set to arg
{{ end }}
```

```go
{{ with arg }}
Dot is set to arg
{{ else }}
Fallback if arg is empty
{{ end }}
```

#### Include actions
```go
{{ template "name" }}
```

```go
{{template "name" arg }}
```
where arg is the argument you want to pass on to the included template

#### arguments, variables, and pipelines

An argument is a value that’s used in a template. It can be a Boolean, integer, string, and so on. It can also be a struct, or a field of a struct, or the key of an array. Arguments can be a variable, a method (which must return either one value, or a value and an error) or a function. An argument can also be a dot (.), which is the value passed from the template engine

```go
{{ range $key, $value := . }}
The key is {{ $key }} and the value is {{ $value }}
{{ end }}
```

Pipelines are arguments, functions, and methods chained together in a sequence.
This works much like the Unix pipeline. In fact, the syntax looks very similar

```go
{{ p1 | p2 | p3 }}
```

#### Functions

* Creating a FuncMap map, which has the name of the function as the key and the actual function as the value
* Attach FuncMap to the template

```go
func process(w http.ResponseWriter, r *http.Request) {
funcMap := template.FuncMap { "fdate": formatDate }
t := template.New("tmpl.html").Funcs(funcMap)
t, _ = t.ParseFiles("tmpl.html")
t.Execute(w, time.Now())
}
```
#### Context awareness

```
Original text I asked: <i>"What's up?"</i>
{{ . }} I asked: &lt;i&gt;&#34;What&#39;s
up?&#34;&lt;/i&gt;
<a href="/{{ . }}"> I%20asked:%20%3ci%3e%22What%27s%20up?%22%
3c/i%3e
<a href="/?q={{ . }}"> I%20asked%3a%20%3ci%3e%22What%27s%20up%3f
%22%3c%2fi%3e
<a onclick="{{ . }}"> I asked: \x3ci\x3e\x22What\x27s
up?\x22\x3c\/i\x3e
```

Context-awareness isn’t just for HTML, it also works on XSS attacks on JavaScript, CSS, and even URLs.

#### Unescaping  HTML
Just cast your input string to template.HTML and use that instead, and our
code is happily unescaped.
```go
func process(w http.ResponseWriter, r *http.Request) {
t, _ := template.ParseFiles("tmpl.html")
t.Execute(w, template.HTML(r.FormValue("comment")))
}
```

#### Nesting templates

We can explicitly define a template in a template file using the define action

```go
{{ define "layout" }}
{{end}}
```

The following listing shows how we use these templates in our handler.

```go
func process(w http.ResponseWriter, r *http.Request) {
t, _ := template.ParseFiles("layout.html")
t.ExecuteTemplate(w, "layout", "")
}
```

#### Using block action to define default template
```html
{{ define "layout" }}
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Go Web Programming</title>
</head>
<body>
{{ block "content" . }}
<h1 style="color: blue;">Hello World!</h1>
{{ end }}
</body>
</html>
{{ end }}
```
```go
func process(w http.ResponseWriter, r *http.Request) {
rand.Seed(time.Now().Unix())
var t *template.Template
if rand.Intn(10) > 5 {
t, _ = template.ParseFiles("layout.html", "red_hello.html")
} else {
t, _ = template.ParseFiles("layout.html")
}
t.ExecuteTemplate(w, "layout", "")
}
```

#### In-memeory storage
In-memeory storage usually stored in data structures, and for go, this primarily means with arrays, slices,maps,and most importantly, structs.

#### File storage

```go
import (
"fmt"
"io/ioutil"
"os"
)
func main() {
data := []byte("Hello World!\n")
err := ioutil.WriteFile("data1", data, 0644)
if err != nil {
panic(err)
}
read1, _ := ioutil.ReadFile("data1")
fmt.Print(string(read1))
file1, _ := os.Create("data2")
defer file1.Close()
bytes, _ := file1.Write(data)
fmt.Printf("Wrote %d bytes to file\n", bytes)
file2, _ := os.Open("data2")
defer file2.Close()
read2 := make([]byte, len(data))
bytes, _ = file2.Read(read2)
fmt.Printf("Read %d bytes from file\n", bytes)
fmt.Println(string(read2))
}
```

##### Reading and writing CSV files
    
##### The gob package

The encoding/gob package manages streams of gobs, which are binary data, exchange between an encoder and a decoder. It's designed for serialization and transporting data but it can also be userd for persisting data. 
```go
func store(data interface{}, filename string) {
buffer := new(bytes.Buffer)
encoder := gob.NewEncoder(buffer)
err := encoder.Encode(data)
if err != nil {
panic(err)
}
err = ioutil.WriteFile(filename, buffer.Bytes(), 0600)
if err != nil {
panic(err)
}
}
func load(data interface{}, filename string) {
raw, err := ioutil.ReadFile(filename)
if err != nil {
panic(err)
}
buffer := bytes.NewBuffer(raw)
dec := gob.NewDecoder(buffer)
err = dec.Decode(data)
if err != nil {
panic(err)
}
}
```
### Go and SQL

* connecting  to the database
```go
var Db *sql.DB
func init() {
var err error
Db, err = sql.Open("postgres", "user=gwp dbname=gwp password=gwp
sslmode=disable")
if err != nil {
panic(err)
}
}
```

* insert
```go
func (post *Post) Create() (err error) {
statement := "insert into posts (content, author) values ($1, $2)
➥ returning id "
stmt, err := db.Prepare(statement)
if err != nil {
return
}
defer stmt.Close()
err = stmt.QueryRow(post.Content, post.Author).Scan(&post.Id)
if err != nil {
return
}
return
}
```
* query
```go
func GetPost(id int) (post Post, err error) {
post = Post{}
err = Db.QueryRow("select id, content, author from posts where id =
➥ $1", id).Scan(&post.Id, &post.Content, &post.Author)
return
}
```

* update
```go
func (post *Post) Update() (err error) {
_, err = Db.Exec("update posts set content = $2, author = $3 where id =
➥ $1", post.Id, post.Content, post.Author)
return
}
```
* delete
```go
func (post *Post) Delete() (err error) {
_, err = Db.Exec("delete from posts where id = $1", post.Id)
return
}
```

* query all
```go
func Posts(limit int) (posts []Post, err error) {
rows, err := Db.Query("select id, content, author from posts limit $1",
➥ limit)
if err != nil {
return
}
for rows.Next() {
post := Post{}
err = rows.Scan(&post.Id, &post.Content, &post.Author)
if err != nil {
return
}
posts = append(posts, post)
}
rows.Close()
return
}
```

#### Go and SQL relationships
Go relational mapper

* Sqlx

```shell
go get "github.com/jmoiron/sqlx"
```
* Gorm

the fantastic ORM for Go(lang)
```shell
go get "github.com/jinzhu/gorm"
```




























