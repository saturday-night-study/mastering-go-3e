# 유닉스 파일 시스템에서 순환 참조 찾기

파일 시스템의 순환 참조 찾기

```
package main  
  
import (  
    "fmt"  
    "os"    "path/filepath")  
  
// https://github.com/mactsouk/mastering-Go-3rd/blob/main/ch06/FScycles.go  
  
var visited = map[string]int{}
  
func walkFunction(path string, info os.FileInfo, err error) error {  
    fileInfo, err := os.Stat(path)  
    if err != nil {  
       return err  
    }  
  
    fileInfo, _ = os.Lstat(path)  
    mode := fileInfo.Mode()  
  
    // 일반 디렉토리 일 경우에는 방문 찍고 에러가 없음.
    if mode.IsDir() {  
       abs, _ := filepath.Abs(path)  
       _, ok := visited[abs]  
       if ok {  
          fmt.Println("Found cycle:", abs)  
          return nil  
       }  
       visited[abs]++  
       return nil  
    }  

    // 심볼릭 링크 (다른 파일이나 디렉터리를 가리킴)
    if mode&os.ModeSymlink != 0 {  
       temp, err := os.Readlink(path)  
       if err != nil {  
          fmt.Println("os.Readlink():", err)  
          return err  
       }  
  
       newPath, err := filepath.EvalSymlinks(temp)  
       if err != nil {  
          return err  
       }  
  
       linkFileInfo, err := os.Stat(newPath)  
       if err != nil {  
          return err  
       }  
  
       linkMode := linkFileInfo.Mode()  
       if linkMode.IsDir() {  
          fmt.Println("Following...", path, "-->", newPath)  
          abs, _ := filepath.Abs(newPath)  
          _, ok := visited[abs]  
          if ok {  
             fmt.Println("Found cycle!", abs)  
             return nil  
          }  
          visited[abs]++  
  
          return filepath.Walk(newPath, walkFunction)  
       }  
    }  
    return nil  
}  
  
func main() {  
    arguments := os.Args  
    if len(arguments) == 1 {  
       fmt.Println("Not enough arguments!")  
       return  
    }  
  
    Path := arguments[1]
    // filepath.Walk 함수는 시작 경로부터 시작하여 모든 하위 파일 및 디렉터리를 순회하며 각 항목에 대해 콜백 함수를 호출
    err := filepath.Walk(Path, walkFunction)  
    if err != nil {  
       fmt.Println(err)  
    }  
  
    for k, v := range visited {  
       if v > 1 {  
          fmt.Println(k, v)  
       }  
    }  
}
```

프로그램 실행

```
PS C:\Users\prtra\OneDrive\문서\GitHub\mastering-go-example\2024-03-30 file-embed\cmd> go run main.go c:\  
Following... c:\Documents and Settings --> C:\Users
...
```

1. `walkFunction` 함수는 `filepath.Walk` 함수의 콜백으로 사용됩니다. `filepath.Walk` 함수는 시작 경로부터 시작하여 모든 하위 파일 및 디렉터리를 순회하며 각 항목에 대해 콜백 함수를 호출합니다. 그리고 이 콜백 함수에서는 `walkFunction`을 재귀적으로 호출하여 디렉터리 내의 모든 파일 및 디렉터리를 다시 순회합니다.
2. 심볼릭 링크가 있는 경우에도 `walkFunction`은 해당 심볼릭 링크가 가리키는 디렉터리로 들어가서 순회를 계속합니다. 이때 재귀적으로 `filepath.Walk`를 호출하여 해당 심볼릭 링크가 가리키는 경로를 순회합니다.
3. 핵심은 여기서 파일을 순회하면서 방문한 적이 있었다면, 더 방문하지 않고, 중복 방문했던 곳을 찾는건데요. 본 프로그램은 심볼릭 링크(링크 파일) 의 위치를 찾는 프로그램임을 알 수 있습니다.
