# 전화번호부 애플리케이션 개선



## 소개에 앞서



&#x20;맵과 구조체에 대한 이해가 이전 챕터 들을 보면서  충분해 졌다면, 전화번호부 애플리케이션을 한번 제작해 보려 하는데요, 그 전에 몇 가지 개념과 라이브러리 들을 알아두시면 좋겠습니다.



### DDD Layied architecture



<figure><img src="../.gitbook/assets/image (1).png" alt="" width="300"><figcaption></figcaption></figure>

&#x20; 복잡한 비즈니스 로직을 처리하기 위해 레이어로 구역을 나누어 유지 보수성, 확장성을 향상 시킨 아키텍처 입니다. 앞으로 시스템이 많은 변경점을 가지기 때문에 코드를 각자의 위치에서 응집성 있게 작성하는것이 중요합니다.



### 의존성 주입(DI)

&#x20;

&#x20;Go 에서는 정적 분석([wire](https://github.com/google/wire)), 런타임 주입 ([dig](https://github.com/uber-go/dig)) 방식으로 의존성 주입을 사용하는데요 여기서는 런타임 주입을 사용하였습니다.&#x20;

&#x20;장단점으로  정적 분석의 경우 코드 생성이 필요하나, 오류 검출이 빌드시 가능하고, 런타임 주입시에는 코드 생성이 필요없으나 빌드시 오류를 확인할 수 없습니다.

&#x20; 의존성 주입을 사용한 이유는 순환 참조로 인해 코드 작성이 어려워지는 경우가 많은데 이를 의존성 주입으로 좀 더 깔끔하게 관리하기 쉽습니다. (다만순환  참조 위험성도 높아집니다\~)



### 기본   프로젝트 레이아웃



&#x20;[https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout) 에서는 Go 생태계의 공통된 프로젝트 레이아웃을 제공하는데요, 이를 통해 보편적으로 프로젝트에 폴더에 소스코드를 배치하고 있었는지 알 수 있게 됩니다.



### Ent.go



Go 용 ORM 입니다.  현 시점에서 가장 완성도 높은 ORM 인터페이스를 제공하고 있는데요. 빠른 프로젝트 구성을 위해 도입하게 되었습니다.



간단히 사용법을 보자면



```powershell
go run -mod=mod entgo.io/ent/cmd/ent new Entry
```

{project-path}/ent/schema 에 아래와 같이 파일이 생성됩니다.

```go
type Entry struct {
	ent.Schema
}

// Fields of the Entry.
func (Entry) Fields() []ent.Field {
	return nil
}

// Edges of the Entry.
func (Entry) Edges() []ent.Edge {
	return nil
}

```

이 파일을

```go
type Entry struct {
	ent.Schema

	Name     string
	Surname  string
	Tel      string
	CreateAt time.Time
}

// Fields of the Entry.
func (Entry) Fields() []ent.Field {
	return []ent.Field{
		field.String("name").Default("unknown"),
		field.String("surname").Default("unknown"),
		field.String("tel").Unique(),
		field.Time("created_at").Default(time.Now),
	}
}

// Edges of the Entry.
func (Entry) Edges() []ent.Edge {
	return nil
}

```

필드와, 필드 정의를 해 주고

```
 go generate ./ent
```

명령어를 실행하면

```go
func (r *EntryService) Add(name string, surName string, tel string) (*ent.Entry, error) {
	client := r.dbContext.Client

	result, err := client.Entry.Create().
		SetName(name).
		SetSurname(surName).
		SetTel(tel).
		Save(context.Background())

	if err != nil {
		return nil, err
	}

	return result, nil
}

```

ORM 을 사용할 수 있습니다. 다만 Entiry 객체를 직접 넣을순 없기때문에, 약간의 불편함은 있습니다



## 개선 목표



03장 합성 데이터 타입에서는 맵, 구조체, 정규표현식을 주요 기능으로 활용하여 애플리케이션을 작성 하는 것을 목표로 두었는데요.



* CMD 명령을 지원한다.
* DI를 지원한다. (struct 활용)
* **CRUD를 모두 지원**하고, 이 데이터를 DB 에 저장할 수 있다. (struct 활용)
* CSV Import / Export 기능을 지원한다.
* 각 항목을  마지막으로  방문한 조회 시간, 생성 시간이 필드로 존재한다.
* **Map을 이용해  DB 인덱스를 구현** 할 수 있다.
* 전화 번호의 유효성을 검증할 수 있도록 **정규 표현식을 사용**한다.



## 프로그램 핵심 동작



저는 이번 챕터에서 알고 가야할 것을

* 맵과 구조체를 활용하기&#x20;
* Regex 를 활용하기

로 정리하였는데요, 요 두가지가 실제 코드 상에서 가장 많이 쓰게 되는 부분 같아 별도로 적었습니다.



### Map 과  구조체 활용하여 인덱스 사용하기



```go
package main

import (
	"bufio"
	"os"
	"strings"
	"white-page/cmd/exec"
	"white-page/internal/db"
	"white-page/internal/di"
)

var execMap map[string]exec.Exec

func init() {
	execMap = make(map[string]exec.Exec)
	execMap["search"] = exec.NewSearchExec()
	execMap["list"] = exec.NewListExec()
	execMap["insert"] = exec.NewInsertExec()
	execMap["delete"] = exec.NewDeleteExec()
	execMap["export"] = exec.NewExportExec()
	execMap["import"] = exec.NewImportExec()
	execMap["map"] = exec.NewMapExec()
}

func main() {
	err := di.Initialize()
	if err != nil {
		panic(err)
	}

	dbContext, err := di.GetService[db.DbContext]()
	if err != nil {
		panic(err)
	}

	err = dbContext.Open()
	if err != nil {
		panic(err)
	}

	err = dbContext.GenerateSchema()
	if err != nil {
		panic(err)
	}

	reader := bufio.NewReader(os.Stdin)

	for {
		input, _ := reader.ReadString('\n')
		splitValues := strings.Split(strings.Replace(input, "\r\n", "", -1), " ")

		ex, ok := execMap[splitValues[0]]
		if ok {
			err = ex.Execute(splitValues[1:])
		} else {
			break
		}
	}

	err = dbContext.Close()
	if err != nil {
		panic(err)
	}
}

```



map\[string]struct 를 이용하여 (맵 & 구조체 를 활용) 명령을 분석하고 실행하는 부분을 담당하는데요 맵을 응용하여 switch case 문을 대신하는 형태가 되었습니다.

### Regex



```go

package exec

import (
	"errors"
	"fmt"
	"regexp"
	"strings"
	"white-page/internal/di"
	"white-page/internal/entries"
)

type InsertExec struct {
}

func NewInsertExec() *InsertExec {
	return &InsertExec{}
}

func (*InsertExec) Execute(args []string) error {
	service, err := di.GetService[entries.EntryService]()
	if err != nil {
		return fmt.Errorf("error : %v", err)
	}

	var name = args[0]
	var surname = args[1]
	var tel = args[2]

	match, err := regexp.Match(`^\d{3}-\d{3,4}-\d{4}$`, []byte(tel))
	if err != nil {
		return err
	}

	if !match {
		return errors.New("tel format error")
	}

	tel = strings.ReplaceAll(tel, "-", "")

	result, err := service.Add(name, surname, tel)
	if err != nil {
		return fmt.Errorf("db error : %v", err)
	}

	fmt.Println(fmt.Sprintf("entry : %v", result))

	return nil
}

```



Regex를 활용하는 핵심은

* Regex가 Presentaion 레이어에 위치하여 사용자의 입력을 체크하는 위치에 있게 하였습니다.&#x20;



## 참조



1. [실습 예제 ](https://github.com/MarkYoo23/white-page)(github)
2. [ent.go](https://entgo.io/) : ORM
3. [dig](https://github.com/uber-go/dig) : defendency - injection
4. [goland-standards](https://github.com/golang-standards/project-layout) : project-layout
5. [DDD 레이어드 아키테처에 대하여](https://velog.io/@wodyd202/DDD-%EB%A0%88%EC%9D%B4%EC%96%B4%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
6. [의존성 주입의 개념](https://velog.io/@sana/DI-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85Dependency-Injection-%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%B0%A9%EB%B2%95)



\*\* 다음에는 go cobra 를 이용하여 CLI 를 구성하기

\*\* 챕터에 있는 GO 의 기능을 활용해서 예제를 좀더 풍부하게 작성  & 설명해보기



