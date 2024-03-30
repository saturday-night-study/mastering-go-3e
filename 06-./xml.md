# XML 다루기

## 개요

해당 파트에서는 Go에서 [XML](https://ko.wikipedia.org/wiki/XML) 데이터를 다루는 방법을 간략하게 설명합니다. GO에서 XML을 다루는 방식은 JSON을 다루는 방법과 동일하게 동작합니다.

Go `구조체(struct)`에 XML 관련 태그를 추가하여 XML 레코드를 `encoding/xml` 패키지의 `xml.Marshal()` 함수와 `xml.Unmarshal()` 함수를 사용하여 직렬화 및 역직렬화 할 수 있습니다.

### XML 변환예시코드

```go
package main

import (
    "encoding/xml"
    "fmt"
)

type Employee struct {
    XMLName      xml.Name  `xml:"employee"`
    ID           int       `xml:"id,attr"`
    FirstName    string    `xml:"name>first"`
    LastName     string    `xml:"name>last"`
    Height       float32   `xml:"height,omitempty"`
    Comment      string    `xml:",comment"`
}

func main() {
    emp := &Employee{
        ID: 951220,
        FirstName: "Karriz",
        LastName: "LEE",
    }
    
    emp.Comment = "Blockchain Developer"
    
    out, err := xml.MarshalIndent(&emp, " ", " ")
    
    out = []byte(xml.Header + string(out))
    fmt.Println("%s\n",out)
}
```

예시 코드에서 볼 수 있듯이 XML 태그는 JSON 태그와 약간의 차이점이 존재합니다.

필드의 이름은 `xml.Name`으로 정의하고 `",comment"` 태그의 경우에는 **XML의 주석을 의미**합니다.  또한 `>` 가 있는 경우에는 **왼쪽의 태그에 오른쪽의 태그가 children 노드가 됨**을 의미합니다.&#x20;

위의 예시 코드를 출력하면,  아래와 같은 형태로 출력 됩니다. :smile:

<pre class="language-xml"><code class="lang-xml"><strong>&#x3C;?xml version="1.0" encoding="UTF-8"?>
</strong><strong>&#x3C;employee id="951220">
</strong>    &#x3C;name>
        &#x3C;first>Karriz&#x3C;/first>
        &#x3C;last>LEE&#x3C;/last>
    &#x3C;/name>
    &#x3C;!-- Blockchain Developer -->
&#x3C;/employee>
</code></pre>

## JSON과 XML 변환

JSON과 XML은 위와 같이 직렬화, 역직렬화를 할 수 있기 때문에 JSON -> struct -> XML 또는 XML -> struct -> JSON 과 같은 형태로 변환이 가능하게 됩니다.

### JSON -> XML 변환예시코드

```go
package main

import (
	"encoding/json"
	"encoding/xml"
	"fmt"
	"os"
)

type XMLrec struct {
	Name    string `xml:"username"`
	Surname string `xml:"surname,omitempty"`
	Year    int    `xml:"creationyear,omitempty"`
}

type JSONrec struct {
	Name    string `json:"username"`
	Surname string `json:"surname,omitempty"`
	Year    int    `json:"creationyear,omitempty"`
}

func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Need XML or JSON input")
		return
	}
	// This can be a JSON or an XML record
	input := []byte(arguments[1])
	fmt.Println(string(input))

	// Check if it is XML
	checkJSON := false
	tempXML := XMLrec{}
	err := xml.Unmarshal(input, &tempXML)
	if err != nil {
		checkJSON = true
	} else {
		tempJSON := JSONrec{Name: tempXML.Name}
		if tempXML.Surname != "" {
			tempJSON.Surname = tempXML.Surname
		}
		if tempXML.Year != 0 {
			tempJSON.Year = tempXML.Year
		}
		s, err := json.Marshal(&tempJSON)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(string(s))
		return
	}

	if !checkJSON {
		return
	}

	// ELSE Check if it is JSON
	tempXML = XMLrec{}
	tempJSON := JSONrec{}
	err = json.Unmarshal(input, &tempJSON)
	if err != nil {
		fmt.Println("Not valid input")
		return
	} else {
		tempXML = XMLrec{Name: tempJSON.Name}
		if tempJSON.Surname != "" {
			tempXML.Surname = tempJSON.Surname
		}
		if tempJSON.Year != 0 {
			tempXML.Year = tempJSON.Year
		}
		s, err := xml.Marshal(&tempXML)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Println(string(s))
	}
}
```

위 코드를 통해 XML 레코드를 JSON 레코드로 변환할 수 있습니다.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>실제 예시 출력 화면 (XML -> JSON)</p></figcaption></figure>
