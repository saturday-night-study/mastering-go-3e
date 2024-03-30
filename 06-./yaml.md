# YAML 다루기

## 개요

Go 표준에서는 YAML을 다루지 않고 있기 때문에 별도의 라이브러리를 통해서 다룰 수 있습니다.

* [go-gypsy](https://github.com/kylelemons/go-gypsy)
* [go-yaml (gopkg.in, v3)](https://github.com/go-yaml/yaml)
* [goccy](https://github.com/goccy/go-yaml)

해당 파트에서는 `go-yaml` 패키지를 사용하였습니다.

## 예시코드

```go
package main

import (
	"fmt"
	"gopkg.in/yaml.v3"
)

var yamlfile = `
image: Golang
matrix:
  docker: python
  version: [2.7, 3.9]
`

type Mat struct {
	DockerImage string    `yaml:"docker"`
	Version     []float32 `yaml:",flow"`
}

type YAML struct {
	Image  string
	Matrix Mat
}

func main() {
	data := YAML{}

	err := yaml.Unmarshal([]byte(yamlfile), &data)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("After Unmarshal (Structure):\n%v\n\n", data)

	d, err := yaml.Marshal(&data)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("After Marshal (YAML code):\n%s\n", string(d))
}
```

이후 코드를 실제 실행하면 아래와 같은 값을 확인할 수 있습니다.

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption><p>결과 예시 화면</p></figcaption></figure>
