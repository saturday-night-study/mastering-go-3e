# net/http 패키지

해당 장에서는 `net/http`패키지를 사용해서 [HTTP 프로토콜](https://developer.mozilla.org/ko/docs/Web/HTTP/Overview)을 사용하는 실습 위주의 내용입니다. 기본적인 사용방법과 전화번호부 어플리케이션 업데이트를 통해 웹 어플리케이션을 만들어보고, 변환 된 어플리케이션과 통신하기 위해 커맨드라인 클라이언트도 만들어보겠습니다. :clap:

## 들어가기 전

### HTTP/1.1, HTTP/2.0, HTTP/3.0 프로토콜 소개

#### HTTP/1.1

1997년에 도입된 `HTTP/1.1`은 `HTTP`의 가장 오랜 버전으로, 여전히 널리 사용되고 있습니다. 이 버전은 `HTTP/1.0`에 비해 여러 가지 개선점을 제공합니다.

* **Persistent Connections**: 연결 유지 기능을 통해 여러 요청과 응답을 하나의 연결로 처리할 수 있게 되어 효율성이 향상되었습니다.
* **Pipelining**: 클라이언트가 여러 요청을 연속적으로 보내고, 서버가 순서대로 응답을 보낼 수 있는 기능을 추가하여 네트워크 지연 시간을 줄였습니다.
* **Chunked Transfers**: 데이터를 여러 조각으로 나누어 전송할 수 있는 기능을 도입하여 대용량 데이터 처리를 개선했습니다.

`HTTP/1.1`은 헤더 압축을 지원하지 않아 헤더가 큰 경우 많은 대역폭을 차지할 수 있으며, 하나의 연결에서 여러 요청을 동시에 보낼 수 없는 단점이 있습니다.

#### HTTP/2.0

2015년에 공식화된 `HTTP/2.0`는 `HTTP/1.1`의 성능 문제를 해결하기 위해 개발되었습니다. 이 버전은 다음과 같은 주요 특징을 지니고 있습니다.

* **Binary Framing Layer**: `HTTP/2.0`은 텍스트 기반의 `HTTP/1.1`과 달리 이진 프레이밍 구조를 사용하여 메시지를 보다 효율적으로 처리합니다.
* **Multiplexing**: 여러 요청과 응답을 동시에 같은 연결에서 처리할 수 있게 되어 리소스의 동시 다운로드가 가능해지고, 블록킹 문제를 해결했습니다.
* **Server Push**: 서버가 클라이언트의 요청을 기다리지 않고 미리 필요한 리소스를 푸시할 수 있어 페이지 로드 시간을 단축할 수 있습니다.
* **Header Compression**: HPACK 압축 기법을 사용하여 헤더 데이터를 효율적으로 압축하여 전송합니다.

`HTTP/2.0`는 성능 향상을 크게 이루었지만, 모든 트래픽이 하나의 연결을 사용하기 때문에 하나의 큰 파일 전송이 다른 전송을 지연시킬 수 있는 **"Head-of-Line Blocking" 문제**가 여전히 존재합니다.

#### HTTP/3.0

`HTTP/3.0`는 현재 최신 버전으로, 주로 **"Head-of-Line Blocking" 문제를 해결**하기 위해 **UDP 기반의 QUIC 프로토콜을 도입**했습니다. `HTTP/3.0`의 특징은 다음과 같습니다.

<details>

<summary>QUIC 프로토콜</summary>

**QUIC (Quick UDP Internet Connections)**은 원래 구글에 의해 개발되었고, 이후 IETF (인터넷 공학 태스크 포스)에 의해 표준화된 새로운 인터넷 전송 프로토콜입니다. 해당 프로토콜은 기존의 TCP + TLS + HTTP/2.0 프로토콜 스택의 단점을 극복하고자 설계되었습니다. 주요 목적은 웹 페이지의 로딩 시간을 단축하고, 전체적인 인터넷 성능을 향상 시키는 것입니다.&#x20;

#### UDP 기반

**TCP가 아닌 UDP를 기반**으로 합니다. UDP는 연결 지향적이지 않고, 가볍고 빠르며, 최소한의 통신 오버헤드만을 가집니다. 이는 **QUIC**가 초기 연결 설정 시간을 단축하고, 네트워크 변경 시 빠른 연결 복구를 가능하게 합니다.

#### 연결 설정 및 암호화

연결 설정과 암호화를 한 번의 과정으로 처리합니다. 전통적인 TCP + TLS 조합에서는 여러 차례의 손을 흔드는(handshaking) 과정이 필요하지만, **QUIC**는 0-RTT (Round-Trip Time) 또는 1-RTT 설정을 통해 이를 최소화합니다. 이는 웹 페이지의 로딩 속도를 빠르게 하고 사용자 경험을 개선합니다.

#### 스트림 멀티플렉싱

복수의 독립적인 스트림을 한 번의 연결에서 동시에 다룰 수 있습니다. 이 기능은 TCP의 Head-of-Line Blocking 문제를 해결합니다. **QUIC**에서는 각 스트림이 독립적으로 처리되기 때문에, 한 스트림의 패킷 손실이 다른 스트림에 영향을 주지 않습니다.

#### 연결 이전

네트워크 변경 (예: 사용자가 모바일 데이터에서 Wi-Fi로 변경할 때) 이 발생해도 연결을 유지할 수 있는 기능을 제공합니다. 이는 특히 모바일 환경에서 유용하며, 사용자가 네트워크를 바꾸더라도 세션이 끊기지 않도록 돕습니다.

#### 포워딩 보안

TLS 1.3을 기반으로 포워딩 보안을 제공합니다. 이는 과거의 통신이 노출되어도 현재의 통신 키가 안전하다는 것을 의미합니다. 사용자 데이터의 보안과 개인 정보 보호를 강화합니다.

#### 효율적인 오류 제어 및 혼잡 제어

패킷 손실 감지와 혼잡 제어 메커니즘을 내장하여, 네트워크 상황에 따라 데이터 전송률을 조정합니다. 이를 통해 효율적인 대역폭 사용이 가능하며, 네트워크 혼잡 시 성능 저하를 최소화합니다.

</details>

* **QUIC Protocol**: TCP 대신 `QUIC`를 사용하여 연결 초기화와 복구 시간을 단축하고, 전체적인 데이터 전송 효율을 향상시킵니다.
* **Improved Multiplexing**: `QUIC`는 개별 스트림의 데이터 손실이 다른 스트림에 영향을 미치지 않도록 설계되었기 때문에, 더욱 효율적인 데이터 스트리밍이 가능합니다.
* **Connection Migration**: 네트워크 변경이나 IP 주소 변경 시에도 연결을 유지할 수 있습니다.

`HTTP/3.0`은 여전히 개발 중이며 모든 웹 브라우저와 서버에서 지원되는 것은 아니지만, 미래의 인터넷 통신 표준으로 자리잡을 잠재력을 가지고 있습니다.

### HTTPS

HTTPS(Hypertext Transfer Protocol Secure)은 데이터 전달 과정에서 정보가 노출될 수 있는 부분을 SSL(Secure Socket Layer)를 통해서 문제를 해결하였습니다.

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption><p>브라우저내에서 https를 통한 연결 시 안전함을 표시</p></figcaption></figure>

HTTP 프로토콜과 가장 큰 차이점은 SSL 인증서를 통한 데이터 교환입니다. 아래의 그림과 같이 인증서를 통해 데이터를 암호화하면 해커가 해당 데이터를 중간 통신 과정에서 확인할 수 없게 됩니다.

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption><p>HTTP를 통한 해커의 데이터포워딩</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>HTTPS로 암호화 된 데이터의 포워딩</p></figcaption></figure>

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

## [http.Transport](https://go.dev/src/net/http/transport.go) 타입

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
