# cobra 패키지

## 개요

`cobra` 패키지는 `커맨드(Command)`, `하위 커맨드(Sub-Command)`, `별칭(Alias)` 등 [CLI](https://ko.wikipedia.org/wiki/%EB%AA%85%EB%A0%B9%EC%A4%84\_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4) 유틸리티 프로그램을 만드는데 사용되는 인기있는 패키지입니다.

hugo, docker, kubectl과 같은 도구를 사용해보셨다면 아마 패키지를 사용하는 방법에 대해서 금세 이해할 수 있을 거에요 (위 프로그램이 cobra 패키지를 활용해 구성 되었다고 하네요 !!)

## 세 개의 커맨드가 있는 유틸리티

해당 파트에서는 `cobra add` 커맨드를 사용해 새로운 커맨드를 추가해봅시다. 커맨드의 이름은 one, two, three로 진행됩니다.

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

위 와 같은 형태로 `cobra` 명령어를 입력하면 최상단 폴더 아래에 `cmd` 폴더 내부에 `one.go`, `two.go`, `three.go` 3개의 파일이 자동으로 생성됩니다.

이후 가장 먼저 `root.go` 파일에서 사용하지 않을 내용을 삭제하거나 필요한 내용을 수정합니다.

## 커맨드라인 플래그 추가

각각의 커맨드에는 플래그를 추가해서 사용할 수 있습니다. 여기서는 문자열 플래그인 directory와 depth 플래그를 정의합니다.&#x20;

해당 플래그는 모두 `cmd` 폴더 아래 있는 `root.go` 내부의 `init()` 함수에 정의하면 됩니다.

```go
// ./cmd/root.go

package cmd

import (
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

// A Global variable
var Special = "This is a special message."

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
	Use:   "go-cobra",
	Short: "A brief description of your application",
	Long:  `This is a long description of the commnad line application.`,
}

// Execute adds all child commands to the root command and sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
	cobra.CheckErr(rootCmd.Execute())
}

func init() {
	rootCmd.PersistentFlags().StringP("directory", "d", "/tmp", "Path to use.")
	rootCmd.PersistentFlags().Uint("depth", 2, "Depth of search.")

	viper.BindPFlag("directory", rootCmd.PersistentFlags().Lookup("directory"))
	viper.BindPFlag("depth", rootCmd.PersistentFlags().Lookup("depth"))
}
```

위 예시코드에서 볼 수 있듯이 `rootCmd.PersistnetFlags()` 함수를 사용해 전역 플래그와 플래그의 데이터 타입을 정의할 수 있습니다.

여기서 StringP 와 같은 형태의 함수를 사용하면 플래그 입력 시에 2가지 유형으로 사용될 수 있습니다.

```bash
# 아래 두 명령 플래그는 동일하게 동작함

$ exmaple.go --directory ./dir
$ example.go -d ./dir
```

## 커맨드 별칭(Alias) 생성

기존 생성 된 커맨드에 별칭을 생성해 코드를 손 쉽게 변경할 수 있습니다. `one`명령에 대응되는 `cmd1`과 같이 별칭을 만들어 사용해보는 예시를 작성해보겠습니다.

```go
// ./cmd/one.go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

// oneCmd represents the one command
var oneCmd = &cobra.Command{
	Use:     "one",
	Aliases: []string{"cmd1"},
	Short:   "Command one",
	Long:    `This is the description for one.`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("one called")

		dir := viper.GetString("directory")
		if dir == "" {
			fmt.Println("Dir is", dir)
		}

		depth := viper.GetUint("depth")
		fmt.Println(dir, depth)
	},
}

func init() {
	rootCmd.AddCommand(oneCmd)
}
```

{% hint style="warning" %}
실수로 여러 커맨드에 cmd1이나 다른 별칭을 지정할 경우에는 Go 컴파일러가 오류로 표시 하지 않습니다.&#x20;

만약 중복 된 별칭을 지정할 경우에는 첫 번째 지정한 별칭만 실행이 됩니다.
{% endhint %}
