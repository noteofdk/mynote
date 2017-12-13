---
title: 'Go package: HTTP 服务器实现'
date: 2017-12-12 16:29:22
categories: [golang]
tags: [golang]
---

本文简单分析 go 1.9 http 服务器的实现。介绍 net/http 包中实现 HTTP 服务器的几个重要的函数和结构体的实现和用法。

<!--more-->

# hello world
可以用如下代码实现简单的 go 语言 HTTP 服务的 hello world：
```go
package main

import (
    "io"
    "net/http"
    "log"
)

// hello world, the web server
func HelloServer(w http.ResponseWriter, req *http.Request) {
    io.WriteString(w, "hello, world!\n")
}

func main() {
    http.HandleFunc("/hello", HelloServer)
    log.Fatal(http.ListenAndServe(":12345", nil))
}
```

最简单的 golang HTTP 服务器由 `http.HandleFunc(pattern string, handler func(ResponseWriter, *Request))` 做路由分发， `http.ListenAndServe(addr, hanlder)` 开启监听并启动服务。

## http.HandleFunc

```
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

HandleFunc 函数在 DefaultServeMux 中为指定的 pattern 注册 HTTP 服务处理函数 func(ResponseWriter, *Request) ，即由该 handler 处理对 pattern 的请求。

## http.ListenAndServe

代码如下：
```
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

ListenAndServe 监听 TCP 地址，调用 Server 处理该连接中的请求。如果 handler 为空，则使用 DefaultServeMux。

该函数，其实是默认 Server 的 ListenAndServe 方法的封装。

# ServeMux

ServeMux 是一个 HTTP 请求多路转接器。它将收到的 HTTP 请求的 URL 和已经注册的 pattern 做匹配，并调用最匹配的 pattern 对应的 hanlder 来处理该请求。

pattern 是固定的、以跟路径符号 `/` 开始的路径，如 "/favicon.ico"。也可以是跟路径的子树，如 "/images/"。匹配时，长 pattern 优先级更高。

以 `/` 结尾的 pattern 定义了一个子树。`/` 本身可以匹配任何 pattern。

如果已经注册了一个如 `/images/` 的子树 pattern，接收到一个如 `/images` 的不带最后 `/` 符号的请求时，ServeMux 会将该请求重定向到子树 pattern `/images/` 对应的处理器 handler（除非 `/images` 被单独注册）。

pattern 可以可选地指定域名。指定域名的 pattern 只会匹配该域名的 URL。指定域名的 pattern 优先级高于未指定的。

ServeMux also takes care of sanitizing the URL request path,redirecting any request containing . or .. elements or repeated slashes to an equivalent, cleaner URL.

```
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	hosts bool // whether any patterns contain hostnames
}

type muxEntry struct {
	explicit bool
	h        Handler
	pattern  string
}
```

ServeMux 的 ServeHTTP 方法分发 HTTP 请求到与其 URL 匹配度最高的 pattern 注册的处理器 h：mux.Handler(r) 返回匹配度最高的处理器 h，然后调用 h.ServeHTTP(w, r) 处理请求。

```
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	if r.RequestURI == "*" {
		if r.ProtoAtLeast(1, 1) {
			w.Header().Set("Connection", "close")
		}
		w.WriteHeader(StatusBadRequest)
		return
	}
	h, _ := mux.Handler(r)
	h.ServeHTTP(w, r)
}
```

# Server
Server 结构体定义启动 HTTP 服务器的参数
```
type Server struct {
	Addr      string
	Handler   Handler
	TLSConfig *tls.Config

	ReadTimeout time.Duration
	ReadHeaderTimeout time.Duration
	WriteTimeout time.Duration
	IdleTimeout time.Duration

	MaxHeaderBytes int

	TLSNextProto map[string]func(*Server, *tls.Conn, Handler)

	ConnState func(net.Conn, ConnState)

	ErrorLog *log.Logger
}
```

此处的 Handler 是个接口，意味着只要实现 ServeHTTP(ResponseWriter, *Request) 方法，就能作为 Server 的 Handler：
```
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

## ListenAndServe 方法

ListenAndServe 方法监听 Addr 字段指定的 TCP 连接，处理请求。接收的连接激活了 TCP keep-alives。如果 Addr 为空，则使用 ":http"。总是返回非空 error。
```
func (srv *Server) ListenAndServe() error {
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}
```

ListenAndServe 方法的实现主要有两个步骤：

- net.Listen("tcp", addr) 得到 net.Listener
- Serve 方法并发处理监听到的请求

下面看看 Serve 方法实现的主要步骤（只保留主要逻辑）：

```
func (srv *Server) Serve(l net.Listener) error {
	defer l.Close()
	
	...
	
	baseCtx := context.Background() // base is always background, per Issue 16220
	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, _ := l.Accept()
		c := srv.newConn(rw)
		go c.serve(ctx)
	}
}
```

最终，serve 方法调用 Server 中的 Handler 的 ServeHTTP 方法处理该请求。


# 注册函数
Handle 和 HandleFunc 将 HTTP 请求处理函数注册到 ServeMux 中
```
func Handle(pattern string, handler Handler) { 
    DefaultServeMux.Handle(pattern, handler) 
}
```

```
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```
使用方式略有差别。

`HandleFunc（pattern, handler)` 与 `Handle(pattern, HandlerFunc(handler))` 效果一样。例如：

```
http.Handle("/foo", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
}))

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
```

也可以从 ServeMux.HandleFunc 的实现看出来：
```
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}
```

# 比较
http 包有几个概念易混，特别分析对比一下。

## HandlerFunc

HandlerFunc 是一种类型，是以 func(ResponseWriter, *Request) 申明的函数。是个将普通函数转化为 HTTP 处理器的适配器。

```
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

用法：

定义普通函数 handlerFunc 如下，HandlerFunc(f) 即是一个调用 f 的 HTTP 处理器。
```
func f(w http.ResponseWriter, r *http.Request) { ... }
```

## Handler
Handler 是个接口。功能如其名：处理器。

http 包中实现 ServeHTTP(ResponseWriter, *Request) 方法即拥有处理 HTTP 请求的能力。
```
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
HandlerFunc 类型和 ServeMux 结构体都实现了 ServeHTTP(w ResponseWriter, r *Request) 方法， 也就是说，HandlerFunc 类型和 ServeMux 结构体都是一种 Handler。

HandlerFunc 的 ServeHTTP 方法直接调用自身，处理 HTTP 请求。

ServeMux 调用与请求路径匹配的 `func(ResponseWriter, *Request)` 类型的处理函数。

同样的，我们可以自己定义一个实现 ServeHTTP(ResponseWriter, *Request) 方法的结构体，定制我们自己的 HTTP 处理器。第三方的路由处理器（如 mux, httprouter 等）都是 Handler 的一种实现。

Handle 和 HandleFunc 的作用就是将 HTTP 请求处理函数注册到 ServeMux 中。
 
