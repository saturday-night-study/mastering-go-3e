# 유닉스 시그널 처리

유닉스 **시그널(Signal)**이란, 운영체제에서 프로세스에 발생한 특정 이벤트를 알리기 위한 비동기적인 통지 메커니즘입니다. 운영체제는 시그널을 사용하여 프로세스에게 다양한 이벤트를 효과적으로 전달할 수 있으며, 프로세스는 이에 적절히 반응하여 작업을 수행하거나 상태를 변경할 수 있습니다.

* 프로세스 간 통신 (IPC) : 다른 프로세스 또는 운영 체제로부터 시그널을 받아 특정 작업을 수행하거나 프로세스의 상태를 변경
* 비정상 종료 처리 : 예외 상황이나 치명적인 오류가 발생했을 때 프로세스를 안전하게 종료시키기 위해서 사용
* 외부 이벤트 반응 : 사용자로부터 입력이나 시스템 이벤트에 반응하기 위해 사용

유닉스에서는 아래와 같은 시그널이 존재합니다.

<table><thead><tr><th width="151">시그널</th><th>설명</th></tr></thead><tbody><tr><td>SIGINT</td><td>인터럽트 시그널<br>일반적으로 사용자가 Ctrl+C를 입력했을 때 발생</td></tr><tr><td>SIGTERM</td><td>종료 시그널<br>시스템이나 사용자가 프로세스를 종료하고자 할 때 보내는 시그널<br>프로세스는 이 시그널을 받으면 정리 작업을 수행하고 안전하게 종료</td></tr><tr><td>SIGKILL</td><td><p>즉시 종료 시그널</p><p>프로세스에 의해 잡히거나 무시될 수 없으며, 운영체제가 프로세스를 강제로 즉시 종료</p></td></tr><tr><td>SIGSEGV</td><td>세그먼테이션 위반 시그널<br>프로세스가 유효하지 않은 메모리를 참조할 때 발생</td></tr><tr><td>SIGCHLD</td><td>자식 프로세스 상태 변경 시그널<br>자식 프로세스가 종료되거나 멈추었을 때 부모 프로세스에게 전달</td></tr></tbody></table>

Go 언어에서 유닉스 시그널을 처리하려면 **고루틴과 채널**을 사용해야 합니다. 고루틴은 Go의 **동시성 실행 단위**입니다. 고루틴(Gorouine)을 생성하려면 동시성 생성자인 go 키워드와 함께 함수를 실행합니다. 채널(Channel)은 고루틴 간에 데이터를 주고받을 수 있도록 상호작용하는 매커니즘입니다.

아래 예제는 **SIGINT**와 **SIGUSR1**를 처리합니다. Go 언어에서는 각 시그널을 `syscall.SIGINT, syscall.SIGUSR1`로 표현합니다. 프로세스가 운영체제로부터 시그널을 받으면 채널을 이용해서 시그널을 전달 받습니다. 채널에 시그널이 들어오기를 대기하는 고루틴을 만들고 채널에 실제 시그널이 들어오면 적절한 처리를 진행합니다.

{% embed url="https://gist.github.com/kineo2k/157f29a5986eb1013bce15e4d8f12160" %}
