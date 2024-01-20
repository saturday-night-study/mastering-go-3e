# 두 가지 CSV  파일 포맷 다루기



## 개요



### 학습 목표

* **정의한 구조체(A)**에 **인터페이스(I)를 구현**  하여 I를 사용하는 함수로  원하는 결과를 얻을 수 있는 프로그램을 작성한다.



### 프로그램 정의

1. CLI 프로그램은 CSV 파일의 위치를 인수(argument) 값으로 전달한다.
2. CSV 파일은 두가지 포멧을 가진다.
   1. 포멧 1 : 이름, 성 전화번호, 마지막 접근 시간
   2. 포멧 2 : 이름, 성, 지역코드, 전화번호, 마지막 접근 시간
3. CSV 파일을 읽은  후, 이름 순으로 정렬후 출력 한다.

### 시퀸스 다이어그램

<figure><img src="../.gitbook/assets/image (2).png" alt="" width="312"><figcaption></figcaption></figure>



## 핵심 소스코드



```go
package sort

import "math/bits"

// An implementation of Interface can be sorted by the routines in this package.
// The methods refer to elements of the underlying collection by integer index.
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int

    // Less reports whether the element with index i
    // must sort before the element with index j.
    //
    // If both Less(i, j) and Less(j, i) are false,
    // then the elements at index i and j are considered equal.
    // Sort may place equal elements in any order in the final result,
    // while Stable preserves the original input order of equal elements.
    //
    // Less must describe a transitive ordering:
    //  - if both Less(i, j) and Less(j, k) are true, then Less(i, k) must be true as well.
    //  - if both Less(i, j) and Less(j, k) are false, then Less(i, k) must be false as well.
    //
    // Note that floating-point comparison (the < operator on float32 or float64 values)
    // is not a transitive ordering when not-a-number (NaN) values are involved.
    // See Float64Slice.Less for a correct implementation for floating-point values.
    Less(i, j int) bool

    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

```go
package reader

import "strings"

type Figure01 struct {
	Name       string
	Surname    string
	Tel        string
	LastAccess string
}

type Figure01s []Figure01

// 구현된 인터페이스
func (f Figure01s) Len() int {
	return len(f)
}

func (f Figure01s) Less(i, j int) bool {
	return strings.Compare(f[i].Name, f[j].Name) < 0
}

func (f Figure01s) Swap(i, j int) {
	f[i], f[j] = f[j], f[i]
}
```

* sort 패키지의 Interface interface 를 구현합니다.

```go
// Sort sorts data in ascending order as determined by the Less method.
// It makes one call to data.Len to determine n and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
//
// Note: in many situations, the newer slices.SortFunc function is more
// ergonomic and runs faster.
func Sort(data Interface) {
    n := data.Len()
    if n <= 1 {
       return
    }
    limit := bits.Len(uint(n))
    pdqsort(data, 0, n, limit)
}
```

```go
switch figure.(type) {
case reader.Figure01s:
    sort.Sort(figure.(reader.Figure01s))
case reader.Figure02s:
    sort.Sort(figure.(reader.Figure02s))
}
```

* sort 패키지의 Sort 메서드를 실행

## 팁

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* intellij 에서 ctrl + enter 시 위의 사진과 같이 Implement interface 를 이용하여, 인터페이스를 구현할  수 있습니다.

