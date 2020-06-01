# ItemReader
Spring Batch의 ItemReader는 데이터를 읽어들입니다.<br/>
DB 데이터뿐만 아니라 File, XML, JSON 등 다른 데이터 소스를 배치 처리의 입력으로 사용할 수 있습니다.<br/>
<br/>
Spring Batch에서 지원하지 않는 Reader가 필요한 경우 직접 해당 Reader를 만들수도 있습니다.<br/>
Spring Batch는 이를 위해 Custom Reader 구현체를 만들기 쉽게 제공하고 있습니다.<br/>
<br/>

Spring Batch의 Reader에서 읽어올 수 있는 데이터 유형은 다음과 같습니다.
+ 입력 데이터에서 읽어오기
+ 파일에서 읽어오기
+ Database에서 읽어오기
+ Java Message Service등 다른 소스에서 읽어오기
+ 본인만의 커스텀한 Reader로 읽어오기

ItemReader의 구현체들이 어떻게 되어있는지 살펴보면<br/>
JdbcPagingItemReader가 있습니다.<br/>
해당 클래스의 계층 구조를 살펴보면 다음과 같습니다.<br/>

![JdbcPagingItemReader](../image/JdbcPagingItemReader.PNG)

ItemReader외에 ItemStream 인터페이스도 같이 구현되어 있습니다. <br/>
먼저 ItemReader를 살펴보면 read()만 가지고 있습니다.
+ read()의 경우 데이터를 읽어오는 메소드입니다.

Reader가 하는 본연의 임무를 담당하는 인터페이스임을 알 수 있습니다.<br/>
그럼 ItemStream 인터페이스는?<br/>
ItemStream 인터페이스는 주기적으로 상태를 저장하고 오류가 발생하면 해당 상태에서 복원하기 위한 마커 인터페이스입니다.<br/>
즉, 배치 프로세스의 실행 컨텍스트와 연계해서 ItemReader의 상태를 저장하고 실패한 곳에서 다시 실행할 수 있게 해주는 역할을 합니다.<br/>
<br/>

ItemStream의 3개의 메소드는 다음과 같은 역할을 합니다.
+ open(), close()는 스트림을 열고 닫습니다.
+ update()를 사용하면 Batch의 처리의 상태를 업데이트 할 수 있습니다.

개발자는 ItemReader와 ItemStream 인터페이스를 직접 구현해서 원하는 형태의 ItemReader를 만들 수 있습니다. <br/>
다만 Spring Batch에서 대부분의 데이털 형태는 ItemReader로 이미 제공하고 있기 때문에 커스텀한 ItemReader를 구현할 일은 많이 없을 것입니다.

