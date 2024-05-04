# 웹 서버 생성

해당 파트에서는 Go로 간단한 웹 서버를 개발해보면서 웹 어플리케이션의 동작과정을 함께 확인해보도록 하겠습니다.

{% hint style="info" %}
Go 웹 서버도 많은 일을 효율적이면서 안전하게 수행할 수 있지만, 고수준의 서버를 제작 해야한다면 상용 웹 서버(Apache, Nginx, Caddy 등...)를 활용하는 것이 좋음
{% endhint %}

## 개요

해당 파트에서 소개하는 웹 서버가 `HTTPS`가 아닌 `HTTP`를 쓰는 이유는 **Go 웹 서버의 경우 마이크로서비스 형태로 컨테이너에 배포**하게 되는데, 이때 실질적으로 사용자의 연결 지점은 Apache, Nginx와 같은 웹 서버를 통해 전달받게 되고 웹 서버를 통해 받은 처리는 `Proxy`를 통하여 실제 어플리케이션으로 전달하는 방식으로 사용 됩니다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>HTTPS 부분과 WAS(Web Application Server)의 예시 개념도</p></figcaption></figure>

`net/http` 패키지는 웹 서버와 클라이언트를 개발할 수 있는 함수와 데이터 타입들을 제공합니다. `http.Set()`과 `http.Get()` 메서드는 `HTTP` 및 `HTTPS` 요청을 생성하는데 사용할 수 있고 `http.ListenAndServe()`는 들어오는 요청을 사용자 정의 핸들러 함수를 이용해 처리할 수 있게 만들어줍니다.

대부분의 웹 서비스는 여러 가지의 End-Point를 가지고 있으므로 각각의 End-Point로 들어오는 요청을 처리할 여러 함수가 필요하게 됩니다.&#x20;

이를 처리하기 위해서는 `http.HandleFunc()`를 통해 여러 End-Point 지점에 대한 요청을 처리해주는 `핸들러(Handler)함수`를 정의할 수 있습니다.&#x20;

{% code title="wwwServer.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"net/http"
	"os"
	"time"
)

func myHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Serving: %s\n", r.URL.Path)
	fmt.Printf("Served: %s\n", r.Host)
}

func timeHandler(w http.ResponseWriter, r *http.Request) {
	t := time.Now().Format(time.RFC1123)
	Body := "The current time is:"
	fmt.Fprintf(w, "<h1 align=\"center\">%s</h1>", Body)
	fmt.Fprintf(w, "<h2 align=\"center\">%s</h2>\n", t)

	fmt.Fprintf(w, "Serving: %s\n", r.URL.Path)
	fmt.Printf("Served time for: %s\n", r.Host)
}

func main() {
	HOST := ":1220"
	arguments := os.Args
	if len(arguments) != 1 {
		HOST = ":" + arguments[1]
	}
	fmt.Println("Using host: ", HOST)

	http.HandleFunc("/time", timeHandler)
	http.HandleFunc("/", myHandler)

	err := http.ListenAndServe(HOST, nil)
	if err != nil {
		fmt.Println(err)
		return
	}
}
```
{% endcode %}

실행 후 실제로 localhost:1220 으로 접근하면 핸들러 함수에서 제공한 응답 값이 웹 화면에 표시되는 것을 확인할 수 있습니다.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>localhost:1220에 접근한 모습</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>localhost:1220/time에 접근한 모습</p></figcaption></figure>
