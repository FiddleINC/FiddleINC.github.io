---
layout: blog_post
title: Developing a Simple Server in Go
category: blog
---

**Go**, devloped by the big boy, **Google** soon rose to prominence in the Backend world thanks to, of course, big boy *Google*. For the last couple of years, since its inception, major IT companies are moving towards this newbie in their development and why not.

In a year of me using the language, I really feel the Old and New coming together in one entity. I started out with C++ and moved onto scripting using Javascript, and I really dig the easiness of the latter and the detailing of the former. 

Now let me tell you that I have developed servers in Node, Python and even C++. Other than the JSON Handling capabilities of Node, Go really gives it a tough fight. If you are a Data Science Guy, I don't envy you. You are gonna most probably fiddle your whole life with Django and Flask.

So in this article, I just want to show you how to develop a simple Form Based Front-End and a GoLang Based Back-End

## Client Side

For the Front-End I have a basic form with a Name Label and an Address Label

```html
<form action="/data" method="post">
        <input type="text" id="name" name="name" placeholder="Enter Your Name">
        <input type="text" id="address" name="address" placeholder="Enter Your Address"> 
        <input type="submit" value="submit">
</form>
```
When we enter the form and press enter, it redirects to ```/data```.

## Server Side 

For working with HTTP Services, we will use ```net/http``` package for Go. You can also use the ```gorilla mux``` package provided by Firefox which is really helpful Full Scale Server Development. In this case, I have used the former just to give a brief idea. The package is already included Go, so we don't need to get any external package.

```go
import (
	"net/http"
	"log"
)
```

I would be using ```log``` in the server stdout as it also shows the timestamp for each log which is really convienient ( also it reminds me of my dear ```console.log``` ). 

To start a basic server listening on, say ```PORT = 8080```

```go
func main() {
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This will start the server at ```http://localhost:8080``` 

Now just like middleware in Express in NodeJS, Go used ```http.Handle``` & ```http.HandleFunc```. The difference being that ```http.Handle``` takes in a interface as an argument along with the url whereas ```http.HandleFunc``` takes a function as an argument.

```go
func Handle(pattern string, handler Handler) {
    DefaultServeMux.Handle(pattern, handler) 
}

//Defination for Handler as given in the Documentation
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

```go
HandleFunc(pattern string, func(w ResponseWriter, r *Request) {
    // function logic
}
```

If you are not used to interfaces, don't worry I will be using ```http.HandleFunc``` in this case.

For the Home URL, we can serve our Client Side using the ```http.ServeFile``` function

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		http.ServeFile(w, r, "client/index.html")
    })
```

So, the whole code becomes,

```go
package main

import (
	"net/http"
	"strconv"
	"log"
)

func main() {
	port := 8080

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		http.ServeFile(w, r, "client/index.html")
    })
    
    //I have intialized the port before
    log.Fatal(http.ListenAndServe(":" + strconv.Itoa(port) , nil))
    log.Printf("The Server is running on Port ", + strconv.Itoa(port))
}
```

Now, if you run the program using,

```bash
go run main.go
```
and open ```http://localhost:8080```,
we will be able to see the form.

Now let's handle the form.

---

As you remember we provided the redirect url for the form at ```/data```, therefore when the form is submitted, the request body now contains the form data.

A major point to be noted here is that to parse the data recieved from the client to the Go Server, Normally a client sends a data as an XML Query, JSON Object or a URL Query. In the case of form data, set the set Content Type to application/x-www-form-urlencoded to process the request data as Form Data. If you set your method to "POST" , then it will automatically set the above. Thanks to Edward Pie for the trick

Now we have the request body. Now to access the form data, we will use ```ParseForm``` function which will automatically parse the form data into key-value pairs for me.

```go
http.HandleFunc("/data", func(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()
	})
```
Here w is basically the response to be sent and r is the request from the server

Now you can access the value of the data using the key using the ```FormValue``` function

```go
value := r.FormValue("key")
```
In our case we will save the data in to their respective variables

```go
name := r.FormValue("name")
address := r.FormValue("address")
```
You can also create a struct and store the data,

```go
type Data struct {
    name string
    address string
}
```
Now you can use the data to write to the client or log in the server

```go
//Send to Client
w.Write([]byte(name))

//Log in Server
log.Printf("Name : %s, Address : %s", name, address)
```

Now the server code looks like

```go
package main

import (
	"net/http"
	"strconv"
	"log"
)

func main() {
	port := 8080

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		http.ServeFile(w, r, "client/index.html")
	})
	
	http.HandleFunc("/data", func(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()

		name := r.FormValue("name")
		address := r.FormValue("address")
		
		log.Printf("Name : %s  Address : %s", name, address)
	})

	log.Fatal(http.ListenAndServe(":" + strconv.Itoa(port) , nil))
}
```
Your client and server is ready.