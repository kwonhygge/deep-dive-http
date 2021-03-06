# 17. 내용 협상과 트랜스코딩

* 하나의 URL이 여러 리소스에 대응할 필요가 있는 경우가 있다: 같은 URL의 웹페이지가 영어, 프랑스어 콘텐츠를 제공하는 경우
* HTTP는 클라이언트와 서버가 이런 판단을 할 수 있게 내용 협상(content-negotiation) 방법을 제공하며, 이로써 하나의 URL이 여러 리소스 중 적합한 것에 대응되도록 할 수 있다.
  * 여기서 서로 다른 버전을 배리언트(variant)라고 한다.
* 서버는 특정 URL에 대해 어떤 콘텐츠를 클라이언트에게 보내주기 적절한지 판단할 수 있어야 한다.
  * 어떤 경우 서버가 커스터마이징된 페이지를 자동 생성하기도 하며, HTML페이지를 WAP 환경에서 동작하는 구형 휴대용 단말기에 맞게 WML 페이지로 변환할 수도 있다.
  * 트랜스코딩은 HTTP 클라이언트와 서버 사이의 내용 협상에 대한 응답에서 수행된다.

## 17.1 내용 협상 기법

* 서버의 페이지들 중 어떤 것이 클라이언트에게 맞는지 판단하는 세 가지 내용 협상 방법이 있다.

| 기법 | 어떻게 동작하는가 | 장점 | 단점 |
|:---|:---|:---|:---|
| 클라이언트 주도 | 클라이언트가 요청을 보내면, 서버는 클라이언트에게 선택지를 보내주고, 클라이언트가 선택함 | 서버 입장에서 가장 구현하기 쉬움. 클라이언트는 최선의 선택을 할 수 있음 | 대기시간이 늘어남. 올바른 콘텐츠를 얻으려면 최소 두 번의 요청이 필요함 |
| 서버 주도 | 서버가 클라이언트 요청 헤더를 검증하여 어떤 버전 제공할지 결정 | 클라이언트 주도 협상보다 빠름. HTTP는 서버가 가장 적절한 선택을 할 수 있도록 `q`값 메커니즘을 제공하고, 서버가 다운스트림 장치에게 요청이 어떻게 평가되는지 알려주기 위해 `Vary` 헤더를 제공 | 헤더에 맞는 것이 없으면 서버는 추측해야 함 |
| 투명 | 투명한 중간 장치(주로 프락시 캐시)가 서버를 대신해 협상함 | 웹 서버가 협상할 필요가 없음. 클라이언트 주도 협상보다 빠름 | 투명 협상을 어떻게 하는지에 대한 정형화된 명세가 없음 |

## 17.2 클라이언트 주도 협상

* 서버가 클라이언트의 요청을 받았을 때 가능한 페이지의 목록을 응답으로 돌려주어 클라이언트가 원하는 것을 선택하게 하는 것이다.
* 장점: 서버 입장에서 가장 구현이 쉽고, 클라이언트가 올바른 사본을 선택할 수 있는 충분한 정보를 제공받을 경우 최선의 사본을 선택할 수 있다.
* 단점: 각 페이지에 두 번의 요청이 필요하다. 한 번은 목록을 얻는 것, 두 번은 선택한 사본을 얻는 것이다. 이 과정은 느리고 지루하며 클라이언트에게는 번거로운 일이다.
  * 또한 여러 개의 URL(주 페이지이 하나와 각 특정 조건별 페이지들)을 요구한다는 단점이 있다: `~.com` 요청에 대해 `~.com/english`, `~.com/french` 링크를 담은 페이지를 반환하게 된다.
* 기술적으로 서버는 클라이언트에게 줄 선택지를 표현하는 두 방법이 있다.
  * 여러 버전에 대한 링크와 각 설명이 담긴 HTML 페이지를 돌려주거나, `300 Multiple Choices` 응답 코드로 HTTP/1.1 응답을 돌려주는 것이다.
  * 첫 번째 메서드의 결과로 클라이언트 브라우저는 이런 응답을 받아 링크와 함께 페이지를 보여주거나, 사용자가 결정할 수 있게 대화창을 띄운다. 이 경우 결정은 클라이언트 쪽 브라우저에서 사용자에 의해 수동으로 행해진다.
## 17.3 서버 주도 협상

* 서버가 어떤 페이지를 돌려줄 것인지 결정하게 하여 클라이언트와 서버 사이 커뮤니케이션을 줄일 수 있다.
* 그러기 위해서 클라이언트는 반드시 자신의 선호에 대한 충분한 정보를 서버에게 제공해야 한다. 서버는 이 정보를 클라이언트 요청 헤더에서 얻는다.
* HTTP 서버가 클라이언트에게 보내줄 적절한 응답 계산에 사용하는 메커니즘
  * 내용 협상 헤더들을 살펴본다. 서버는 클라이언트의 `Accept` 관련 헤더를 보고 그에 맞는 응답 헤더를 준비한다.
  * 내용 협상 헤더 외에 다른 헤더를 살펴본다. 예를 들어 서버는 클라이언트의 `User-Agent` 헤더에 기반해 응답을 보낼 수도 있다.

### 17.3.1 내용 협상 헤더

* 클라이언트는 다음 HTTP 헤더를 이용해 자신의 선호 정보를 보낼 수 있다.
  * `Accept`: 서버가 어떤 미디어 타입으로 보내도 되는지 알려준다.
  * `Accept-Language`: 서버가 어떤 언어로 보내도 되는지 알려준다.
  * `Accept-Charset`: 서버가 어떤 차셋으로 보내도 되는지 알려준다.
  * `Accept-Encoding`: 서버가 어떤 인코딩으로 보내도 되는지 알려준다.
* 이 헤더들은 엔터티 헤더(15장 참고)와 비슷하지만 차이가 있다.
  * 엔터티 헤더는 메시지를 서버에서 클라이언트로 전송할 때 필요한 메시지 본문의 속성을 가리킨다.
  * 내용 협상 헤더는 클라이언트와 서버가 선호 정보를 서로 교환하고 문서 버전 중 하나를 선택하는 것을 도와, 클라이언트의 선호에 가장 잘 맞는 문서를 제공해주는 목적으로 사용된다.
* 서버는 클라이언트의 `Accept` 관련 헤더를 엔터티 헤더들과 짝을 지어준다.
  * `Accept` --- `Content-Type`
  * `Accept-Language` --- `Content-Language`
  * `Accept-Charset` --- `Content-Type`
  * `Accept-Encoding` --- `Content-Encoding`
* HTTP는 상태가 없는 프로토콜이라(서버는 클라이언트가 이전 요청에서 보낸 선호 정보를 기억하지 않는다는 뜻), 클라이언트는 자신의 선호 정보를 반드시 매 요청마다 보내야 한다.
* 클라이언트가 자신이 이해할 수 있는 언어를 지정한 `Accept-Language` 헤더 정보를 보내면, 서버는 어떤 사본을 클라이언트에게 돌려줘야할지 판단할 수 있다.
* 서버가 자동으로 돌려보낼 문서를 고르게 되면, 클라이언트 주도 모델에서 협상을 위해 수차례 메시지가 오가며 발생한 커뮤니케이션 대기 시간을 줄여준다.
* 클라이언트가 요청하는 것과 맞는 문서가 없으면, 서버는 추측하거나 클라이언트 주도 모델로 전환하여 클라이언트가 선택하도록 묻는다.
* HTTP는 클라이언트 선호에 대한 풍부한 설명을 품질값(quality value, `q`값)을 이용해 전달할 수 있는 메커니즘을 제공한다.

### 17.3.2 내용 협상 헤더의 품질값

* HTTP 프로토콜은 클라이언트가 각 선호 카테고리마다 여러 선택 가능한 항목을 선호도와 함께 나열할 수 있게 품질값을 정의하였다.
* `q`값은 가장 낮은 선호도인 `0.0`부터 가장 높은 선호도 `1.0`까지 값을 가질 수 있다.
* 예) `Accept-Language: en;q=0.5, fr;q=0.0, nl;q=1.0, tr;q=0.0` : 네덜란드어로된 문서를 받기를 원하나, 영어로 된 문서라도 받아들일 것이며, 어떤 경우에도 터키어나 프랑스어 버전을 원하지 않음을 의미한다.
* 서버가 클라이언트 선호에 대응하는 문서를 갖고 있지 않으면, 서버는 클라이언트 선호에 맞게 문서를 고치거나 트랜스코딩 할 수 있다.

### 17.3.3 그 외의 헤더들에 의해 결정

* 서버는 `User-Agent`와 같은 클라이언트의 다른 요청 헤더를 이용해 알맞은 요청을 만들어내려고 시도할 수 있다.
* 서버가 오래된 버전의 웹브라우저는 자바스크립트를 지원하지 않는다는 것을 알면, 그들에게는 자바스크립트를 포함하지 않은 페이지를 돌려줄 수도 있다.
  * 이 때에는 `q`값 메커니즘이 없다. 서버는 정확한 대응을 찾아내거나 그냥 갖고 있는 것을 제공해야하며, 이는 서버의 구현에 달려있다.
* 캐시는 반드시 캐시된 문서의 올바른 '최선의' 버전을 제공해야 하므로 HTTP 프로토콜은 서버가 응답에 넣어 보낼 수 있는 `Vary` 헤더를 정의한다.
* 캐시나 클라이언트, 그 외 모든 다운스트림 프락시에게 서버가 내어줄 응답의 최선 버전을 결정하기 위해 어떤 요청 헤더를 참고하고 있는지를 `Vary` 헤더를 통해 알려준다.

### 17.3.4 아파치의 내용 협상

* 내용 협상은 웹 사이트 콘텐츠 제공자에게 달려있다. 색인 페이지를 여러 버전으로 제공하려고 한다면, 콘텐츠 제공자가 각각 버전에 해당하는 파일을 아파치 서버의 적절한 디렉터리에 모두 담아야 한다. 그 후 다음 둘 중 한 가지 방법으로 내용 협상을 동작시킬 수 있다.
  * 웹 사이트 디렉터리에서 배리언트(variant)를 갖는 웹 사이트의 각 URL을 위한 `type-map` 파일을 만든다. 그 `type-map` 파일은 모든 배리언트와 그 각각에 대응하는 내용 협상 헤더를 나열한다.
  * 아파치가 그 디렉터리에 대해 자동으로 `type-map` 파일을 생성하도록 하는 `MultiViews` 지시어를 켠다.

#### type-map 파일 사용하기

* 아파치 서버는 `type-map` 파일이 어떻게 생겼는지 알아야 한다. 이 설정을 위해, 서버 설정 파일에 `type-map` 파일을 위한 파일 접미사를 명시한 핸들러를 추가한다.
  * `AddHandler type-map .var`: `.var` 확장자를 가진 파일이 `type-map` 파일임을 의미
  * `type-map` 파일 예
    ```
    URI: abc.html

    URI: abc.en.html
    Content-type: text/html
    Content-language: en

    URI: abc.fr.de.html
    Content-type: text/html;charset=iso-8859-2
    Content-language: fr, de
    ```
    * 아파치 서버는 영어로 요청한 클라이언트에게는 `abc.en.html` 파일을, 프랑스어로 요청한 클라이언트에게는 `abc.fr.de.html` 파일을 보냄을 알 수 있다. 품질값 또한 지원된다.

#### MultiViews 사용하기

* `MultiViews`를 사용하려면 `access.conf` 파일에 적절한 절(`<Directory>`, `<Location>`, 혹은 `<Files>`)에 `Options` 지시어를 이용하여, 웹 사이트를 포함한 디렉터리에 `MultiViews`를 반드시 켜야 한다.
* `MultiViews`가 켜져 있고 브라우저가 `abc` 라는 이름의 리소스를 요청했다면, 서버는 이름에 'abc'가 들어있는 모든 파일을 살펴보고 그에 대한 `type-map` 파일을 생성한다. 이름에 근거하여 서버는 각 파일에 대응하는 적절한 내용 협상 헤더를 추측한다. 예를 들어 abc의 프랑스어 버전은 `.fr`을 포함해야 한다.
### 17.3.5 서버 측 확장

* 서버에서 내용 협상 구현의 또 다른 방법으로, 마이크로소프트의 액티브 서버 페이지(ASP)와 같이 서버 쪽에서 확장을 하는 방법이 있다. (서버 측 확장 개요는 8장을 참고)

## 17.4 투명 협상

* 투명 협상은 클라이언트 입장에서 협상하는 중개자 프락시를 두어 클라이언트와의 메시지 교환을 최소화하고, 서버 주도 협상으로 인한 부하를 서버에서 제거한다.
* 프락시는 클라이언트의 기대가 무엇인지 알고, 클라이언트 입장에서 협상을 수행할 수 있는 능력이 있는 것으로 가정된다.
  * 프락시는 콘텐츠에 대한 요청을 보고 클라이언트의 요구사항을 파악하고 있다.
* 투명한 내용 협상을 지원하기 위해, 서버는 클라이언트의 요청에 가장 잘 맞는 것이 무엇인지 판별하려면 어떤 요청 헤더를 검사해야하는지 프락시에게 말해줘야 한다.
* HTTP/1.1 명세는 투명 협상에 대한 어떤 메커니즘도 정의하지 않지만, 대신 `Vary` 헤더를 정의했다. 서버는 응답에 `Vary` 헤더를 포함시켜 보냄으로써 중개자에게 내용 협상을 위해 어떤 헤더를 사용하는지 알려준다.
* 캐시 프락시는 단일 URL을 통해 접근할 수 있는 문서의 여러 다른 사본을 저장할 수 있다.
* 서버가 그들 캐시에 대한 의사결정 프로세스를 캐시에게 알려주었다면, 캐시는 서버 입장에서 클라이언트와 협상할 수 있다.
* 캐시는 또한 콘텐츠 트랜스코딩에 훌륭한 장소로, 캐시 안에 설치되어 있는 범용 트랜스코더는 특정 서버에 국한되지 않고 어떤 서버의 콘텐츠든 트랜스코딩할 수 있다.

### 17.4.1 캐시와 얼터네이트(alternate)

* 캐시는 클라이언트에게 올바로 캐시된 응답을 돌려주기 위해 서버가 응답을 돌려줄 때 사용한 의사결정 로직의 상당 부분을 그대로 사용해야 한다.
* 캐시는 캐시된 응답을 돌려보낼 때, 클라이언트가 보내는 Accept 관련 헤더와 서버가 각 요청에 최적의 응답을 하기 위해 알맞게 만들어 사용하는 엔터티 헤더와 동일한 헤더를 사용해야 한다.
* 캐시는 첫 번째 요청을 서버에 그대로 전달하고 응답을 저장한다. 두 번째 요청에서 사용자가 다른 언어를 원한다면 저장해둔 첫 번째 응답을 전달할 수 없다. 캐시는 반드시 두 번째 응답도 서버에게 그대로 전달하고 그 URL에 대한 이번 응답과 지난 응답을 모두 저장해야 한다.
  * 서버와 마찬가지로 캐시는 이제 같은 URL에 대해 두 개의 다른 문서를 갖게 된다. 이 다른 버전은 배리언트(variant)나 얼터네이트(alternate)로 불린다.
* 내용 협상은 배리언트 중에서 클라이언트 요청에 가장 잘 맞는 것을 선택하는 과정으로 이해할 수 있다.
### 17.4.2 Vary 헤더

* HTTP `Vary` 응답 헤더는 서버가 문서를 선택하거나 커스텀 콘텐츠를 생성할 때 고려한 클라이언트 요청 헤더 모두(일반적인 내용 협상 헤더 외에 추가로 더해서)를 나열한다.
* 제공된 문서가 `User-Agent` 헤더에 의존한다면, `Vary` 헤더는 반드시 `User-Agent`를 포함해야 한다.
* 새 요청이 도착하면, 캐시는 내용 협상 헤더들을 이용해 가장 잘 맞는 것을 찾는다. 그러나 캐시가 클라이언트에게 문서를 제공하기 전, 캐시는 반드시 캐시된 응답 안에 서버가 보낸 `Vary` 헤더가 있는지 확인해야 한다.
  * 만약 `Vary` 헤더가 존재하면, 그 `Vary` 헤더가 명시하는 헤더들은 새 요청과 오래된 캐시된 요청에서의 그 값이 서로 맞아야 한다. 서버는 클라이언트의 요청 헤더에 따라 그들의 응답이 달라질 수 있기 때문에, 투명 협상을 구현하기 위해 캐시는 반드시 캐시된 배리언트와 함께 클라이언트 요청 헤더와 그에 알맞은 서버 응답 헤더 모두를 저장해야 한다.
  * 서버의 `Vary` 헤더가 `Vary: User-Agent, Cookie`라면 거대한 수의 다른 `User-Agent`와 `Cookie` 값이 많은 배리언트를 만들어 낼 것이다.
  * 캐시는 각 배리언트마다 알맞은 문서 버전을 저장해야 한다. 캐시가 검색을 할 때, 먼저 내용 협상 헤더로 적합한 콘텐츠를 맞춰보고, 다음 요청의 배리언트를 캐시된 배리언트와 맞춰본다. 맞는 것이 없으면 캐시는 서버에서 문서를 가져온다.

## 17.5 트랜스코딩

* 서버가 클라이언트의 요구에 맞는 문서를 아예 갖고 있지 않다면 서버는 에러로 응답해야겠지만, 이론적으로 서버는 기존 문서를 클라이언트가 사용할 수 있는 무언가로 변환할 수도 있다. 이를 트랜스코딩이라고 한다.
* 트랜스코딩에는 포맷 변환, 정보 합성, 내용 주입의 세 종류가 있다.

### 17.5.1 포맷 변환

* 포맷 변환은 데이터를 클라이언트가 볼 수 있도록 한 포맷에서 다른 포맷으로 변환하는 것이다.
  * 예) 접속 속도가 느리고 고해상도 이미지가 필요없는 클라이언트에서 이미지가 많은 페이지를 쉽게 볼 수 있도록 하려면 이미지를 칼라에서 흑백으로 변환하고 축소하여 크기와 해상도를 줄인다.
* 포맷 변환은 내용 협상 헤더에 의해 주도된다. (`User-Agent` 헤더에 의해 주도될 수도 있다)
* 내용 변환 혹은 트랜스코딩은 콘텐츠 인코딩이나 전송 인코딩과는 다르다. 전자는 콘텐츠를 특정 접근 장치에서 볼 수 있도록 하기 위함이고, 후자 둘은 보통 콘텐츠의 더 효율적이고 안전한 전송을 위한 것이다.
### 17.5.2 정보 합성

* 문서에서 정보의 요점을 추출하는 것을 정보 합성(information synthesis)이라고 하며, 이는 트랜스코딩 과정에서 유용하다.
  * 예) 각 절의 제목에 기반한 문서의 개요 생성, 페이지에서 광고 및 로고 제거
* 본문의 키워드에 기반해 페이지를 분류하는 더 복잡한 기술은 문서 핵심을 요약할 때 역시 유용하다. 포털 사이트의 웹페이지 디렉터리처럼 자동화된 웹페이지 분류 시스템에 종종 사용된다.

### 17.5.3 콘텐츠 주입

* 일반적으로 웹 문서의 양을 줄이는 위의 두 트랜스코딩과 다르게, 오히려 양을 늘리는 내용 주입 트랜스코딩(content-injection transcoding)이라는 것도 있다. 자동 광고 생성과 사용자 추적시스템이 그 예다.
* 지나가는 모든 HTML 페이지에 자동으로 광고를 삽입하는 광고 삽입 트랜스코더에서 현재 관련있거나 특정 사용자를 대상으로 하는 광고를 그때그때 효과적으로 삽입하기 위해 동적으로 트랜스코딩이 이루어진다.
* 사용자 추적 시스템 또한 어떻게 페이지가 보여지고 클라이언트가 웹을 돌아다니는지에 대한 통계 수집을 위해 페이지에 동적으로 콘텐츠를 추가할 수 있게 만들어져 있다.

### 17.5.4 트랜스코딩 vs. 정적으로 미리 생성해놓기

* 트랜스코딩의 대안은 웹 서버에서 웹페이지의 여러 사본을 만드는 것이다. 그러나 현실적이지 않다.
* 페이지의 작은 변화로도 여러 페이지를 수정해야 하고, 각 페이지의 모든 버전을 저장하기 위해 더 많은 공간이 필요하며, 페이지를 관리하고 그 중 올바른 것을 골라 제공해주는 웹 서버를 프로그래밍하기 어려워진다. 광고 삽입과 같은 몇몇 트랜스코딩은 정적으로는 수행될 수 없다. 페이지 요청 사용자에 따라 삽입할 광고가 결정되기 때문이다.
* 루트 페이지를 그때그때 필요마다 변경하는 것은 정적으로 미리 생성해 놓는 것 보다 더 쉬운 해결책이지만, 콘텐츠 제공에 있어 대기시간 증가로 인한 비용을 초래할 수 있다. 그러나 더 싼 프락시나 캐시에 있는 외부 에이전트에게 수행하게 하여 웹 서버의 부담을 덜 수 있다.

## 17.6 다음 단계

* 내용 협상에 대한 이야기는 다음 두 가지 이유로 `Accept`나 `Content` 관련 헤더들에서 끝나지 않는다.
  * HTTP의 내용 협상은 성능 제약을 초래한다. 적절한 콘텐츠를 위해 여러 배리언트(variant)를 탐색하는 것이나, 혹은 가장 잘 맞는 것을 추측하려는 것은 비용이 클 수 있다. 간소화되었으면서 내용 협상 프로토콜에 집중하는 방법에 대해 [RFC 2295](https://tools.ietf.org/html/rfc2295)와 [RFC 2296](https://tools.ietf.org/html/rfc2296)은 질문을 던진다.
  * HTTP는 내용 협상이 필요한 유일한 프로토콜이 아니다. 미디어 스트리밍과 팩스는 클라이언트와 서버가 클라이언트의 요청에 대한 최적의 답을 하기 위해 논의해야 할 필요가 있는 두 가지 다른 예다. 일반적인 내용 협상 프로토콜이 TCP/IP 응용 프로토콜 위에서 만들어질 수 있을까? 콘텐츠 협상 작업 그룹은 이 의문 때문에 생겨났고 지금은 닫혔지만 여러 RFC에 기여했다.

## 17.7 추가 정보