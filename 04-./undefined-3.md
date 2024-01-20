# 전화번호부 애플리케이션 업데이트



## 신규 기능 추가

* 환경 변수 사용하여 CSV 파일 경로 수정
* list 커맨드는 '성' 필드를 기반으로 결과를 정렬

### 학습 목표

* Ertry 구조체에 sort 인터페이스를 적용하여, 성, 이름 순으로 정렬된 데이터를 받는다.



## 개발



### 환경 변수 설정



우선, csv 파일의 위치를 설정하기 위해서 환경 변수를 이용하여 보겠습니다. 아래의 코드는 환경 변수를 조회 하여 csv 파일의 위치를 설정하는 코드 입니다.



```go
package envs

import "os"

var (
    PhoneBookCsvLocation string = ".bin/phonebook.csv"
)

func init() {
    // 환경 변수가 없다면, 기본값으로 설정
    phonebookPath := os.Getenv("PHONEBOOK_CSV_PATH")
    if phonebookPath != "" {
       PhoneBookCsvLocation = phonebookPath
    }
}
```

환경 변수는 아주 간단하게 설정을 불러 올 수 있지만, Go 에서는 어떻게 환경  변수를 불러 올 수 있는지 조금 더 알아 보겠습니다.



```go
// env.go
// Getenv retrieves the value of the environment variable named by the key.
// It returns the value, which will be empty if the variable is not present.
// To distinguish between an empty value and an unset value, use LookupEnv.
func Getenv(key string) string {
	testlog.Getenv(key)
	v, _ := syscall.Getenv(key)
	return v
}


// package syscall / evn_windows
func Getenv(key string) (value string, found bool) {
	keyp, err := UTF16PtrFromString(key)
	if err != nil {
		return "", false
	}
	n := uint32(100)
	for {
		b := make([]uint16, n)
		n, err = GetEnvironmentVariable(keyp, &b[0], uint32(len(b)))
		if n == 0 && err == ERROR_ENVVAR_NOT_FOUND {
			return "", false
		}
		if n <= uint32(len(b)) {
			return UTF16ToString(b[:n]), true
		}
	}
}

func GetEnvironmentVariable(name *uint16, buffer *uint16, size uint32) (n uint32, err error) {
	r0, _, e1 := Syscall(procGetEnvironmentVariableW.Addr(), 3, uintptr(unsafe.Pointer(name)), uintptr(unsafe.Pointer(buffer)), uintptr(size))
	n = uint32(r0)
	if n == 0 {
		err = errnoErr(e1)
	}
	return
}

// dll_windows.go
func Syscall(trap, nargs, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)

```



동작 시스템이 windows 라 그런지, 윈도우 dll 에 환경 변수를 가져오라고 요청하고 있는데요

위의 코드를 요약하자면, 환경 변수의 이름을 특정 포인터 주소로 변환 후, 값을 불러오는 것을 알 수 있습니다.



### 정렬



전화번호부를 정렬하여 불러오기 위해서 아래와 같은 순서로 코드를 작성합니다.



&#x20;sort.Interface를 구현니다.

```go
package entries

import "white-page/ent"

type Entry struct {
	ent.Entry
}

func NewEntry(entry ent.Entry) *Entry {
	return &Entry{Entry: entry}
}

type Entries []*Entry

// sort.Interface
func (e Entries) Len() int {
	return len(e)
}

func (e Entries) Less(i, j int) bool {
	EntryI := e[i]
	EntryJ := e[j]

	// 성이 같으면 이름을 비교한다.
	if EntryI.Surname == EntryJ.Surname {
		return EntryI.Name < EntryJ.Name
	}
	
	// 다른 성이면 성을 비교한다.
	return EntryI.Surname < EntryJ.Surname
}

func (e Entries) Swap(i, j int) {
	e[i], e[j] = e[j], e[i]
}
```



데이터베이스에서 값을 불러와, Sort 함수를 이용해 정렬합니다.



```go
package entries

import "sort"

type EntryService struct {
    entryRepository *EntryRepository
}

func NewEntryService(entryRepository *EntryRepository) *EntryService {
    return &EntryService{entryRepository}
}

func (s *EntryService) GetSortByFullName() (Entries, error) {
    entities, err := s.entryRepository.GetAll()
    if err != nil {
       return nil, err
    }

    var entries Entries

    for _, entity := range entities {
       entry := NewEntry(*entity)
       entries = append(entries, entry)
    }

    sort.Sort(entries)

    return entries, nil
}
```



그리고 CLI 인터페이스에서 사용할 수 있게, 동작을 등록합니다.

```go
package commands

import (
	"fmt"
	"github.com/spf13/cobra"
	"white-page/internal/di"
	"white-page/internal/entries"
)

func init() {
	rootCmd.AddCommand(&cobra.Command{
		Use: commandSortList,
		Run: func(cmd *cobra.Command, args []string) {
			service, err := di.GetService[entries.EntryService]()
			if err != nil {
				panic(fmt.Errorf("error : %v", err))
			}

			results, err := service.GetSortByFullName()
			if err != nil {
				panic(fmt.Errorf("not found error : %v", err))
			}

			for _, result := range results {
				fmt.Println(fmt.Sprintf("entry : %v, %v, %v", result.Name, result.Surname, result.Tel))
			}
		}})
}

```



마지막으로, CLI 인터페이스를 실행 해 보겠습니다.



phonebook.csv 에 아래의 무작위 데이터를 삽입합니다.

{% code title="phonebook.csv" %}
```
Isabella,Miller,01020000014
Ava,Smith,01020000005
Ella,Smith,01020000016
Liam,Johnson,01020000004
Amelia,Smith,01020000009
Oliver,Williams,01020000011
Ethan,Smith,01020000002
Henry,Brown,01020000017
Benjamin,Smith,01020000015
Charlotte,Davis,01020000012
Bob,Smith,01022221111
Alice,Smith,01022224444
James,Williams,01020000010
Scarlett,Smith,01020000018
Noah,Smith,01020000006
Olivia,Johnson,01020000003
William,Davis,01020000013
Sophia,Williams,01020000007
Mason,Johnson,01020000008
Emma,Smith,01020000001
```
{% endcode %}



intellij 에서  아래와 같이 import /  list 조회기능을 설정합니다.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



원본 소스와 다르게, 아래와 같이 정렬된 결과를 얻을 수 있습니다.

```go
  █████████  ██████████   ██████   █████
 ███░░░░░███░░███░░░░███ ░░██████ ░░███ 
░███    ░░░  ░███   ░░███ ░███░███ ░███ 
░░█████████  ░███    ░███ ░███░░███░███ 
 ░░░░░░░░███ ░███    ░███ ░███ ░░██████ 
 ███    ░███ ░███    ███  ░███  ░░█████ 
░░█████████  ██████████   █████  ░░█████
 ░░░░░░░░░  ░░░░░░░░░░   ░░░░░    ░░░░░
-------------------------------------------------------------------------------
Write page application v{ApplicationVersion}, created by 'Saturday Day Night'
-------------------------------------------------------------------------------
args : [sortlist]

entry : Henry, Brown, 01020000017
entry : Charlotte, Davis, 01020000012
entry : William, Davis, 01020000013
entry : Liam, Johnson, 01020000004
entry : Mason, Johnson, 01020000008
entry : Olivia, Johnson, 01020000003
entry : Isabella, Miller, 01020000014
entry : Alice, Smith, 01022224444
entry : Amelia, Smith, 01020000009
entry : Ava, Smith, 01020000005
entry : Benjamin, Smith, 01020000015
entry : Bob, Smith, 01022221111
entry : Ella, Smith, 01020000016
entry : Emma, Smith, 01020000001
entry : Ethan, Smith, 01020000002
entry : Noah, Smith, 01020000006
entry : Scarlett, Smith, 01020000018
entry : James, Williams, 01020000010
entry : Oliver, Williams, 01020000011
entry : Sophia, Williams, 01020000007

```

