# 전화번호부 애플리케이션 업데이트

## 개념

go 에서는 기본 패키지로 HTTP 서버를 제공합니다. 이때 사용되는 구조체들은 http 패키지 안에 모두 정의되어 있는데요. net 패키지를 이용하여 TCP 통신을 한 내용을 HTTP 정의에 맞게 해석하는 내용이 이 패키지 안에 작성되어있습니다.

<구조체>

```go
type Server struct {  
    Handler Handler 
    ReadTimeout time.Duration  
    ConnContext func(ctx context.Context, c net.Conn) context.Context  
    
    inShutdown atomic.Bool // true when server is in shutdown  

    disableKeepAlives atomic.Bool  
    nextProtoOnce     sync.Once // guards setupHTTP2_* init  
    nextProtoErr      error     // result of http2.ConfigureServer if used  
  
    mu         sync.Mutex  
    listeners  map[*net.Listener]struct{}  
    activeConn map[*conn]struct{}  
    onShutdown []func()  
  
    listenerGroup sync.WaitGroup  
}

func (srv *Server) ListenAndServe() error {  
    if srv.shuttingDown() {  
       return ErrServerClosed  
    }  
    addr := srv.Addr  
    if addr == "" {  
       addr = ":http"  
    }
      
	// tcp 통신을 연결한다.
    ln, err := net.Listen("tcp", addr)  
    if err != nil {  
       return err  
    }  
    return srv.Serve(ln)  
}
```

<사용 방법>

```go
// 주소, 핸들러, 타임아웃 정보등을 설정합니다.
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}

// ListenAndServe 를 통해 서버를 시작합니다.
log.Fatal(s.ListenAndServe())
```

```go
// 핸들링 한 내용을 분석하기 위한 구조체 
type ServeMux struct {  
    mu       sync.RWMutex  
    tree     routingNode  
    index    routingIndex  
    patterns []*pattern  // TODO(jba): remove if possible  
    mux121   serveMux121 // used only when GODEBUG=httpmuxgo121=1  
}

func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {  
    // 디버그용
    if use121 {  
       return mux.mux121.findHandler(r)  
    }  
    h, p, _, _ := mux.findHandler(r)  
    return h, p  
}

func (mux *ServeMux) findHandler(r *Request) (h Handler, patStr string, _ *pattern, matches []string) {  
    var n *routingNode  
    host := r.URL.Host  
    escapedPath := r.URL.EscapedPath()  
    path := escapedPath  
    // CONNECT requests are not canonicalized.  
    if r.Method == "CONNECT" {  
       // If r.URL.Path is /tree and its handler is not registered,  
       // the /tree -> /tree/ redirect applies to CONNECT requests       // but the path canonicalization does not.       _, _, u := mux.matchOrRedirect(host, r.Method, path, r.URL)  
       if u != nil {  
          return RedirectHandler(u.String(), StatusMovedPermanently), u.Path, nil, nil  
       }  
       // Redo the match, this time with r.Host instead of r.URL.Host.  
       // Pass a nil URL to skip the trailing-slash redirect logic.       n, matches, _ = mux.matchOrRedirect(r.Host, r.Method, path, nil)  
    } else {  
       // All other requests have any port stripped and path cleaned  
       // before passing to mux.handler.       host = stripHostPort(r.Host)  
       path = cleanPath(path)  
  
       // If the given path is /tree and its handler is not registered,  
       // redirect for /tree/.       var u *url.URL  
       n, matches, u = mux.matchOrRedirect(host, r.Method, path, r.URL)  
       if u != nil {  
          return RedirectHandler(u.String(), StatusMovedPermanently), u.Path, nil, nil  
       }  
       if path != escapedPath {  
          // Redirect to cleaned path.  
          patStr := ""  
          if n != nil {  
             patStr = n.pattern.String()  
          }  
          u := &url.URL{Path: path, RawQuery: r.URL.RawQuery}  
          return RedirectHandler(u.String(), StatusMovedPermanently), patStr, nil, nil  
       }  
    }  
    if n == nil {  
       // We didn't find a match with the request method. To distinguish between  
       // Not Found and Method Not Allowed, see if there is another pattern that       // matches except for the method.       allowedMethods := mux.matchingMethods(host, path)  
       if len(allowedMethods) > 0 {  
          return HandlerFunc(func(w ResponseWriter, r *Request) {  
             w.Header().Set("Allow", strings.Join(allowedMethods, ", "))  
             Error(w, StatusText(StatusMethodNotAllowed), StatusMethodNotAllowed)  
          }), "", nil, nil  
       }  
       return NotFoundHandler(), "", nil, nil  
    }  
    return n.handler, n.pattern.String(), n.pattern, matches  
}

mux := http.NewServeMux()  
```

참고 자료

* https://pkg.go.dev/net/http

## 실습

#### 라우터 구현

```go
package main  
  
import (  
    "fmt"  
    "mastering-go/common_phonebook/cmd/www_phonebook/handlers"    
    "net/http"    
    "time"
)  
  
func main() {  
  
    addr := "localhost:8080"  
  
    mux := http.NewServeMux()  
    s := &http.Server{  
       Addr:         addr,  
       Handler:      mux,  
       IdleTimeout:  10 * time.Second,  
       ReadTimeout:  time.Second,  
       WriteTimeout: time.Second,  
    }  
  
    mux.Handle("/list", http.HandlerFunc(handlers.List))  
    mux.Handle("/add/", http.HandlerFunc(handlers.Add))  
    mux.Handle("/add", http.HandlerFunc(handlers.Add))  
    mux.Handle("/search", http.HandlerFunc(handlers.Search))  
    mux.Handle("/search/", http.HandlerFunc(handlers.Search))  
    mux.Handle("/delete/", http.HandlerFunc(handlers.Del))  
    mux.Handle("/", http.HandlerFunc(handlers.Default))  
  
    fmt.Println("Ready to serve at -> ", addr)  
    err := s.ListenAndServe()  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
}
```

#### 핸들러 구현

```go
func List(w http.ResponseWriter, r *http.Request) {  
    repo := phonebook.NewRepositoryInMemory()  
    pb, err := repo.GetEntries()  
    if err != nil {  
       w.WriteHeader(http.StatusInternalServerError)  
       return  
    }  
  
    jsonText, err := json.Marshal(pb)  
    if err != nil {  
       w.WriteHeader(http.StatusInternalServerError)  
       return  
    }  
  
    w.WriteHeader(http.StatusOK)  
    _, err = w.Write(jsonText)  
    if err != nil {  
       return  
    }  
}

func Add(w http.ResponseWriter, r *http.Request) {  
    paramStr := strings.Split(r.URL.Path, "/")  
    fmt.Println("Path:", paramStr)  
  
    if len(paramStr) < 5 {  
       w.WriteHeader(http.StatusNotFound)  
       _, _ = fmt.Fprintln(w, "Not enough arguments: "+r.URL.Path)  
       return  
    }  
  
    name := paramStr[2]  
    surname := paramStr[3]  
    tel := strings.ReplaceAll(paramStr[4], "-", "")  
  
    entry := phonebook.NewEntry(name, surname, tel)  
    if !entry.MatchTel(tel) {  
       w.WriteHeader(http.StatusBadRequest)  
       Body := "Invalid telephone number\n"  
       _, _ = fmt.Fprintf(w, "%s", Body)  
       return  
    }  
  
    repo := phonebook.NewRepositoryInMemory()  
    err := repo.AddEntry(*entry)  
    if err != nil {  
       w.WriteHeader(http.StatusInternalServerError)  
       Body := "Failed to add record\n"  
       _, _ = fmt.Fprintf(w, "%s", Body)  
       return  
    }  
  
    log.Println("Serving:", r.URL.Path, "from", r.Host)  
    Body := "New record added successfully\n"  
    w.WriteHeader(http.StatusOK)  
    _, _ = fmt.Fprintf(w, "%s", Body)  
    log.Println("Serving:", r.URL.Path, "from", r.Host)  
}


func Del(w http.ResponseWriter, r *http.Request) {  
    paramStr := strings.Split(r.URL.Path, "/")  
    fmt.Println("Path:", paramStr)  
  
    if len(paramStr) < 3 {  
       w.WriteHeader(http.StatusNotFound)  
       _, _ = fmt.Fprintln(w, "Not enough arguments: "+r.URL.Path)  
       return  
    }  
  
    tel := strings.ReplaceAll(paramStr[2], "-", "")  
  
    repo := phonebook.NewRepositoryInMemory()  
    if repo.DeleteEntry(tel) != nil {  
       w.WriteHeader(http.StatusInternalServerError)  
       _, _ = fmt.Fprintln(w, "Failed to delete record")  
       return  
    }  
  
    w.WriteHeader(http.StatusOK)  
    _, _ = w.Write([]byte("Delete Handler\n"))  
}

func Search(w http.ResponseWriter, r *http.Request) {  
    paramStr := strings.Split(r.URL.Path, "/")  
    fmt.Println("Path:", paramStr)  
  
    if len(paramStr) < 3 {  
       w.WriteHeader(http.StatusNotFound)  
       _, _ = fmt.Fprintln(w, "Not enough arguments: "+r.URL.Path)  
       return  
    }  
  
    tel := strings.ReplaceAll(paramStr[2], "-", "")  
  
    repo := phonebook.NewRepositoryInMemory()  
    entry, err := repo.GetEntry(tel)  
    if err != nil {  
       return  
    }  
  
    jsonText, err := json.Marshal(entry)  
    if err != nil {  
       w.WriteHeader(http.StatusInternalServerError)  
       return  
    }  
  
    w.WriteHeader(http.StatusOK)  
    _, _ = w.Write(jsonText)  
}
```

```
```
