# net/http 패키지

해당 장에서는 `net/http`패키지를 사용해서 [HTTP 프로토콜](https://developer.mozilla.org/ko/docs/Web/HTTP/Overview)을 사용하는 실습 위주의 내용입니다. 기본적인 사용방법과 전화번호부 어플리케이션 업데이트를 통해 웹 어플리케이션을 만들어보고, 변환 된 어플리케이션과 통신하기 위해 커맨드라인 클라이언트도 만들어보겠습니다. :clap:

## 개요

`net/http` 패키지는 웹 서버와 클라이언트를 개발하는 데 필요한 기능을 제공합니다. 예를 들어 `http.Get()`과 `http.NewRequest()`를 이용하면 클라이언트에서 HTTP 요청을 보낼 수 있고 `http.ListenAndServe()` 함수를 이용하면 명시 된 IP 주소와 TCP 포트로 웹 서버를 시작할 수 있습니다.

또한 `http.HandleFunc()`를 통해 지원하는 URL 및 해당 URL로 들어오는 요청을 처리할 함수를 정의할 수 있습니다. 다음 [#http.response](net-http.md#http.response "mention"), [#http.request](net-http.md#http.request "mention"), [#http.transport](net-http.md#http.transport "mention")절에서는 `net/http` 패키지의 중요한 타입 세 가지를 설명하면서 실습도 함께 진행해보도록 하겠습니다.

## [http.Response 타입](https://golang.org/src/net/http/response.go)

`http.Response`는 HTTP 요청에 대한 응답을 표현하는 타입입니다. `http.Client`와 `http.Transport`는 응답 헤더가 도착할 때 `http.Response` 값을 반환합니다.&#x20;

{% code title="net/http/response.go" lineNumbers="true" %}
```go
type Response struct {
	Status     string // 상태		(e.g. "200 OK")
	StatusCode int    // 상태코드		(e.g. 200)
	Proto      string // 프로토콜 버전	(e.g. "HTTP/1.1")
	ProtoMajor int    // 프로토콜 메이저 버전	(e.g. 1)
	ProtoMinor int    // 프로토콜 마이너 버전  (e.g. 1)

	Header Header
	Body io.ReadCloser
	ContentLength int64

	TransferEncoding []string

	Close bool
	
	Uncompressed bool

	Trailer Header

	Request *Request

	TLS *tls.ConnectionState
}
```
{% endcode %}

위 구조체를 보면 많은 내용을 담고 있지만 사실 `Status`, `StatusCode`, `Header`, `Body`에 대한 내용을 일반적으로 많이 다루기 때문에 더 자세한 내용은 위 `response.go` 코드를 참고하시면 됩니다.

{% hint style="info" %}
**표준 Go 라이브러리 패키지에는 대부분 주석이 매우 상세하게 적혀있음**으로 코드를 참고하면서 개발하면 더욱 많은 정보를 알 수 있음
{% endhint %}

## [http.Request 타입](https://go.dev/src/net/http/request.go)

`http.Request`는 서버가 수신 또는 클라이언트가 송신하기 위해 사용되는 구조체입니다.

{% code title="net/http/request.go" lineNumbers="true" %}
```go
type Request struct {
	Method string
	
	URL *url.URL

	Proto      string
	ProtoMajor int
	ProtoMinor int

	// Header contains the request header fields either received
	// by the server or to be sent by the client.
	//
	// If a server received a request with header lines,
	//
	//	Host: example.com
	//	accept-encoding: gzip, deflate
	//	Accept-Language: en-us
	//	fOO: Bar
	//	foo: two
	//
	// then
	//
	//	Header = map[string][]string{
	//		"Accept-Encoding": {"gzip, deflate"},
	//		"Accept-Language": {"en-us"},
	//		"Foo": {"Bar", "two"},
	//	}
	Header Header

	Body io.ReadCloser

	GetBody func() (io.ReadCloser, error)

	ContentLength int64

	TransferEncoding []string

	Close bool

	Host string

	Form url.Values

	PostForm url.Values

	MultipartForm *multipart.Form

	Trailer Header

	RemoteAddr string

	RequestURI string

	TLS *tls.ConnectionState

	Cancel <-chan struct{}

	Response *Response
}
```
{% endcode %}

`Body` 필드는 `Request`의 `본문(body)`을 가지고 있는데, 이때 `GetBody()` 함수를 호출하면 해당 본문의 복제본을 만들어서 사용할 수도 있습니다.

## http.Transport 타입

마지막으로 `http.Transport` 타입은 HTTP 연결을 제어하기 위한 가장 낮은 수준의 타입입니다. 따라서 정의가 길고 복잡도가 높습니다.

{% code title="net/http/transport.go" lineNumbers="true" %}
```go
type Transport struct {
	idleMu       sync.Mutex
	closeIdle    bool
	idleConn     map[connectMethodKey][]*persistConn
	idleConnWait map[connectMethodKey]wantConnQueue
	idleLRU      connLRU

	reqMu       sync.Mutex
	reqCanceler map[cancelKey]func(error)

	altMu    sync.Mutex
	altProto atomic.Value 
	connsPerHostMu   sync.Mutex
	connsPerHost     map[connectMethodKey]int
	connsPerHostWait map[connectMethodKey]wantConnQueue

	Proxy func(*Request) (*url.URL, error)

	OnProxyConnectResponse func(ctx context.Context, proxyURL *url.URL, connectReq *Request, connectRes *Response) error

	DialContext func(ctx context.Context, network, addr string) (net.Conn, error)

	Dial func(network, addr string) (net.Conn, error)

	DialTLSContext func(ctx context.Context, network, addr string) (net.Conn, error)

	DialTLS func(network, addr string) (net.Conn, error)
	
	TLSClientConfig *tls.Config

	TLSHandshakeTimeout time.Duration

	DisableKeepAlives bool

	DisableCompression bool

	MaxIdleConns int

	MaxIdleConnsPerHost int

	MaxConnsPerHost int

	IdleConnTimeout time.Duration

	ResponseHeaderTimeout time.Duration

	ExpectContinueTimeout time.Duration

	TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper

	ProxyConnectHeader Header

	GetProxyConnectHeader func(ctx context.Context, proxyURL *url.URL, target string) (Header, error)

	MaxResponseHeaderBytes int64

	WriteBufferSize int

	ReadBufferSize int
}
```
{% endcode %}

이러한 사용하기 복잡한 구조체 대신 쉽게 사용할 수 있도록 별도의 고수준 구조체인 `http.Client`를 제공하고 있습니다.

`http.Client`는 각각 `Transport` 필드를 가지고 있으며, 필드의 값이 `nil`이라면 `DefaultTransport`가 적용 됩니다. ([https://go.dev/src/net/http/transport.go](https://go.dev/src/net/http/transport.go) 43번 라인 참고)
