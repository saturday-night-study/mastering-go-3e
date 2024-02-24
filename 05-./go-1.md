# 깃허브에 Go 패키지 저장

Go 언어는 오픈소스에 특화되어 있다고 할만큼 구성이 잘 되어있습니다. 그렇기에 **GitHub에 등록 된  Go 패키지를 다루는 방법에 대해서 알아**보고 직접 **패키지를 저장, 설계, 구현, 테스트 해보며 패키지를 다루는 방법에 대해**서 알아보겠습니다.

## GitHub 저장소 패키지 다루기

먼저 `GitHub`는 `Git`이라고 불리는 프로그램을 웹 어플리케이션 환경에서 조금 더 사용하기 쉽도록 만들어진 Hub 서비스입니다. 여러 사람들이 각자의 저장소를 만들고 이를 공유할 수 있으며, 개인 저장소를 생성해 자신만의 패키지를 다룰 수도 있습니다.

```bash
$ git clone https://github.com/mactsouk/post05.git
```

위 명령어를 실행하면 해당 페이지에 기록 된 것처럼 post05에 대한 소스코드를 직접 받을 수 있게 됩니다.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>위 github 페이지 실제 접속 화면</p></figcaption></figure>

소스를 가져오는 것 대신에 `go get` 명령어를 통해서 개발하고 있는 자신의 패키지에 직접 소스를 추가하는 것도 쉽게 진행할 수 있습니다. (아래에서 추가 기술)

## 데이터베이스를 다루는 Go 패키지

해당 파트에서는 PostgreSQL 데이터베이스를 다루는 Go 패키지를 만들어보며 패키지를 개발, 저장, 사용하는 방법에 대해 실습을 진행합니다.

보통 어플리케이션에서는 특정 `스키마(Schema)`와 `테이블(Table)`에 접근할 때는 **관련 함수를 별도의 패키지(라이브러리)로 만들어 사용**합니다. (NoSQL도 동일)

Go는 [데이터베이스를 다루는 범용 패키지](https://golang.org/pkg/database/sql/) 제공합니다. 그러나 실제 각 데이터베이스에 연결하고 사용할 때에는 `드라이버(Driver)` 역할을 담당할 패키지가 필요합니다.

### Go 패키지를 사용하는 이유

Go 패키지를 사용하는 이유에 대해서는 [go.md](go.md "mention")에서도 기술하였지만 조금 더 자세히 이야기하자면, 아래 목록과 같습니다.

* Go 패키지는 어플리케이션을 개발하는 팀원에게 **자유롭게 공유**할 수 있음
* Go 패키지를 통해 **문서화**를 진행할 수 있음
* Go 패키지로 제공하는 특수한 함수가 **요구 사항을 쉽게 충족**할 수 있음
* **특정 패키지의 기능을 모두 사용할 필요 없이** 해당 패키지에서 제공되는 함수와 기능을 가져와 **새로운 패키지를 구성**할 수 있음
* 특정 패키지에서 다루는 내부 기능이 변경되어도 Go 패키지의 인터페이스(또는 함수)가 동일하다면 내부 기능에 대해서는 몰라도 됨 (일종의 캡슐화 🤔)&#x20;

### 데이터베이스의 이해

`PostgreSQL`, `MySQL`, `MongoDB`와 같은 데이터베이스 서버와 통신하려면 추가 패키지가 필요합니다. 현재 파트에서는 `PostgreSQL`을 사용하고 있으니 `PostgreSQL`에 알맞는 패키지가 필요합니다.

`PostgreSQL`은 `sqlx`, `pg`, `pgx` 등의 일반적인 패키지도 있고 `GORM`과 같은 `ORM(Object Relational Mapping)` 패키지도 존재합니다. [(2023년 ORM 패키지 비교)](https://www.libhunt.com/l/go/topic/orm)

당 파트에서는 `PostgreSQL`를 사용하기 위해 `pq` 패키지를 추가합니다.

```go
$ go get github.com/lib/pq
```

`PostgreSQL` 서버는 아래 `docker-compose.yml`을 사용해 `도커(Docker)` 이미지로 실행하여 구성하면 됩니다.&#x20;

(해당 yml 파일 작성 후 `docker-compose up` 명령으로 실행)

```yaml
version: '3'

services:
  postgres:
    image: postgres
    container_name: postgres
    environment:
      - POSTGRES_USER=karriz          # 원하는 사용자 이름
      - POSTGRES_PASSWORD=pass1234    # 원하는 사용자 비밀번호
      - POSTGRES_DB=master            # 원하는 DB 이름
    volumes:
      - ./postgres:/var/lib/postgresql/data/
    networks:
      - psql
    ports:
      - "5432:5432"
      
volumes:
  postgres:
  
networks:
  psql:
    driver: bridge
```

이후 getSchema.go 파일을 작성해 PostgreSQL의 스키마를 확인할 수 있습니다.

{% hint style="info" %}
`PostgreSQL`에서의 `스키마(Schema)`는 `테이블(Table)`, `뷰(View)`, `인덱스(Index)`를 포함하는 `네임스페이스(Namespace)`를 말하며, 새 데이터베이스가 생성될 때 `public`이라는 스키마를 자동으로 생성하여 위 3가지 정보를 관리한다.
{% endhint %}

```go
// getSchema.go
package main

import (
    "database/sql"
    "fmt"
    "os"
    "strconv"
    
    _ "github.com/lib/pq"
)

func main() {
    args := os.Args
    if len(args) != 6 {
        fmt.Println("getSchema.go needed 6 arguments")
    }
    
    host := args[1]
    port := args[2]
    user := args[3]
    password := args[4]
    database := args[5]
    
    port, err := strconv.Atoi(p)
    
    if err != nil {
         fmt.Println("Not a valid port number: ", err)
         return 
    }
    
    conn := fmt.Sprintf("host=%s port=%s user=%s password=%s dbname=%s sslmode=disable", host, port, user, password, database)
    
    dbInstance, err := sql.Open("postgres", conn)
    
    if err != nil {
         fmt.Println("DB Open() Error: ", err)
         return 
    }
    
    defer dbInstance.Close()
    
    rows, err := dbInstance.Query(`SELECT "datname" FROM "pg_database" WHERE datistemplate = false`)
    
    if err != nil {
         fmt.Println("DB Qeury() Error: ", err)
         return 
    }
    
    for rows.Next() {
         var name string
         err = rows.Scan(&name)
         if err != nil {
              fmt.Println("DB Qeury() Error: ", err)
              return 
         }
         fmt.Println("*", name)
    }
    
    defer rows.Close()
    
    query := `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' ORDER BY table_name`
    
    rows, err = dbInstance.Query(query)
    
    if err != nil {
         fmt.Println("DB Qeury() Error: ", err)
         return 
    }
    
    for rows.Next() {
         var name string
         err = rows.Scan(&name)
         if err != nil {
              fmt.Println("DB Qeury() Error: ", err)
              return 
         }
         fmt.Println("+T", name)
    }
    
    defer rows.Close()
}
```

```go
// 실행명령어
$ go run getSchema.go localhost 5432 user password go

// 실행 결과
* postgres
* master
* go
+T userdata
+T users
```

### Go 패키지 저장

패키지를 저장할 때에는 Git을 통해서 저장소에 저장을 하면됩니다. GitHub, GitLab 등과 같은 Hub 서비스를 통해도 좋고 자신만의 Git 원격지  서버를 구축해서 저장해도 좋습니다.

본 파트에서는 앞서 사용했던 post05 패키지를 그대로 사용하게 됩니다.

{% hint style="info" %}
GitHub 저장소를 생성할 때에는 `개인(Private)`, `공공(Public)` 2가지 유형으로 생성할 수 있는데, **Go 패키지의 쉬운 공유를 위해서 공공 저장소를 생성**하는 것이 바람직하다.
{% endhint %}

### Go 패키지 설계

해당 `post05` 패키지에서는 `users`테이블과 `userdata`테이블 2가지를 사용할 예정입니다. 하지만 이미 `users`와 `userdata`테이블이 기존에 생성되어 있기에 SQL 파일을 작성할 때 이점을 생각해야 됩니다.

<figure><img src="../.gitbook/assets/image (10).png" alt="" width="312"><figcaption><p>2가지 테이블의 구조</p></figcaption></figure>

위 구조를 생각하면서 아래 `create_tables.sql` 소스파일을 확인하며, 전체 테이블 구성에 대해서 확인해보겠습니다.

```sql
DROP DATABASE IF EXISTS go;
CREATE DATABASE go;

DROP TABLE IF EXISTS Users;
DROP TABLE IF EXISTS Userdata;

\c go;

CREATE TABLE Users (
    ID SERIAL,
    Username VARCHAR(100) PRIMARY KEY
);

CREATE TABLE Userdata (
    UserID Int NOT NULL,
    Name VARCHAR(100),
    Surname VARCHAR(100),
    Description VARCHAR(200)
);
```

위와 같은 형태로 SQL 파일을 작성해 `psql` 명령어를 사용하면 데이터베이스에 직접 명령을 실행하여 기존 데이터베이스 및 테이블 삭제 새로운 테이블의 생성 과정을 거칠 수 있게 됩니다.

```bash
$ psql -h localhost -p 5432 -U user master < create_tables.sql
```

이제 실행할 수 있는 구조를 갖췄기에 패키지의 기능을 다뤄볼 시간입니다. 해당 Go 패키지의 기능은 다음목록과 같이 구현될 예정입니다.

* 새 사용자 생성
* 기존사용자 삭제
* 기존 사용자 수정
* 현재 생성 된 모든 사용자 목록 조회

### Go 패키지 구현

위 기능을 구현하기 위해서는 다음 목록과 같은 함수(또는 메서드)가 필요합니다.

1. PostgreSQL과의 연결을 초기화하는 함수
2. 연결하기 위한 세부 정보의 기본 값
3. 주어진 사용자 이름이 존재하는지 확인하는 함수
4. 새로운 사용자를 DB에추가하는 함수
5. 존재하는 사용자를 DB에서 삭제하는 함수
6. 존재하는 사용자의 정보를 수정하는 함수
7. 존재하는 모든 사용자들의 목록을 조회하는 함수

```go
package post05

import (
	"database/sql"
	"errors"
	"fmt"
	"strings"

	_ "github.com/lib/pq"
)

// Connection details
var (
	Hostname = ""
	Port     = 2345
	Username = ""
	Password = ""
	Database = ""
)

// Userdata is for holding full user data
// Userdata table + Username
type Userdata struct {
	ID          int
	Username    string
	Name        string
	Surname     string
	Description string
}

func openConnection() (*sql.DB, error) {
	// connection string
	conn := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		Hostname, Port, Username, Password, Database)

	// open database
	db, err := sql.Open("postgres", conn)
	if err != nil {
		return nil, err
	}
	return db, nil
}

// The function returns the User ID of the username
// -1 if the user does not exist
func exists(username string) int {
	username = strings.ToLower(username)

	db, err := openConnection()
	if err != nil {
		fmt.Println(err)
		return -1
	}
	defer db.Close()

	userID := -1
	statement := fmt.Sprintf(`SELECT "id" FROM "users" where username = '%s'`, username)
	rows, err := db.Query(statement)

	for rows.Next() {
		var id int
		err = rows.Scan(&id)
		if err != nil {
			fmt.Println("Scan", err)
			return -1
		}
		userID = id
	}
	defer rows.Close()
	return userID
}

// AddUser adds a new user to the database
// Returns new User ID
// -1 if there was an error
func AddUser(d Userdata) int {
	d.Username = strings.ToLower(d.Username)

	db, err := openConnection()
	if err != nil {
		fmt.Println(err)
		return -1
	}
	defer db.Close()

	userID := exists(d.Username)
	if userID != -1 {
		fmt.Println("User already exists:", Username)
		return -1
	}

	insertStatement := `insert into "users" ("username") values ($1)`
	_, err = db.Exec(insertStatement, d.Username)
	if err != nil {
		fmt.Println(err)
		return -1
	}

	userID = exists(d.Username)
	if userID == -1 {
		return userID
	}

	insertStatement = `insert into "userdata" ("userid", "name", "surname", "description")
	values ($1, $2, $3, $4)`
	_, err = db.Exec(insertStatement, userID, d.Name, d.Surname, d.Description)
	if err != nil {
		fmt.Println("db.Exec()", err)
		return -1
	}

	return userID
}

// DeleteUser deletes an existing user
func DeleteUser(id int) error {
	db, err := openConnection()
	if err != nil {
		return err
	}
	defer db.Close()

	// Does the ID exist?
	statement := fmt.Sprintf(`SELECT "username" FROM "users" where id = %d`, id)
	rows, err := db.Query(statement)

	var username string
	for rows.Next() {
		err = rows.Scan(&username)
		if err != nil {
			return err
		}
	}
	defer rows.Close()

	if exists(username) != id {
		return fmt.Errorf("User with ID %d does not exist", id)
	}

	// Delete from Userdata
	deleteStatement := `delete from "userdata" where userid=$1`
	_, err = db.Exec(deleteStatement, id)
	if err != nil {
		return err
	}

	// Delete from Users
	deleteStatement = `delete from "users" where id=$1`
	_, err = db.Exec(deleteStatement, id)
	if err != nil {
		return err
	}

	return nil
}

// ListUsers lists all users in the database
func ListUsers() ([]Userdata, error) {
	Data := []Userdata{}
	db, err := openConnection()
	if err != nil {
		return Data, err
	}
	defer db.Close()

	rows, err := db.Query(`SELECT "id","username","name","surname","description"
		FROM "users","userdata"
		WHERE users.id = userdata.userid`)
	if err != nil {
		return Data, err
	}

	for rows.Next() {
		var id int
		var username string
		var name string
		var surname string
		var description string
		err = rows.Scan(&id, &username, &name, &surname, &description)
		temp := Userdata{ID: id, Username: username, Name: name, Surname: surname, Description: description}
		Data = append(Data, temp)
		if err != nil {
			return Data, err
		}
	}
	defer rows.Close()
	return Data, nil
}

// UpdateUser is for updating an existing user
func UpdateUser(d Userdata) error {
	db, err := openConnection()
	if err != nil {
		return err
	}
	defer db.Close()

	userID := exists(d.Username)
	if userID == -1 {
		return errors.New("User does not exist")
	}
	d.ID = userID
	updateStatement := `update "userdata" set "name"=$1, "surname"=$2, "description"=$3 where "userid"=$4`
	_, err = db.Exec(updateStatement, d.Name, d.Surname, d.Description, d.ID)
	if err != nil {
		return err
	}

	return nil
}
```

### Go 패키지 테스트

패키지를 테스트하기 위해서는 다양한 조건을 만들 수 있고, 해당 조건에 대한 예상 결과 값의 일치 여부 등을 확인하는 기능이 필요합니다.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
    
    "github.com/mactsouk/post05"
)

var MIN = 0
var MAX = 26

func random(min, max int) int {
    return rand.Intn(max-min) + min
}

func getString(length int64) string {
    startChar:= "A"
    temp := ""
    var i int64 = 1
    for {
        myRand := random(MIN, MAX)
        newChar := string(startChar[0] + byte(myRand))
        temp = temp + newChar
        if i == length { 
            break
        }
        i++
    }
    return temp
}

func main() {
    post05.Hostname = "localhost"
    post05.Port = 5432
    post05.Username = "user"
    post05.Password = "password"
    post05.Database = "go"
    
    data, err := post05.ListUsers()
    
    if err != nil {
        fmt.Println(err)
        return 
    }
    
    // 현재 존재하는 사용자 목록 조회
    for _, v := range data {
        fmt.Println(v)
    }
    
    SEED := time.Now().Unix()
    
    rand.Seed(SEED)
    
    // 랜덤 사용자 5명 생성
    random_username := getString(5)
    
    t := post05.Userdata {
        Username:        random_username,
        Name:            "동현",
        Surname:         "이",
        Description:     "안녕하세요 블록체인개발자 이동현입니다.",
    }
 
     // 새로 생성한 사용자를 추가함 
     id := post05.AddUser(t)
     
     if id == -1 {
         fmt.Println("error occured :", t.Username)
     }
     
     // 해당 id의 삭제 시도
     err = post05.DeleteUser(id)
     
     if err != nil {
         fmt.Println(err)
     }
     
     // 1번더 동일한 id의 사용자 삭제 시도, 실패해야 정상임
     err = post05.Deleteuser(id)
     
     if err != nil {
         fmt.Println(err)
     }
}
```
