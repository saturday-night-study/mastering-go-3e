# 전화번호부 애플리케이션 개선



## 개요



맵과 구조체에 대한 이해가 이전챕터 들을 보면서  충분해 졌다면, 전화번호부 애플리케이션을 한번 제작해 보겠습니다.&#x20;



### 링크 및 라이브러리

* [실습 예제 ](https://github.com/MarkYoo23/white-page)(github)
* [ent.go](https://entgo.io/) : ORM
* [dig](https://github.com/uber-go/dig) : defendency - injection



## 개선 목표



03장 합성 데이터 타입에서는  맵, 구조체, 정규표현식을 주요 기능으로 활용하여 애플리케이션을 작성 하는 것을 목표로 두었는데요. 그 외에도 조금 더 챌린지 할 수 있는 부분을 저는 핵심 가치로 두고 금주 스터디를 진행 하였습니다.



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







