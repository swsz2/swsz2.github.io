---
layout: post title: 'Jsoup throws "IOException: stream is closed"'
subtitle: 'Java HTML 라이브러리 Jsoup의 캐시 처리'
categories: devlog 
tags: java 
comments: true
---

## 이슈 발생

<pre>
 ???  : swsz2님! HTTP 요청할 때 IOE 발생하는데 어떻게 처리해야 하나요?   
swsz2 : 메시지는요?
 ???  : stream is closed 이라고 나와요!  
swsz2 : 스트림 닫지 말고 처리하면 되지 않을까요?
 ???  : 닫지 않았는데...
swsz2 : ???
</pre>

## 이슈 분석

후임의 질문에 바로 이슈 트래킹을 시작했습니다.

- 상황 요약 : HTTP 요청을 할 때 Stream is closed라는 메시지와 함께 IOE가 발생
- 특이사항
    - 동시에 동일한 웹 사이트의 다른 페이지로 HTTP 요청을 보내는 도중 발생
    - 위 작업은 병렬로 수행
    - Jsoup 1.13.1 사용
    - 외부에서는 스트림을 닫는 코드가 존재하지 않음
      <br />

이러한 상황을 봤을 때 의심이 되는 부분이 한 가지 있었습니다.
<pre>
동시, 병렬로 수행한다, 그리고 외부엔 스트림을 닫는 코드가 존재하지 않는다.
→  HTTP 요청을 병렬로 처리하면 기본적으로 캐시를 공유하기 때문에
선행된 요청의 스트림이 닫힌 후 후행 요청이 닫힌 스트림을 참조하는 케이스가 발생하지 않을까?</pre>  

### [참고]

java net의 ResponseCache 관련 document를 보면 아래와 같은 내용을 볼 수 있습니다.

- https://docs.oracle.com/javase/8/docs/api/java/net/ResponseCache.html

<pre>
Retrieve the cached response based on the requesting uri, request method and request headers. Typically this method is called by the protocol handler before it sends out the request to get the network resource. If a cached response is returned, that resource is used instead.
요청 uri, 요청 방법 및 요청 헤더를 기준으로 캐시된 응답을 검색합니다. 일반적으로 이 메서드는 네트워크 리소스를 가져오기 위한 요청을 전송하기 전에 프로토콜 처리기에 의해 호출됩니다. 캐시된 응답이 반환되면 해당 리소스가 대신 사용됩니다.
</pre> 


이러한 의심과 함께 Jsoup github issue를 확인해보니 역시 동일한 케이스가 있었는데요.
![jsoup-stream-is-closed-issue-open](../assets/img/post/jsoup-stream-is-closed-issue-open.png)
없어졌습니다.
![jsoup-stream-is-closed-issue-closed](../assets/img/post/jsoup-stream-is-closed-issue-closed.png)

## 이슈 해결

그래서 직접 버그를 수정하기로 했습니다.

<pre>
1. Jsoup에서 제공하는 Connection, Response 객체의 기능을 그대로 활용하기 위해 usermodel 패키지에 기능 이관
2. Connection 객체 생성 시 시스템 캐시 활성화 여부 설정 추가
    -  캐시가 필요한 경우도 있으니 default는 false로 하되 외부에서 설정할 수 있게 한다.
</pre>

### 코드

```java
public interface Connection {
    Connection useCache(boolean useCache);
}
```

```java
 public static class Request extends HttpConnection.Base<Connection.Request> implements Connection.Request {
    private boolean useCache;

    Request() {
        useCache = false;
    }
}
```

```java
public static class Response extends HttpConnection.Base<Connection.Response> implements Connection.Response {
    static HttpConnection.Response execute(Connection.Request req, HttpConnection.Response previousResponse) throws IOException {
        HttpURLConnection conn = createConnection(req);
        conn.setUseCaches(req.useCache());
        return res;
    }
}
```

*전체 코드를 참고하고 싶으시다면 댓글 부탁드리겠습니다.*

## 아쉬운 점

github에 issue가 등록됐음에도 불구하고 특별한 이유 없이 처리하지 않았다는 것은 개발자의 의도와 다르게 사용하고 있기 때문이라는 생각이 듭니다.  
위와 같은 처리는 임시방편이라고 생각하고 apache http component에 대한 연구를 시작해야 할 거 같습니다. (https://hc.apache.org/)

## P.S

두서없는 글을 읽어주셔서 감사합니다.  