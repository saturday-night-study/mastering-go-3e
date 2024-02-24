# 텍스트 파일 읽기

#### 줄 단위로 텍스트 파일 읽기

다음 예제는 텍스트 파일을 줄 단위로 읽어서 출력하는 예제입니다. 파일을 열때는 `os.Open()` 함수를 사용합니다.

{% embed url="https://gist.github.com/kineo2k/5807d8d4d15787632f9d8304fb12f454" %}

#### 단어 단위로 텍스트 파일 읽기

{% embed url="https://gist.github.com/kineo2k/951ccc31de99c81323478f78c5406b56" %}

#### 문자 단위로 텍스트 읽기

{% embed url="https://gist.github.com/kineo2k/a7b4f9bb1e5c6be24c814423cf41cdd5" %}

#### /dev/random 읽기

`/dev/random` 시스템 장치는 랜덤 데이터를 생성하고자 존재합니다. `/dev/random`에서 읽을때는 바이너리 데이터로 읽기 때문에 이를 정수값으로 변환해서 사용해야 합니다. 바이너리 데이터를 정수값으로 변환하기 위해서 `encoding/binary` 패키지를 사용합니다. 실습을 위해서 M1 macbookpro를 사용중이라서 정수값 변경 시 리틀 엔디안(Little endian)을 사용했습니다.

{% embed url="https://gist.github.com/kineo2k/8f93493737a7b12be228ea1e19447cbf" %}

#### 파일에서 원하는 만큼만 데이터 읽기

파일에서 원하는 크기만큼 데이터를 읽고 싶은 경우 읽기 버퍼의 크기를 읽고 싶은 크기로 설정해서 읽습니다.

{% embed url="https://gist.github.com/kineo2k/a0148ce821557cf32e3ec056f921a913" %}
