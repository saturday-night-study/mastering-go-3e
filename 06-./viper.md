# viper 패키지

## 개요

`플래그(flag)`란 프로그램의 동작을 제어하고자 하는 특별한 형태의 문자열 값입니다. 여러 가지의 플래그와 옵션을 지원하기 위해 코드를 구성하는 것은 매우 어렵고 귀찮은 일이죠. :joy:

Go에서 제공하는 flag 패키지를 사용해 커맨드라인 옵션, 매개변수, 플래그를 관리할 수 있지만 외부 패키지보다는 적은 기능을 제공하고 있기 때문에 더욱 좋은 기능을 제공하는 `viper` 패키지를 사용하여 구성해보도록 하겠습니다.

### viper 패키지 사용예시코드

```go
package main

import (
	"fmt"

	"github.com/spf13/pflag"
	"github.com/spf13/viper"
)

func aliasNormalizeFunc(f *pflag.FlagSet, n string) pflag.NormalizedName {
	switch n {
	case "pass":
		n = "password"
		break
	case "ps":
		n = "password"
		break
	}
	return pflag.NormalizedName(n)
}

func main() {
	pflag.StringP("name", "n", "Mike", "Name parameter")
	pflag.StringP("password", "p", "hardToGuess", "Password")
	pflag.CommandLine.SetNormalizeFunc(aliasNormalizeFunc)

	pflag.Parse()
	viper.BindPFlags(pflag.CommandLine)

	name := viper.GetString("name")
	password := viper.GetString("password")

	fmt.Println(name, password)

	// Reading an Environment variable
	viper.BindEnv("GOMAXPROCS")
	val := viper.Get("GOMAXPROCS")
	if val != nil {
		fmt.Println("GOMAXPROCS:", val)
	}

	// Setting an Environment variable
	viper.Set("GOMAXPROCS", 16)
	val = viper.Get("GOMAXPROCS")
	fmt.Println("GOMAXPROCS:", val)
}
```

### 결과 예시화면

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption><p>--help 플래그를 사용한 예시</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption><p>플래그를 사용하지 않았을 때 예시</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>플래그를 통해 값을 변경한 예시</p></figcaption></figure>

## JSON 설정 파일 읽기

`viper` 패키지는 flag 외 JSON 설정 파일을 직접 읽어오는 것도 가능합니다.

### viper 패키지를 사용한 JSON 설정 파일 읽기 예시

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"

	"github.com/spf13/viper"
)

type ConfigStructure struct {
	MacPass     string `mapstructure:"macos"`
	LinuxPass   string `mapstructure:"linux"`
	WindowsPass string `mapstructure:"windows"`
	PostHost    string `mapstructure:"postgres"`
	MySQLHost   string `mapstructure:"mysql"`
	MongoHost   string `mapstructure:"mongodb"`
}

func PrettyPrint(v interface{}) (err error) {
	b, err := json.MarshalIndent(v, "", "  ")
	if err == nil {
		fmt.Println(string(b))
	}
	return
}

var CONFIG = ".config.json"

func main() {

	if len(os.Args) == 1 {
		fmt.Println("Using default file", CONFIG)
	} else {
		CONFIG = os.Args[1]
	}

	viper.SetConfigType("json")
	viper.SetConfigFile(CONFIG)
	fmt.Printf("Using config: %s\n", viper.ConfigFileUsed())
	viper.ReadInConfig()

	if viper.IsSet("macos") {
		fmt.Println("macos:", viper.Get("macos"))
	} else {
		fmt.Println("macos not set!")
	}

	if viper.IsSet("active") {
		value := viper.GetBool("active")
		if value {
			postgres := viper.Get("postgres")
			mysql := viper.Get("mysql")
			mongo := viper.Get("mongodb")
			fmt.Println("P:", postgres, "My:", mysql, "Mo:", mongo)
		}
	} else {
		fmt.Println("active is not set!")
	}

	if !viper.IsSet("DoesNotExist") {
		fmt.Println("DoesNotExist is not set!")
	}

	var t ConfigStructure
	err := viper.Unmarshal(&t)
	if err != nil {
		fmt.Println(err)
		return
	}
	PrettyPrint(t)
}
```

### 예시 결과화면

먼저 JSON 설정파일을 go 파일과 동일한 경로에 생성해줍니다.

```json
{
    "macos": "pass_macos",
    "linux": "pass_linux",
    "windows": "pass_windows",
    
    "active": true,
    "postgres": "machine1",
    "mysql": "machine2",
    "mongodb": "machin3"
}
```

이후 아래와 같이 go 파일을 실행하면 결과 값을 확인 할 수 있습니다. :tada:

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>jsonViper.go 프로그램 실행 화면</p></figcaption></figure>

## 실시간으로 파일 설정 읽어오기

viper 패키지는 위 제공 기능 외에 실시간으로 설정 파일 변경을 감지해 이를 활용해 Hot-load 할 수 있는 기능도 제공하고 있습니다. :thumbsup:

이 기능을 사용하면 서버 설정 값 변경을 위해 프로그램을 다시 재 실행할 필요가 없어집니다.

```go
package main

import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
)

type Config struct {
	Name string `yaml:"name"`
}

func main() {
	viperConfig := viper.New()
	viperConfig.AddConfigPath(".")
	viperConfig.SetConfigFile("config.yaml")

	err := viperConfig.ReadInConfig()
	if err != nil {
		fmt.Println("Error on Reading Viper Config")
		panic(err)
	}

	var config Config
	err = viperConfig.Unmarshal(&config)
	if err != nil {
		fmt.Println("Error on Unmarshal Viper Config")
		panic(err)
	}
	fmt.Println(config)

	viperConfig.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("config 파일 변경 감지 !!")
	})
}
```
