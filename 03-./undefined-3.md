# 전화번호부 애플리케이션 개선



## 개요



맵과 구조체에 대한 이해가 이전챕터 들을 보면서  충분해 졌다면, 전화번호부 애플리케이션을 한번 제작해 보겠습니다.&#x20;



### 링크 및 라이브러리

* [실습 예제 ](https://github.com/MarkYoo23/white-page)(github)
* [ent.go](https://entgo.io/) : ORM
* [dig](https://github.com/uber-go/dig) : defendency - injection



## 개선 목표



03장 합성 데이터 타입에서는  맵, 구조체, 정규표현식을 주요 기능으로활용하여 애플리케이션을 작성 하는 것을 목표로 두었는데요. 그 외에도 조금 더 챌린지 할 수 있는 부분이 있나 보면서 아래와 같은 시도를 해 보았습니다.



* CMD 명령을 지원한다.
* DI를 지원한다. (struct 활용)
* **CRUD를 모두 지원**하고, 이 데이터를 DB 에 저장할 수 있다. (struct 활용)
* CSV Import / Export 기능을 지원한다.
* 각 항목을  마지막으로  방문한 조회 시간, 생성 시간이 필드로 존재한다.
* **Map을 이용해  DB 인덱스를 구현** 할 수 있다.
* 전화 번호의 유효성을 검증할 수 있도록 **정규 표현식을 사용**한다.



## 프로그램



### CMD 지원





{% @github-files/github-code-block %}

```
func init() {
        execMap = make(map[string]exec.Exec)
        execMap["search"] = exec.NewSearchExec()
        execMap["list"] = exec.NewListExec()
        execMap["insert"] = exec.NewInsertExec()
        execMap["delete"] = exec.NewDeleteExec()
        execMap["export"] = exec.NewExportExec()
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


}// Some code
```





