# Go 1.16의 새로운 기능

## 파일 임베드 (embed)



### 파일 임베딩과의 차이

파일 + 임베딩의 합성어로 추측됩니다.

* 파일 : 시스템에서 데이터를 저장하는데 사용되는 단위
* 임베딩 : 사람이 쓰는 자연어를 기계가 이해할 수 있는 벡터로 바꾸는 과정.

\*\* 벡터 : 크기와 방향을 가지는 수학적 정량. 즉 임베딩이란 자연어를 수학적인 값으로 변환한 다음, 이를 저장하는 것을 뜻한다.

또한리눅스 시스템은 주어진 파일의 내용, 크기, 권한, 생성 일자, 수정 일자, 소유자 정보 등과 같은 다양한 메타 데이터를 수집하여 이를 벡터화 합니다. 이를 통해 리눅스 파일 시스템은 파일을 단순 저장이 아닌 검색, 파일 분류, 그룹화 등을 지원하는 시스템으로 기능할 수 있습니다. (chat-gpt)

이는 비유하자면, 사람이 물건을 판매하기 위해 표를 만들고 이를 통해 그룹화 하고 쉽게 인식하듯이. 파일 시스템 역시도 이를 해석하는 과정을 거친다는 뜻입니다.



### Go의 파일 임베드&#x20;

임의의 파일을 파일/폴더를 바이너리 안에 내장(포함)하는 방법입니다.

아래의 세가지 방법으로 사용 가능한데요

* 한개의 파일 임베딩
* 폴더를 임베딩
* 빌드 태그를 이용한 조건 임베딩

예제에서는 한개의 파일 임베딩을 이용하여 임베딩 합니다.



```

package main  
  
import (  
    // 임베딩 패키지 직접 이용하진 않으나, 사용하므로 _로 참조  
    _ "embed"  
    "fmt"    "os")  
  
// 경로에 해당하는 파일을 전역 변수에 포함 
//go:embed static/image.jpg  
var f1 []byte  
  
//go:embed static/textfile  
var f2 string  
  
func writeToFile(s []byte, path string) error {  
    fd, err := os.OpenFile(path, os.O_CREATE|os.O_WRONLY, 0644)  
    if err != nil {  
       return err  
    }  
    defer fd.Close()  
  
    n, err := fd.Write(s)  
    if err != nil {  
       return err  
    }  
    fmt.Printf("wrote %d bytes\n", n)  
    return nil  
}  
  
func main() {  
    arguments := os.Args  
    if len(arguments) == 1 {  
       fmt.Println("Print select 1|2")  
       return  
    }  
  
    fmt.Println("f1:", len(f1), "f2:", len(f2))  
  
    switch arguments[1] {  
    case "1":  
       filename := "/tmp/temporary.png"  
       err := writeToFile(f1, filename)  
       if err != nil {  
          fmt.Println(err)  
          return  
       }  
    case "2":  
       fmt.Print(f2)  
    default:  
       fmt.Println("Not a valid option!")  
    }  
}

```

1. 임베드 패키지를 \_ "embed" 로 import 합니다. 이를 통해서 임베드 기능을 사용할 수 있습니다.
2. //go:embed static/textfile 주석을 추가 하여 전역 변수에 임베드 패키지가 자동으로 파일을 읽을 수 있도록 추가합니다.
3. 그후 전역변수에 로드된 파일을 사용하면 됩니다! 설정 파일, 웹서버의 스태틱 파일등을 제공하는데 유용 하겠네요.

참조 자료

1. https://medium.com/bgpworks/golang-1-16%EC%97%90-%EC%83%88%EB%A1%9C-%EC%B6%94%EA%B0%80%EB%90%9C-%EA%B8%B0%EB%8A%A5-embed%EB%A1%9C-%EC%8A%A4%ED%83%9C%ED%8B%B1-%EB%A6%AC%EC%86%8C%EC%8A%A4-%EB%84%A3%EA%B8%B0-1675c4564f5e
2. https://blog.kesuskim.com/archives/go-embed/
3. https://pkg.go.dev/embed



## 폴더 관련 기능



### io/ioutil 패키지가 사라진 이유

(본 내용은 어느정도 뇌피셜이 섞여있습니다)

1.16에서는 io/ioutil 패키지가 지원 중단 되었는데요. 이것을 이야기 하기 위해선 Go 1 이전의 표준 라이브러리에 대한 내용을 알 필요성이 있습니다.

Go 1 이전에서는 논리적으로 프로그램은 OS에 요청하여 파일 기능과 io 기능을 사용한다라는 개념이 맞지 않았던것으로 보입니다.

&#x20;![](<../.gitbook/assets/Pasted image 20240330180254.png>)

(Go 1 이전의 표준 라이브러리 종속성)

논리적으로 프로그램은 OS에 요청하여 파일 기능과 io 기능을 사용한다라는 개념이니, os 기능과 os 기능에 종속된 io를 처리하도록 논리적 순서를 맞추고 라이브러리를 재 배치했는데요.



<div align="left">

<figure><img src="../.gitbook/assets/Pasted image 20240330180303.png" alt=""><figcaption></figcaption></figure>

</div>

(Go 1 이후의 표준 라이브러리 종속성)

이후 io/util에 있는 기능들을 재 배치할 필요성이 보였고, io/ioutil 패키지의 기능들이 각각 기능에 맞게 os 패키지와 io 패키지로 재 배치되었습니다.

참조

* https://github.com/golang/go/issues/40025
* https://github.com/golang/go/issues/42026
* https://go.dev/talks/2016/refactor.article



### 폴더를  os.ReadDir 로 읽고 os.DirEntry 구조체로 분석

```

package main  
  
import (  
    "fmt"  
    "os"    "path/filepath")  
  
func GetSize(path string) (int64, error) {  
    // 하위 폴더 DIR 및 파일 리스트를 을 조회한다.  
    // 이때 os.[]DirEntry 를 가져온다  
    contents, err := os.ReadDir(path)  
    if err != nil {  
       return -1, err  
    }  
  
    // 하우  
    var total int64  
    for _, entry := range contents {  
       // Visit directory entries  
       if entry.IsDir() {  
          // 폴더일 경우 재귀 호출하여 하위 폴더의 데이터를 계산해온다.  
          temp, err := GetSize(filepath.Join(path, entry.Name()))  
          if err != nil {  
             return -1, err  
          }  
          total += temp  
          // Get size of each non-directory entry  
       } else {  
          // 일반 파일일 경우 파일 값을 계산한다.  
          info, err := entry.Info()  
          if err != nil {  
             return -1, err  
          }  
          // Returns an int64 value  
          total += info.Size()  
       }  
    }  
    return total, nil  
}  
  
func main() {  
    arguments := os.Args  
    if len(arguments) == 1 {  
       fmt.Println("Need a <Directory>")  
       return  
    }  
  
    // 파일의 진짜 위치를 가져온다. (심볼릭 링크일경우)  
    root, err := filepath.EvalSymlinks(arguments[1])  
  
    // 파일 정보를 가져온다.  
    fileInfo, err := os.Stat(root)  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
  
    // 파일 정보로 디렉터리인지 확인한다.  
    fileInfo, _ = os.Lstat(root)  
    mode := fileInfo.Mode()  
    if !mode.IsDir() {  
       fmt.Println(root, "not a directory!")  
       return  
    }  
  
    // 디렉터리 전체 크기를 계산한다.  
    i, err := GetSize(root)  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
    fmt.Println("Total Size:", i)  
}
```

본 프로그램을 통해

1. os.ReadDir 함수를 이용하여 폴더를 읽어올 경우 os.DirEntry 구조체를 이용하여 폴더내의 트리 구조를 읽을 수 있는것을 확인하였습니다.



### io/fs 패키지

#### os 패키지와 포지션이 중복 되는데요?

파일 시스템과의 상호 작용을 위해 os 패키지가 사용되었는데, 이는 특정 운영 체제에 의존하도록 작성할 수 밖에 없습니다. (예시 파일 패스...). 이는 여러 운영 체제 간에 이식성과 호환성이 떨어지는 단점이 있었고, 이 패키지는 운영 체제와 독립적인 파일 시스템 인터페이스를 제공하여, Go 언어로 작성된 프로그램이 여러 운영 체제에서 일관된 방식으로 파일 시스템에 접근할 수 있도록 도와줍니다.

#### 임베딩과 io/fs 패키지 활용

임베딩을 이용해서 폴더를 위치를 가져오면, 이를 io/fs 패키지를 통해 쉽게 동일한 방식(파일 패스) 로 가져올 수 있게 합니다.

```

package main  
  
import (  
    "embed"  
    "fmt"    "io/fs"    "os")  
  
//go:embed static  
var f embed.FS  
  
var searchString string  
  
func walkFunction(path string, d fs.DirEntry, err error) error {  
    if err != nil {  
       return err  
    }  
    fmt.Printf("Path=%q, isDir=%v\n", path, d.IsDir())  
    return nil  
}  
  
func walkSearch(path string, d fs.DirEntry, err error) error {  
    if err != nil {  
       return err  
    }  
    if d.Name() == searchString {  
       fileInfo, err := fs.Stat(f, path)  
       if err != nil {  
          return err  
       }  
       fmt.Println("Found", path, "with size", fileInfo.Size())  
       return nil  
    }  
    return nil  
}  
  
func list(f embed.FS) error {  
    return fs.WalkDir(f, ".", walkFunction)  
}  
  
func search(f embed.FS) error {  
    return fs.WalkDir(f, ".", walkSearch)  
}  
  
func extract(f embed.FS, filepath string) ([]byte, error) {  
    return fs.ReadFile(f, filepath)  
}  
  
func writeToFile(s []byte, path string) error {  
    fd, err := os.OpenFile(path, os.O_CREATE|os.O_WRONLY, 0644)  
    if err != nil {  
       return err  
    }  
    defer fd.Close()  
  
    n, err := fd.Write(s)  
    if err != nil {  
       return err  
    }  
    fmt.Printf("wrote %d bytes\n", n)  
    return nil  
}  
  
func main() {  
    // At this point we do not know what is included in ./static  
  
    // 임베딩과 결합하여, 파일 리스트 호출이 가능합니다.  
    err := list(f)  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
  
    // 임베딩과 결합하여, 파일 검색을 파일 명으로 가능합니다.  
    searchString = "file.txt"  
    err = search(f)  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
  
    // 파일을 불러올 수 있습니다.  
    buffer, err := extract(f, "static/file.txt")  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
  
    // Save it to an actual file.txt  
    curPath, _ := os.Getwd()  
  
    // 실제로 os.File 을 사용하는 경우... windows/linux 기반 시스템에서 같이 사용하지 못함  
    err = writeToFile(buffer, curPath+"\\static\\IOFS.txt")  
    if err != nil {  
       fmt.Println(err)  
       return  
    }  
}

```

참고자료

* https://pkg.go.dev/io/fs
* https://github.com/golang/go/issues/41190

