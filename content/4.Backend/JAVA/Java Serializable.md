# Java Serializable

자바 직렬화

혼자 프로젝트를 진행하던 와중에 최초에 생성된 객체의 정보를 다른 Servlet에서 사용하려고 할 때  java.io.notserializableexception 라는 에러가 뜨면서 프로세스가 진행 되지 않는 오류가 있었다.

해서 인터넷을 조금 뒤져보니 직렬화 와 역직렬화에 관련된 문제 였던 것이었다.

- 사실 해결은 해당 객체에 implements Serializable하면서 해결이 되었다.

1.  직렬화란?

객체를 컴퓨터간 혹은 네트워크 상에서 주고 받을 때 (각 운영체제마다 다른 JVM을 사용 함으로) 송수신을 위해서 객체를 데이터 스트림에 담아 주어야 한다. 이때 우리가 흔히 아는 CSV나 JSON처럼 객체를 연속적인 데이터로 변환하는 것을 말합니다!

1. 역직렬화란?

직렬화 된 객체들을 다시 읽어오는 과정을 역질렬화라고 합니다.

이해한 바로는 client가 데이터를 요청할 때 서버쪽에서 생성된 객체가 통신하는 과정에서 서로 주고 받아야 하는데 객체는 주소 값으로 저장이 되면서 서버의 객체 주소를 그대로 사용하면  client의 주소와 맞지 않기 때문에 이를 방지하고자 **객체를 binary데이터로 만들어서 넘겨주는 역할을 하는 것이 직렬화** 그리고 **이를 해석해서 다시 객체로 만드는 것이 역질렬화이다**.

```java
public interface Serializable {
}
```

위에 보이는 코드 블럭이 Serializable 의 interface인데 딱 이거 하나 있는 걸 볼 수 있다.

결국 Serializable를 implements 하면서 이 객체를 직렬화해달라고 marking하는 것임을 알 수 있다.

1. 직렬화는 어떻게 하는 건데?

Todo라는 class가 Serializable을 implements하면서 만들어 질 때

```java
class Todo implements Serializable{
	String todo;
	String state;
	int date;

	public Todo(...){...}

}
```

java.io.ObjectOutputStream을 이용하면 할 수 있습니다.

```java
import vo.Todo

Todo todo = new Todo(...);
byte[] serializedTodo;
try{BtyeArrayOutputStream baos = new BtyeArrayOutputStream();
	try{
			ObjectOutputStream oss = new ObjectOutputStream();
			oos.writeObject(todo);
			serializedTodo =baos.toBtyeArray();
	}
}
Base64.getEncoder().encodeToString(serializedTodo);
```

결과적으로는 객체를 byte형태로 수송신 할 수 있게 할 수 있다는 것이다!

1. 그럼 역직렬화는?

역직렬화는 일단 객체의 클래스가 클래스 path에 존재해야하면 import 되어 있어야 합니다.

이때 직렬화 대상 객체는 동일한 serialVersionUID를 가지고 있어야 합니다.

```java
public class TodoAdd extends HttpServlet {
	private static final long serialVersionUID = 1L;
...
}
```

가끔 Servlet을 생성하면 final long 값으로 serialVersionUID 값을 자동으로 생성해 주는데 요 부분이 같게 있어야 역직렬화가 가능합니다. 이후에는 아래와 같은 code로  byte를 decode해주고 객체를 풀어주면 됩니다.

```java
import vo.Todo
String base64Member = "...생략";
byte[] serializedTodo = Base64.getDecoder().decode(base64Member);
try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedTodo)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        Object objectMember = ois.readObject();
        Todo todo = (Todo) objectMember;
    }
}
```

1. 보통 직렬화는 어디서 쓰는가?

문제는 이 자바 직렬화는 CSV나 JSON과는 좀 다르게 java system에서 만 사용을 하기 때문에 어느 정도 한정적일 수 있다는 한계점이 있지만, 편의에 따라서 사용을 하면 좋을 것 같다. ( 아직 실제로 구현하고 사용하는 단계까지는 가본 적이 없다.)

직렬화는 JVM메모리에 있는 객체 데이터를 그래도 사용하기 위해서 필요합니다. 주로 Servlet Session (여기서 Error가 났습니다.) , Cache, RMI (이 부분은 처음 들어보네요) 에서 사용을 한다고 합니다. 

이중에 Servlet Session에 대해 조금 더 이야기 해보자면 서블릿 기반에 WAS들은 대부분 자바 직렬화를 지원합니다. 단순히 Session을 Servlet 메모리에서 운용한다면 직렬화를 필요로 하지 않지만 파일을 저장, 세션 클러스터링 DB를 저장하는 행위를 하게 되면 세션 자체가 직렬화 되어 저장되어 전달해 집니다. 

프로젝트 하는 입장에서 보면 어느 부분에서 꼬였는지 명확하게 현재는 알지 못하겠다...

html page에서 Session에 담긴 객체를 사용하지 않아도 이런 이슈가 생길 수 있다고 현재는 생각하고 있다. 이후에 드는 질문은 아래와 같다.

- 내가 Session에 한번 넣으면 계속 그 값을 유지 하면서 사용할 수 있는 것이 아닌가?
- Context, Session, request 이런 범위에 어떻게 값을 집어 넣는 것이 효율적인 것인가?

- 참고자료

[자바 직렬화, 그것이 알고싶다. 훑어보기편 | 우아한형제들 기술블로그](https://techblog.woowahan.com/2550/)