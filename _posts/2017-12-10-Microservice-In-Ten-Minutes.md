---
layout: post
title: A Micro Service In 10 Minutes
---

You can create a simple micro service in Go in just a few lines of code.

First lets create a server that will listen on port ```8080```.
We do this by importing the ```net/http``` library, then creating a new Server struct and calling the ```ListenAndServe``` function.
```go
package main

import (
	"log"
	"net/http"
)

func ListenAndServe() {
	server := &http.Server{Addr: ":8080"}
	err := server.ListenAndServe()
	if nil != err {
		log.Println(err.Error())
	}
}
```
To start the server when our application is run we add a ```main``` function.
```go
func main() {
	ListenAndServe()
}
```
## Router
Our micro service doesn't do much at the moment so lets add some routes.
We will use the [Gorrilla Mux](http://www.gorillatoolkit.org/pkg/mux) library to create routes for our micro service.
```bash
go get github.com/gorilla/mux
```
You can then import the library and create a new function that creates four simple routes.
```go
import "github.com/gorilla/mux"

func router() *mux.Router {
	router := mux.NewRouter()
	router.HandleFunc("/", List).Methods("GET")
	router.HandleFunc("/", Post).Methods("POST")
	router.HandleFunc("/{id}", Get).Methods("GET")
	router.HandleFunc("/{id}", Update).Methods("PUT")
	return router
}
```
The first parameter of ```HandleFunc``` is the path and the second is the function to be called.
The called function just needs two paramters:
- ```ResponseWriter``` we can write the header and body of our response
- ```Request``` we can read the header and body of the request
Lets add some code to be implemented later.
```go
func List(w http.ResponseWriter, req *http.Request) {

}

func Post(w http.ResponseWriter, req *http.Request) {

}

func Get(w http.ResponseWriter, req *http.Request) {

}

func Update(w http.ResponseWriter, req *http.Request) {

}
```
Now we just need to update the server inside the ```ListenAndServe``` function to use our router.
```go
server := &http.Server{Addr: ":8080", Handler: router()}
```
## Responding To Requests
If we want our micro service to output a json object we can create a struct like this one below.
```go
type Item struct {
	Name string `json:"name"`
}
```
Then using the json library we can convert it into a json string and use the ```ResponseWriter``` to write to the body.
```go
import (
	"encoding/json"
	"fmt"
)

func Get(w http.ResponseWriter, req *http.Request) {
	i := Item{"widget"}
	json, err := json.Marshal(i)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		fmt.Fprint(w, err.Error())
		return
	}
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	w.WriteHeader(http.StatusOK)
	w.Write(json)
}
```
We can test this end point by calling our end point with curl.
```bash
curl localhost:8080/1
# {"name":"widget"}
```
We can also output a json list in a similar way:
```go
func List(w http.ResponseWriter, req *http.Request) {
	items := []Item{{"widget one"}, {"widget two"}}
	json, err := json.Marshal(items)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		fmt.Fprint(w, err.Error())
		return
	}
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	w.WriteHeader(http.StatusOK)
	w.Write(json)
}
```
We get a list when we call our end point.
```bash
curl localhost:8080/
# [{"name":"widget one"},{"name":"widget two"}]
```
## Reading The Request
You can also read from the request body and decode a json object back into a struct.
This is done by creating a new variable and then use the decoder to set its values.
```go
func Post(w http.ResponseWriter, req *http.Request) {
	decoder := json.NewDecoder(req.Body)
	i := &Item{}
	err := decoder.Decode(i)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		fmt.Fprint(w, err.Error())
		return
	}
	w.WriteHeader(http.StatusNoContent)
}
```
You can test this using curl to make a ```post``` request and some json.
```bash
curl -X POST -d "{\"name\": \"widget\"}" localhost:8080/
```
We can read parameters from our route using the following
```go
params := mux.Vars(req)
id := params["id"]
```
This will set ```id``` to a string value that we set in the router earlier.
```go
router.HandleFunc("/{id}", Get).Methods("GET")
router.HandleFunc("/{id}", Update).Methods("PUT")
```
## Conclusion
As demonstrated above, a simple micro service can be written in very few lines with Go.

