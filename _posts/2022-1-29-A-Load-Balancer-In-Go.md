---
layout: post
categories: posts
title: "Let's build a load balancer in Go"
tags: [Golang]
date-string: January 29 2022
---
# Building a backend load balancer in Go

## A brief introduction and motivation for this project:
As you can see on a few of the posts on this blog, I am a Computer Science student interested in backend programming and learning new technologies.
I recently accepted an internship at Okta, where I am told I will being mainly doing development in Go. My interviewer was convinced that if I can learn Ocaml (something I think I'll probably make a post about sometime soon), then Go should be no problem. Well I guess I gotta learn the language...

I decided to work build a few personal projects in Go in order to hone my skills and learn more about the language. Overall it has been a really positive experience. Turns out Go is everything I was in a language. I don't remember if I have written anything about this, but I really liked my time writing C at school. It felt like the coolest language I had encountered, with low level control and pretty significant efficiency. Obviously though, C is a bit of a tough pill to swallow. It is becoming less popular in industry and the headaches you encounter when it comes to memory management, along with how easy it is to brick a program or write something that is extremely easy to exploit, make C a bit of a losing pick in 2022. *Enter Go...* The best way I can describe Go is a middle ground between C and python, with an extremely fast compile time. Turns out the language was made as a modern succesor to C, geared around fast compile times, readability, safety, and concurrency.

Go is a statically typed (as all things should be), compiled (also as all things should be), garbage collected (as all things except Rust should be) language. It uses a sort of modernized C type syntax and gives you memory safe and impressively efficient control of low level functions. Anyway, I decided I would build a load balancer as it sounded like a cool example of distributed systems building in Go.
``` c
#include <stdio.h>
int main() {
   // A simple hello world program in C
   printf("Hello, World!");
   return 0;
}
```

``` go
package main
import(
  "fmt"
)
func main() {
   // The equivalent program in Go
   fmt.Printf("Hello, World!")
}
```
### Load Balancer
So here's the basic idea. A typical web server is one machine which takes requests from the user and sends back responses which are interpreted by the browser and displayed to the user. This is pretty straight forward, but we could spend a lot of time talking about the minutia. But a more interesting question is what happens when there are multiple servers which can provide the same service, and lots of users? When this happens we need to distribute the load across multiple proxies and provide the same response the user would expect. So let's do that.

## Building out Structs
Let's build out some structs to represent our servers and hold the important information about our servers. We'll start with a `Server` struct:
``` go
// Server struct with a url, status, mutex, and reverse proxy to handle requests
type Server struct {
	URL          *url.URL
	Alive        bool
	mux          sync.RWMutex
	ReverseProxy *httputil.ReverseProxy
}
```
We have a Url to represent our proxies url, a boolean for the status of the server, a mutext to allow concurrent reading and writing, and a reverse proxy, which I will explain later. Now let's build a `Servers` struct to hold all of our servers.
``` go
// Server set to maintain list of servers
type Servers struct {
	backends  []*Server
	currentId uint64
}
```
This one is pretty self explanatory. An array of `Server` pointers and an int to count the connection number.

Next let's create a list of our server urls, these can be any url you want, and we could easily change the logic to read from input or a file or something, but for our usecase it's hard coded.
``` go
// Predefined list of servers, this can easily be read from a file if wanted
var ServerList = []string{
	"http://localhost:5000",
	"http://localhost:5002",
	"http://localhost:5003",
	"http://localhost:5004",
	"http://localhost:5005",
}
```

## On to the functions
Ok, now we can start to write some functions. We can start by creating a function to add servers to our server list, which we need to declare as well.

``` go
var serverPool Servers

// Append servers to the server list
func (s *Servers) AddServer(backend *Backend) {
	s.backends = append(s.backends, backend)
}
```
This is just a function to append servers to our pool. If you don't understand what it's doing you should look into Array slices in Go. Moving on... We can now write a function to mark an individual server as alive or dead. The syntax here might be a little confusing, but basically we are extending to `Servers` type to have a new function which can be called on any `Servers` instance.
``` go
// Mark the status of a specified server as either alive or dead
func (s *Servers) MarkStatus(url *url.URL, alive bool) {
	for _, back := range s.backends {
		if back.URL.String() == url.String() {
			back.Alive = alive
			return
		}
	}
}
```
We are taking in a url and boolean status, then we look for that specific server in our pool and mark it accordingly.

## Setting and Checking

We need to be able to interact with our servers, and if they are alive we need to know that, if that status changes we need to reflect that somewhere. So we're going to write some functions to accomplish this.

``` go
// Set server as alive
func (b *Server) SetAlive(alive bool) {
	b.mux.Lock()
	b.Alive = alive
	b.mux.Unlock()
}

// Check if server is alive
func (b *Server) IsAlive() (alive bool) {
	b.mux.RLock()
	alive = b.Alive
	b.mux.RUnlock()
	return
}
```

This is where our mutex from before comes in. We use a mutext to synchronize our access of the server status, we don't want one routine setting the status of the server at the same time as another routine, and stuff like that. So we lock our mutex, read or write, and then unlock. Pretty straight forward. Look more into concurrency if this is confusing. 

Now that we can set and read, we need a way to actually determine if a server is alive or not. This is were we need to make some connections to test out the status. We will write a function to test our server's connection.

``` go
// Check if server is alive
func isServerAlive(url *url.URL) (alive bool) {
	alive = false
	connection, err := net.DialTimeout("tcp", url.String()[7:], 2*time.Second)
	if err != nil {
		log.Print(err)
		return
	}
	alive = true
	_ = connection.Close()
	return
}
```
This function takes a url, send a request to that url. If the request times out then it returns false for alive, if it connects then it returns true. Note the use of Go's interesting named return syntax. I just threw it in here because I had never used it, might not be the best for readability to be honest.

## Health Check
So now we can check the status of an individual server and set it as well, but when do we check... How often, how do we initialize this? The answer once again relies on goroutines. We will create a new one to check the status of each server and report back. We will set a timer for every 20 seconds and once the timer finishes it will run again. This way we can constantly know the status of each of our servers.

``` go
// Check the status of each server and log
func (s *Servers) HealthCheck() {
	for _, back := range s.backends {
		alive := isServerAlive(back.URL)
		back.Alive = alive
		var status string
		if alive {
			status = "ok"
		} else {
			status = "down"
		}
		log.Printf("%s [status: %s]\n", back.URL.String(), status)
	}
}

// Periodically run a health check
func healthCheck() {
	timer := time.NewTimer(20 * time.Second)
	for {
		select {
		case <-timer.C:
			log.Print("Starting health check...\n")
			serverPool.HealthCheck()
			log.Printf("End Health Check...\n")
		}
	}
}
```

The for loop in `healthCheck` will run until the program is terminated, and the select will run our `serverPool.HealthCheck()` everytime the timer finishes. The logic in `HealthCheck` is pretty straight forward, we check the status and report back for each server, then we'er done. Within our main function we start our this loop by running `go healthCheck()`.

## Actually balancing the load
So it's all well and good, but we're not actually distributing the load at all. We need a way to cycle through servers and use them according to some sort of criteria. For the sake of simplicity we will just use a round robin cycle. We always just choose the next alive server to handle our connection. Let's write a function to determine which server is next in line.

``` go
// Return the index of next server
func (s *Servers) NextIndex() int {
	return int(atomic.AddUint64(&s.currentId, uint64(1)) % uint64(len(s.backends)))
}

// Starting at next server look for next alive server. This distributes the load in a round robin manner
func (s *Servers) NextAlive() *Server {
	nextInd := s.NextIndex()

	loc := nextInd + len(s.backends)
	for i := nextInd; i < loc; i++ {
		ind := i % len(s.backends)

		if s.backends[ind].Alive {
			if i != nextInd {
				atomic.StoreUint64(&s.currentId, uint64(ind))
			}
			return s.backends[ind]
		}
	}
	return nil
}
```

In our `serverPool` we are storing an index which counts our connections. We can use that to determine what the next server to use is. We start at the next index and look for the next alive server. We then return a pointer to it so we know which server to connect to. We need to actually use the server though, so here is where the `loadBalancer` comes in.

``` go
// Serve the response through the next alive server
func loadBalancer(w http.ResponseWriter, r *http.Request) {
	alive := serverPool.NextAlive()
	if alive != nil {
		alive.ReverseProxy.ServeHTTP(w, r)
		return
	}
	http.Error(w, "Service Not Available", http.StatusServiceUnavailable)
}
```

We take the response sent to our input server and redirect it (server it via a reverse proxy) to the next alive server in our pool. If something doesn't work then we give the user an HTTP error stating that the service is not available.

## Puting it all together
We need a main function to put everything together, handle our requests, and actually delegate the work. first we will store our end point url so we can refer to it later
``` go
	uri, _ := url.Parse("http://127.0.0.1:8080")
	port := 8080
```
 We can then create a server object which will initiate all of the action on our network.
``` go
// Initialize server
server := http.Server{
  ConnState: cw.OnStateChange,
  // Add loadBalancer as function to direct requests
  Handler: http.HandlerFunc(loadBalancer),
  Addr:    fmt.Sprintf(":%d", port),
}
```
Now comes the interesting part. We need to loop through our `ServerList` from before and create a `Server` for each one. We will then attempt to connect to each one and set its's status. There is some variability with servers, so we will attempt to connect 3 times before we give up. In order to do this we will store our attempts count and retries count within the context of our HTTP request. We have to actually have a way to read and write to our context, so we will write a couple functions for this. We will also create a couple const iota ints to hold our attemps and retries.

``` go
// Atomically incrementing ints to count connection attemps and retries
const Attempts int = iota
const Retry int = iota

// Read retries from contett
func GetRetryFromContext(r *http.Request) int {
	if retry, ok := r.Context().Value(Retry).(int); ok {
		return retry
	}
	return 0
}

// Read attempst from context
func GetAttemptsFromContext(r *http.Request) int {
	if attempts, ok := r.Context().Value(Attempts).(int); ok {
		return attempts
	}
	return 0
}
```

Now we can put it all together and actually get our servers going and added to our pool.

``` go
// loop through list of servers and attempt to connect to each
	for _, str := range ServerList {
		url, _ := url.Parse(str)
		// proxy is where the request will be redirected to
		proxy := httputil.NewSingleHostReverseProxy(url)

		// Add an error handler to the proxy for when the connection refuses
		proxy.ErrorHandler = func(writer http.ResponseWriter, request *http.Request, err error) {
			retries := GetRetryFromContext(request)
			// Attempt to connect to backend 3 times, if not successful, mark it as down
			if retries < 3 {
				select {
				case <-time.After(10 * time.Millisecond):
					// store attempts/retries within context in the request object
					cont := context.WithValue(request.Context(), Retry, retries+1)
					proxy.ServeHTTP(writer, request.WithContext(cont))
				}
				return
			}
			// mark backend as dead
			serverPool.MarkStatus(uri, false)
			// read attempts from context of request
			attempts := GetAttemptsFromContext(request)
			cont := context.WithValue(request.Context(), Retry, retries+1)
			loadBalancer(writer, request.WithContext(cont))
		}
		serverPool.AddServer(&Server{
			URL:          url,
			Alive:        true,
			ReverseProxy: proxy,
		})
```

We wrap it all up with a simple `server.ListenAndServer()` and there we go. Our server is being load balanced... well it's not actually, because we don't have any proxy servers going. I won't go through it here, but it is very easy to write a small server with Flask to just print our it's name on a webpage so we know which proxy is handling our request. We can just create some instances of these servers and watch it all work. I'll just throw in the code for a server to make it easier:
``` python
from flask import Flask
app = Flask(__name__)
import sys

server_name = sys.argv[1]

@app.route('/')
def main():
    return server_name

if __name__ == '__main__':
   app.run(port=sys.argv[2])
```
And there you have it. A functional load balancer written in Go. You can view all the code put together (with a couple additions) [here](https://github.com/stefanhts/seasaw). I hope this was fun and helpful and all that. 

See you in the next one!