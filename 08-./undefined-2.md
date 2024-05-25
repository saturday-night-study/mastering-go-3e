# 웹 클라이언트 개발

### http.Get()으로 간단한 웹 클라이언트 개발

`http.Get()`을 사용해서 간단한 클라이언트를 만들어 보겠습니다.

{% embed url="https://gist.github.com/kineo2k/642331b0cfde0763c2539e01882befad" %}

`http.Get()` 을 사용한 방법은 간단한 만큼 유연함이 떨어집니다. 또한 위 예제에서는 URL이 존재하고 접근 가능한지에 대해서 검사하는 과정에 아쉬움이 있습니다.



### http.NewRequest()를 사용해 클라이언트 개선

{% embed url="https://gist.github.com/kineo2k/d11c4cf7632feab3a86d719edd8dcd59" %}

### 전화번호부 서비스의 클라이언트 만들기

