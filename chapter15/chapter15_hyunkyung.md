
# 15장 엔터티와 인코딩
- HTTP가 보장하는 것
    - 객체는 올바르게 식별되므로 브라우저나 다른 클라이언트는 콘텐츠를 바르게 처리할 수 있음
    - 객체는 올바르게 압축이 풀릴 것임
    - 객체는 항상 최신
    - 사용자의 요구를 만족할 것
    - 네트워크 사이를 빠르고 효율적으로 이동할 것
    - 조작되지 않고 온전하게 도착할 것

## 15.1 메시지는 컨테이너, 엔터티는 화물
- HTTP 메시지를 인터넷 운송 시스템의 컨테이너라고 생각한다면, HTTP 엔터티는 메시지의 실질적인 화물
- 10가지 주요 엔터티 헤더 필드
    - Content-Type: 엔터티에 의해 전달된 객체의 종류
    - Content-Length: 전달되는 메시지의 길이나 크기
    - Content-Language: 전달되는 객체와 가장 잘 대응되는 자연어
    - Content-Encoding: 객체 데이터에 대해 행해진 변형
    - Content-Location: 요청 시점을 기준으로, 객체의 또 다른 위치
    - Content-Range: 만약 이 엔터티가 부분 엔터티라면, 이 헤더는 이 엔터티가 전체에서 어느 부분에 해당하는지 정의함
    - Content-MD5: 엔터티 본문의 콘텐츠에 대한 체크섬
    - Last-Modified: 서버에서 이 콘텐츠가 생성 혹은 수정된 날
    - Expires: 이 엔터티 데이터가 더 이상 신선하지 않은 것으로 간주되기 시작하는 날짜와 시각
    - Allow: 이 리소스에 대해 어떤 요청 메서드가 허용되는지
    - ETag: 이 인스턴스에 대한 고유한 검사기.
    - Cache-Control: 어떻게 이 문서가 캐시될 수 있는지에 대한 지시자

### 15.1.1 엔터티 본문
- 엔터티 본문: 가공되지 않는 데이터만을 담고 있음, 엔터티 헤더가 설명할 필요를 가짐
- 엔터티 본문은 헤더 필드의 끝을 의미하는 빈 CRLF 줄 바로 다음부터 시작

## 15.2 Content-Length: 엔터티의 길이
- Content-Length 헤더는 메시지의 엔터티 본문의 크기를 바이트 단위로 나타냄
- Content-Length 헤더는 메시지를 청크 인코딩으로 전송하지 않는 이상, 엔터티 본문을 포함한 메시지에서는 필수적으로 있어야 함
### 15.2.1 잘림 검출
- 클라이언트는 메시지 잘림을 검출하기 위해 Content-Length를 필요로 함
### 15.2.2 잘못된 Content-Length
### 15.2.3 Content-Length와 지속 커넥션
- Content-Length는 지속 커넥션을 위해 필수
- HTTP 애플리케이션은 Content-Length 헤더로 어디까지가 엔터티 본문이고 어디부터가 다음 메시지인지 인식
### 15.2.4 콘텐츠 인코딩
- HTTP는 보안을 강화하거나 압축을 통해 공간을 절약할 수 있도록, 엔터티 본문을 인코딩할 수 있게 해줌
- 만약 본문의 콘텐츠가 인코딩되어 있다면, Content-Length 헤더는 인코딩되지 않은 원본의 길이가 아닌 인코딩된 본문의 길이를 바이트 단위로 정의
### 15.2.5 엔터티 본문 길이 판별을 위한 규칙
- 순서대로 적용되어야 함
    1. 본문을 갖는 것이 허용되지 않는 특정 타입의 HTTP 메시지에서는, 본문 계산을 위한 Content-Length 헤더가 무시. 이 경우 Content-Length 헤더는 부가 정보에 불과하며, 실제 본문 길이를 서술하지 않음
    2. 메시지가 Transfer-Encoding 헤더를 포함하고 있다면 메시지가 커넥션이 닫혀서 먼저 끝나지 않는 이상 엔터티는 '0바이트 청크'라 불리는 특별한 패턴으로 끝나야 함
    3. 메시지가 Content-Length 헤더를 갖는다면, Transfer-Encoding 헤더가 존재하지 않는 이상 Content-Length 값은 본문의 길이를 담게됨
    4. 메시지가 'multipart/byteranges' 미디어 타입을 사용하고 엔터티 길이가 별도로 정의되지 않았다면, 멀티파트 메시지의 각 부분은 각자가 스스로의 크기를 정의할 것
    5. 위의 어떤 규칙에도 해당되지 않는다면, 엔터티는 커넥션이 닫힐 때 끝남. 실질적으로, 오직 서버만이 메시지가 끝났다는 신호를 위해 커넥션을 닫을 수 있음
    6. HTTP/1.0 애플리케이션과의 호환을 위해, 엔터티 본문을 갖고 있는 HTTP/1.1 요청은 반드시 유효한 Content-Length 헤더도 갖고 있어야 함

## 15.3 엔터티 요약
- 최초 엔터티가 생성될 때 송신자는 데이터에 대한 체크섬을 생성할 수 있으며, 수신자는 모든 의도하지 않은 엔터티의 변경을 잡아내기 위해 그 체크섬으로 기본적인 검사를 할 수 있음

## 15.4 미디어 타입과 차셋(Charset)
- Content-Type 헤더 필드는 엔터티 본문의 MIME 타입을 기술함
- MIME 타입은 전달되는 데이터 매체의 기저 형식(HTML 파일, 마이크로소프트 워드 문서, MPEG의 표준화된 이름)
- 주 미디어 타입(텍스트, 이미지, 오디오)으로 시작해서 빗금(/), 미디어 타입을 더 구체적으로 서술하는 부 타입으로 구성됨
- 엔터티가 콘텐츠 인코딩을 거친 경우에도 Content-Type 헤더는 여전히 인코딩 전의 엔터티 본문 유형을 명시할 것

### 15.4.1 텍스트 매체를 위한 문자 인코딩
- Content-Type 헤더는 내용 유형을 더 자세히 지정하기 위한 선택적인 매개변수도 지원함
### 15.4.2 멀티파트 미디어 타입
- MIME '멀티파트' 이메일 메시지는 서로 붙어있는 여러 개의 메시지를 포함하며, 하나의 복합 메시지로 보내짐
### 15.4.3 멀티파트 폼 제출
- HTTP 폼을 채워서 제출하면, 가변 길이 텍스트 필드와 업로드 될 객체는 각각이 멀티파트 본문을 구성하는 하나의 파트가 되어 보내짐
- 멀티파트 본문은 여러 다른 종류와 길이의 값으로 채워진 폼을 허용함
### 15.4.4 멀티파트 범위 응답
- 범위 요청에 대한 HTTP 응답 또한 멀티파트가 될 수도 있음
- 그러한 응답은 Content-Type: multipart/byteranges 헤더 및 각각 다른 범위를 담고 있는 멀티파트 본문이 함께 옴

## 15.5 콘텐츠 인코딩
- HTTP 애플리케이션은 때때로 콘텐츠를 보내기 전에 인코딩을 하려고 함
- 서버는 콘텐츠를 압축하거나 암호화해서 보낼 수도 있음

### 15.5.1 콘텐츠 인코딩 과정
1. 웹 서버가 원본 Content-Type과 Content-Length 헤더를 수반한 원본 응답 메시지를 생성
2. 콘텐츠 인코딩 서버가 인코딩된 메시지를 생성. 인코딩된 메시지는 Content-Type은 같지만 Content-Length는 다름
3. 수신 측 프로그램은 인코딩된 메시지를 받아서 디코딩하고 원본을 얻음

### 15.5.2 콘텐츠 인코딩 유형
- HTTP는 몇 가지 표준 콘텐츠 인코딩 유형을 정의하고 확장 인코딩으로 인코딩을 추가하는 것도 허용함
- 인코딩은 각 콘텐츠 인코딩 알고리즘에 고유한 토큰을 할당하는 IANA를 통해 표준화됨
- `gzip`, `compress`, `deflare` 인코딩은 전송되는 메시지의 크기를 정보의 손실 없이 줄이기 위한 무손실 압축 알고리즘
- 이 중 `gzip`은 일반적으로 가장 효율적이고 가장 널리 쓰이는 압축 알고리즘

### 15.5.3 Accept-Encoding 헤더
- 서버에서 클라이언트가 지원하는 인코딩을 사용하도록 Accept-Encoding 요청 헤더를 통해 전달
- 클라이언트는 각 인코딩에 Q(quality) 값을 매개변수로 더해 선호도를 나타낼 수 있음(0.0 ~ 1.0)

## 15.6 전송 인코딩과 청크 인코딩
- 콘텐츠 인코딩은 콘텐츠 포맷과 긴밀히 연관되어 있음
- 전송 인코딩 또한 엔터티 본문에 적용되는 가역적 변환이지만, 구조적인 이유로 적용되며 콘텐츠 포맷과는 독립적

### 15.6.1 안전한 전송
- HTTP에서 전송된 메시지의 본문이 문제를 일으킬 수 있는 이유
    - 알 수 없는 크기: 몇몇 게이트웨이 애플리케이션과 콘텐츠 인코더는 콘텐츠를 먼저 생성하지 않고서는 메시지 본문의 최종 크기를 판단할 수 없음
    - 보안: 공용 전송 네트워크로 메시지 콘텐츠를 보내기 전에 전송 인코딩을 사용해 알아보기 어렵게 뒤섞어버릴수도 있음

### 15.6.2 Transfer-Encoding 헤더
- `Transfer-Encoding`: 안전한 전송을 위해 어떤 인코딩이 메시지에 적용되었는지 수신자에게 알려줌
- `TE`: 어떤 확장된 전송 인코딩을 사용할 수 있는지 서버에게 알려주기 위해 요청 헤더에 사용

### 15.6.3 청크 인코딩
- 청크 인코딩은 메시지를 일정 크기의 청크 여럿으로 쪼갬
- 서버는 각 청크를 순차적으로 보냄
- 청크와 지속 커넥션
    - 클라이언트와 서버 사이의 커넥션이 지속적이지 않다면, 클라이언트는 자신이 읽고 있는 본문의 크기를 알 필요가 없음
    - 클라이언트는 서버가 커넥션을 닫을 때까지를 본문으로 간주하고 읽을 것
    - 지속 커넥션에서는, 본문을 쓰기 전에 반드시 Content-Length 헤더에 본문의 길이를 담아서 보내줘야 함
    - 콘텐츠가 서버에서 동적으로 생성되는 경우에는, 보내기 전에 본문의 길이를 알아내는 것이 불가능할 것
    - 청크 인코딩이 이 딜레마에 대한 해법 제공
    - 동적으로 본문이 생성되면서, 서버는 그중 일부를 버퍼에 담은 뒤 그 한덩어리를 그의 크기와 함께 보낼 수 있음
    - 본문을 모두 보낼 때까지 이 단계를 반복
    - 서버는 크기가 0인 청크로 본문이 끝났음을 알리고 다음 응답을 위해 커넥션을 열린 채로 유지할 수 있음

- 청크 인코딩된 메시지의 트레일러
    - 다음 중 하나 이상의 조건을 만족하면 청크 메시지에 트레일러를 추가할 수 있음
        - 클라이언트의 TE 헤더가 트레일러를 받아들일 수 있음을 나타내고 있는 경우
        - 트레일러가 응답을 만든 서버에 의해 추가되었으며, 그 트레일러의 콘텐츠는 클라이언트가 이해하고 사용할 필요가 없는 선택적인 메타데이터이므로 클라이언트가 무시하고 버려도 되는 경우

### 15.6.4 콘텐츠와 전송 인코딩의 조합
### 15.6.5 전송 인코딩 규칙
- 전송 인코딩의 집합은 반드시 'chunked'를 포함해야 함. 유일한 예외는 메시지가 코넥션의 종료로 끝나는 경우 뿐
- 청크 전송 인코딩이 사용되었다면, 메시지 본문에 적용된 마지막 전송 인코딩이 존재해야 함
- 청크 전송 인코딩은 반드시 메시지 본문에 한 번 이상 적용되어야 함
## 15.7 시간에 따라 바뀌는 인스턴스
- HTTP 프로토콜은 어떤 특정한 종류의 요청이나 응답을 다루는 방법들을 정의하는데, 이것은 인스턴스 조작이라 불리며 객체의 인스턴스에 작용함
- 이들 중 대표적인 두 가지가 범위 요청, 델타 인코딩
- 클라이언트가 자신이 갖고 있는 리소스의 사본이 서버가 갖고 있는 것과 정확히 같은지 판단하고, 경우에 따라서 새로 요청할 것을 요구

## 15.8 검사기와 신선도
- 조건부 요청: 클라이언트가 서버에게 자신이 갖고 있는 버전을 말해주고 검사기를 사용해 자신의 사본 버전이 더 이상 유효하지 않을 때만 사본을 보내달라고 요청하는 것
### 15.8.1 신선도
- 서버는 클라이언트에게 얼마나 오랫동안 콘텐츠를 캐시하고 그것이 신선하다고 가정할 수 있는지에 대한 정보를 줄 것
- Expires 헤더는 문서가 만료되어 더 이상 신선하다고 간주할 수 없게 되는 정확한 날짜를 명시
- 클라이언트나 헤더의 시계가 동기화되지 않았을 가능성이 있음 -> 상대시간을 이용해 만료를 정의하는 메커니즘 필요
- Cache-Control 헤더는 문서의 최대 수명을 문서가 서버를 떠난 후로부터의 총 시간을 초 단위로 정함

### 15.8.2 조건부 요청과 검사기
- 새로 요청을 했지만 서버에서 최신의 문서를 가지고 있지 않을 수도 있음
- HTTP는 클라이언트에게 리소스가 바뀐 경우에만 사본을 요청하는 조건부 요청이라 불리는 특별한 요청을 할 수 있는 방법을 제공
- 특정 조건이 참일 때만 수행되는 HTTP 요청 메시지(If-로 시작하는 조건부 헤더에 의해 구현됨)
- 각 조건부 요청은 특정 검사기 위에서 동작함
- 검사기는 문서의 테스트된 특정 속성. 개념적으로, 일련번호나 버전 번호 혹은 문서의 최종 변경일과 같은 검사기
- 약한 검사기와 강한 검사기
    - 약한 검사기
        - 리소스의 인스턴스를 고유하게 식별하지 못하는 경우도 있음
        - ex) 바이트 단위 크기: 크기가 같더라도 내용이 다를 수 있으므로 바이트의 개수를 세는 방식으로 동작
    - 강한 검사기
        - 언제나 고유하게 식별
        - 리소스의 콘텐츠에 대한 암호 체크섬: 문서가 변경되면 함께 변경됨

- 최종 변경 시각은 약한 검사기: 정확도가 최대 1초에 불과하기 때문
- ETag 헤더는 강함 검사기: 서버는 ETag 헤더에 매 변경마다 구분되는 값을 넣어두기 때문
- 서버는 태그 앞에 W/ 를 붙임으로써 약한 엔터티 태그임을 알림

## 15.9 범위 요청
- HTTP는 클라이언트가 문서의 일부분이나 특정 범위만 요청할 수 있도록 해줌
- 범위 요청을 이용하면, HTTP 클라이언트는 받다가 실패한 엔터티를 일부 혹은 범위로 요청함으로써 다운로드를 중단된 시점에서 재개할 수 있음
- 서버는 클라이언트에게 자신이 범위를 받아들일 수 있는지 응답에 Accept-Range 헤더를 포함시키는 방법으로 알려줄 수 있음

## 15.10 델타 인코딩
- 변경사항이 있을 때 새 페이지 전체를 보내는 대신, 페이지에 대한 클라이언트의 사본에 대해 변경된 부분만을 서버가 보낸다면 클라이언트는 더 빨리 페이지를 얻을 것
- 델타 인코딩은 객체 전체가 아닌 변경된 부분에 대해서만 통신하여 전송량을 최적화하는, HTTP 프로토콜의 확장
- 일종의 인스턴스 조작 -> 어떤 객체의 특정 인스턴스들에 대한 클라이언트와 서버 사이의 정보 교환에 의존하기 때문
- 클라이언트는 델타(변경된 부분)을 적용하기 위해 어떤 알고리즘을 알고 있는지 서버에게 말해주어야 함
- 서버는 클라이언트의 버전을 갖고 있는지, 어떻게 델타를 계산할 것인지 체크
- 계산해서 보내주고, 델타를 보내고 있음을 알려주고, 새 식별자를 명시해야 함
- 클라이언트는 자신이 갖고 있는 버전에 대한 유일한 식별자를 If-None-Match 헤더에 담음
- 이 헤더에 의해 최신 버전의 페이지를 받을 수도 있음
- 그러나 클라이언트는, 서버에게 A-IM 헤더를 보내서 자신이 페이지에 대한 델타를 받아들일 수 있음을 알려줄 수도
- A-IM == Accept-Instance-Manipulation

### 15.10.1 인스턴스 조작, 델타 생성기 그리고 델타 적용기
- 서버의 '델타 생성기'는 기저 문서와 그 문서의 최신 인스턴스를 취하여 클라이언트의 A-IM 헤더에 지정된 알고리즘을 이용해 둘 사이의 델타를 계싼함
- 델타 인코딩은 전송 시간을 줄일 수 있지만 구현하기가 까다로울 수 있음
- 과거 사본을 모두 유지하기 위해 디스크 공간을 더 늘려야 함 -> 전송량 감소로 얻은 이득을 무의미하게 만들 것