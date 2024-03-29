# JSON 데이터 다루기

## 개요

Go 언어에는 `JSON(JavaScript Object Notation)`을 다루기 위해서 표준 라이브러리인 `encoding/json` 라이브러리를 제공하고 있습니다. 또한 `태그(tag)`기능을 활용해 구조체에서 JSON 필드를 지원할 수 있고, 이는 아래 [#struct-json](json.md#struct-json "mention") 절에서 더욱 자세하게 확인할 수 있습니다.

태그만 이용한다면 JSON 레코드를 인코딩하거나 디코딩할 수 있지만, 먼저 JSON 레코드를 마샬 & 언마샬링하는 방법에 대해서 먼저 알아보도록 하겠습니다.

## `마샬(Marshal)`과 `언마샬(Unmarshal)`

마샬링과 언마샬링은 구조체를 이용해 JSON을 다루기 위한 아주 중요한 절차입니다.

`마샬링(Marshaling)`은 구조체를 JSON 레코드로 변환하는 과정입니다. 보통 JSON 레코드 형태로 네트워크 통신 또는 물리 디스크 저장소에 저장될 때 필요합니다.

`언마샬링(Unmarshaling)`은 \[]byte 형태의 JSON 레코드를 구조체로 변환하는 과정입니다. 보통 네트워크 통신 또는 물리 디스크 저장소에서 데이터를 읽어올 때 사용 됩니다.

{% hint style="info" %}
JSON 레코드와 구조체를 변환할 때 생기는 흔한 버그 중하나로는 필드를 외부로 노출(Export)하지 않아서 생기는 버그가 많다.

마샬 또는 언마샬링이 제대로 되지 않는다면 디버깅 때 이 부분 부터 확인해보자.
{% endhint %}

### 예시코드

바로 예시 코드를 통해서 직접 확인해보겠습니다. 아래 코드의 메타데이터가 알려주는 것은 `UseAll` 구조체의 `Name` 필드가 JSON 레코드에서 `username`으로 변환 됨을 확인할 수 있습니다. 이 정보들은 JSON 데이터를 마샬링 또는 언마샬링할 때 사용 됩니다.

#### Marshal 예시

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UseAll struct {
    Name        string    `json:"username"`
    Surname     string    `json:"usrname"`
    Year        int       `json:"created"`
}

func main() {
    useall := UseAll{Name: "Karriz", Surname: "LEE", Year: 1995}
    
    /*
            // 실제 json 레코드형태
            {
                "username": "Karriz",
                "surname": "LEE",
                "created": 1995
            }
    */
    
    // 구조체(struct) -> byte slice로 변환
    t, err := json.Marshal(&useall)
}
```

#### UnMarshal 예시

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UseAll struct {
    Name        string    `json:"username"`
    Surname     string    `json:"usrname"`
    Year        int       `json:"created"`
}

func main() {
    useall := UseAll{Name: "Karriz", Surname: "LEE", Year: 1995}
    
    /*
            // 실제 json 레코드형태
            {
                "username": "Karriz",
                "surname": "LEE",
                "created": 1995
            }
    */
    
    // 구조체(struct) -> byte slice로 변환
    t, err := json.Marshal(&useall)
    
    if err != nil {
        fmt.Println(err)
        return
    }
    
    // byte slice t -> json 레코드 변환 후 구조체로 할당
    tmp := UseAll{}
    err = json.Unmarshal(t, &temp);
}
```

## 구조체(struct)와 JSON

### 구조체의  JSON 메타데이터 옵션

구조체 변환시 빈 필드를 제외하고 싶다면 다음 코드처럼 `omitempty` 옵션을 사용하면 됩니다.

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UseAll struct {
    Name        string    `json:"username"`
    Surname     string    `json:"surname"`
    Year        int       `json:"created,omitempty"`
}

func main() {
    useall := UseAll{Name: "Karriz", Surname: "LEE"}
    
    /*
            // 실제 json 레코드형태
            {
                "username": "Karriz",
                "surname": "LEE"
            }
    */
    
    // 구조체(struct) -> byte slice로 변환
    t, err := json.Marshal(&useall)
    
    if err != nil {
        fmt.Println(err)
        return
    }
    
    // byte slice t -> json 레코드 변환 후 구조체로 할당
    tmp := UseAll{}
    err = json.Unmarshal(t, &temp);
}
```

또는 민감한 정보이기에 JSON 레코드에는 기록하고 싶지 않다면, `-` 옵션을  활용하면 됩니다.

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UseAll struct {
    Name        string    `json:"username"`
    Surname     string    `json:"surname"`
    Year        int       `json:"created,omitempty"`
    Pass        string    `json:"-"`    // json 레코드 포함 X
}

func main() {
    useall := UseAll{Name: "Karriz", Pass: " PASSWORD"}
    
    /*
            // 실제 json 레코드형태
            {
                "username": "Karriz",
                "surname": ""
            }
    */
    
    // 구조체(struct) -> byte slice로 변환
    t, err := json.Marshal(&useall)
    
    if err != nil {
        fmt.Println(err)
        return
    }
    
    // byte slice t -> json 레코드 변환 후 구조체로 할당
    tmp := UseAll{}
    err = json.Unmarshal(t, &temp);
}
```

### 스트림 형태로 JSON 데이터 읽고 쓰기

처리하고 싶은 JSON 레코드의 구조체가 슬라이스 형태로 있을 때 해당 레코드를 하나씩 처리한다면 매우 비효율적일 것입니다. 이를 **해결하기 위해**서 Go 언어는 여러  JSON 레코드를  처리하기 위해서 **스트림 형태로 JSON 데이터를 처리할 수 있는 방법을 제공**하고 있습니다.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"os"
	"runtime"
	"runtime/pprof"
	"time"
)

type Data struct {
	Key string `json:"key"`
	Val int    `json:"value"`
}

var serializedData []byte

func random(min, max int) int {
	return rand.Intn(max-min) + min
}

var MIN = 0
var MAX = 26

func getString(l int64) string {
	startChar := "A"
	temp := ""
	var i int64 = 1
	for {
		myRand := random(MIN, MAX)
		newChar := string(startChar[0] + byte(myRand))
		temp = temp + newChar
		if i == l {
			break
		}
		i++
	}
	return temp
}

// DeSerialize decodes a serialized slice with JSON records
func DeSerialize(e *json.Decoder, slice interface{}) error {
	return e.Decode(slice)
}

// Serialize serializes a slice with JSON records
func Serialize(e *json.Encoder, slice interface{}) error {
	return e.Encode(slice)
}

func serialize(buf *bytes.Buffer, DataRecords []Data, elapsedSerialize *time.Duration) {
	// Start CPU profiling for serialization
	fSer, err := os.Create("cpu_serialize.prof")
	if err != nil {
		log.Fatal("Could not create CPU profile for serialization: ", err)
	}
	pprof.StartCPUProfile(fSer)

	startSerialize := time.Now() // Start time serialization

	encoder := json.NewEncoder(buf)
	err = Serialize(encoder, DataRecords)
	if err != nil {
		log.Fatal("Serialization error: ", err)
		return
	}

	*elapsedSerialize = time.Since(startSerialize) // End timing serialization
	serializedData = buf.Bytes()
	fmt.Print("After Serialize:", string(serializedData))

	pprof.StopCPUProfile() // Stop CPU profiling
	fSer.Close()
}

func deserialize(buf *bytes.Buffer, elapsedDeserialize *time.Duration) {

	// Start CPU profiling for deserialization
	fDes, err := os.Create("cpu_deserialize.prof")
	if err != nil {
		log.Fatal("Could not create CPU profile for deserialization: ", err)
	}
	pprof.StartCPUProfile(fDes)

	startDeserialize := time.Now() // Start time deserialization

	decoder := json.NewDecoder(buf)
	var temp []Data
	err = DeSerialize(decoder, &temp)
	if err != nil {
		log.Fatal("Deserialization error: ", err)
		return
	}

	*elapsedDeserialize = time.Since(startDeserialize) // End timing deserialization

	fmt.Println("After DeSerialize:")
	for index, value := range temp {
		fmt.Println(index, value)
	}

	pprof.StopCPUProfile() // Stop CPU profiling
	fDes.Close()
}

func main() {

	var DataRecords []Data

	// Create sample data
	var i int
	var t Data
	for i = 0; i < 10; i++ {
		t = Data{
			Key: getString(5),
			Val: random(1, 100),
		}
		DataRecords = append(DataRecords, t)
	}

	var elapsedSerialize, elapsedDeserialize time.Duration

	// bytes.Buffer is both an io.Reader and io.Writer
	buf := new(bytes.Buffer)

	// Serialization with Memory Profiling
	serialize(buf, DataRecords, &elapsedSerialize)
	writeMemProfile("memory_profile_serialize.prof")

	// Reset buffer for Deserialization
	buf.Reset()
	buf.Write(serializedData)

	// Deserialization with Memory Profiling
	deserialize(buf, &elapsedDeserialize)
	writeMemProfile("memory_profile_deserialize.prof")

	fmt.Printf("Serialization took %s \n", elapsedSerialize)
	fmt.Printf("Deserialization took %s \n", elapsedDeserialize)
}

// Write memory profile
func writeMemProfile(filename string) {
	f, err := os.Create(filename)
	if err != nil {
		log.Fatal("Could not create memory profile: ", err)
	}
	defer f.Close()

	runtime.GC()
	if err := pprof.WriteHeapProfile(f); err != nil {
		log.Fatal("Could not write memory profile: ", err)
	}
}

```

{% hint style="info" %}
Decoder와 Encoder의 중복 메모리 할당을 피하기 위해서 매개변수로 포인터 값을 받고 Decoder와 Encoder는 메인 함수에서 1회만 할당하여 사용하는 것이 좋음으로 위와 같은 구조로 함수를 구성한다.



여기서 `DeSerialize` 함수나 `Serialize` 함수의 매개변수가 interface{} 이기 때문에 어떠한 JSON 레코드도 처리 가능하다.
{% endhint %}

### JSON 레코드 출력 다듬기

기본으로  구조체 또는 JSON 레코드를 출력하면 가독성이 매우 떨어집니다.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>가독성이 많이 떨어져 보이는 JSON 레코드 출력형태</p></figcaption></figure>

이를 해결하기 위해서는 `PrettyPrint()` 와 `JSONstream()`이라는 두 가지 방식의 함수를 실제로 구현하면서 해결 방안을 알아보도록 하겠습니다.

#### PrettyPrint() 함수 예제

```go
func PrettyPrint(v interface{}) (err error) {
    b, err := json.MarshalIndent(v, "", "\t")
    
    if err == nil {
        fmt.Println(string(b))
    }
    
    return err
}
```

위 예시에서 볼 수 있듯이 Indent를 넣기 때문에 훨씬 좋은 가독성의 JSON 레코드를 생성하게 됩니다. JSON 데이터의 스트림을 다듬어서 출력하려면 아래 JSONstream() 함수를 이용하면 됩니다.

#### JSONstream() 함수 예제

```go
func JSONstream(data interface{}) (string, error) {
    buffer := new(bytes.Buffer)
    encoder := json.NewEncoder(buffer)
    encoder.SetIndent("", "\t")
    
    err := encoder.Encode(data)    // json encoder를 통해 indent 삽입
    
    if err != nil {
        return "", err
    }
    
    return buffer.String(), nil
}
```

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption><p>예쁜 JSON 레코드출력 화면의 모습</p></figcaption></figure>
