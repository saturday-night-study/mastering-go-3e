# 전화전호부 애플리케이션 업데이트

금번 전화번호부 업데이트는 대부분 기능이 구현 되어 있으므로 두가지 기능만 구현 해 보겠습니다.

### 신규 기능 추가

* Json Import / Export 구현
* Import embed & io/fs 를 이용하여 구현

### Json Import 기능

`embed` 폴더와 & `io/fs` 패키지를 이용하기전에 `embed`는 실행되는 폴더 바깥을 참조하기 어려운데요. (상대 주소를 사용하기 어려움)

아래와 같이 폴더를 설정하여 embed 할수있도록 설정합니다.

<div align="left">

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption><p>프로젝트 폴더 구성 화면</p></figcaption></figure>

</div>

그후 아래와 같이 코드를 작성하여  json 파일을 읽어 db 에 저장할 수 있도록 합니다.

```go
//go:embed static  
var f embed.FS  
  
func init() {  
    rootCmd.AddCommand(&cobra.Command{  
       Use: commandJsonImport,  
       Run: func(cmd *cobra.Command, args []string) {  
          file, err := fs.ReadFile(f, "phonebook.json")  
          if err != nil {  
             return  
          }  
  
          var es []entries.Entry  
          err = json.Unmarshal(file, &es)  
          if err != nil {  
             return  
          }  
  
          service, err := di.GetService[entries.EntryRepository]()  
          if err != nil {  
             return  
          }  
  
          for _, element := range es {  
             _, err = service.Add(element.Name, element.Surname, element.Tel)  
             if err != nil {  
                panic(fmt.Errorf("db error : %v", err))  
             }  
          }  
       }})  
}

```

### Json Export 기능

export 기능은 `io/fs` 에서 지원하지 않는데요, CSV 와 같이 export 를 동일하게 진행합니다.

```go
func init() {  
    rootCmd.AddCommand(&cobra.Command{  
       Use: commandJsonExport,  
       Run: func(cmd *cobra.Command, args []string) {  
          service, err := di.GetService[entries.EntryRepository]()  
          if err != nil {  
             panic(fmt.Errorf("error : %v", err))  
          }  
  
          results, err := service.GetAll()  
          if err != nil {  
             panic(fmt.Errorf("not found error : %v", err))  
          }  
  
          marshal, err := json.Marshal(results)  
          if err != nil {  
             return  
          }  
  
          file, err := os.OpenFile(envs.PhoneBookJsonLocation, os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644)  
          if err != nil {  
             panic(fmt.Errorf("could not create phonebook.json: %v", err))  
          }  
  
          defer func(file *os.File) {  
             _ = file.Close()  
          }(file)  
  
          _, err = file.Write(marshal)  
          if err != nil {  
             return  
          }  
  
          fmt.Println("export success")  
       }})  
}

```
