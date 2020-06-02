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


# Database Reader
Spring 프레임워크의 강점 중 하나는 개발자가 비즈니스 로직에만 집중할 수 있도록 JDBC와 같은 문제점을 추상화한 것입니다.
> 이를 보고 서비스 추상화라고 합니다. <br/>
>
그래서 Spring Batch 개발자들은 Spring 프레임워크의 JDBC 기능을 확장했습니다.<br/>
일반적으로 배치 작업은 많은 양의 데이터를 처리해야 합니다.

> 보통 실시간 처리가 어려운 대용량 데이터나 대규모 데이터일 경우에 배치 어플리케이션을 작업합니다.
>
수백만개의 데이터를 조회하는 쿼리가 있는 경우에 해당 데이터를 모두 한번에 메모리에 불러오길 원하는 개발자는 없을 것입니다.<br/>
그러나 Spring의 JdbcTemplate은 분할 처리를 지원하지 않기 때문에 (쿼리 결과를 그대로 반환하니) 개발자가 직접 limit, offset을 사용하는 등의 작업이 필요합니다.<br/>
Spring Batch는 이런 문제점을 해결하기 위해 2개의 Reader 타입을 지원합니다. <br/>
Cursor는 실제로 JDBC ResultSet의 기본 기능입니다.<br/>
ResultSet이 open 될 때마다 next()메소드가 호출 되어 Database의 데이터가 반환 됩니다.<br/>
이를 통해 필요에 따라 Database에서 데이터를 Streaming할 수 있습니다.<br/>

반면 Paging은 좀 더 많은 작업을 필요로 합니다.<br/>
Paging개념은 페이지라는 Chunk로 Database에서 데이터를 검색한다는 것입니다.<br/>
즉, 페이지 단위로 한번에 데이터를 조회해오는 방식입니다.<br/>

Cursor 방식은 Database와 커넥션을 맺은 후, Cursor를 한칸씩 옮기면서 지속적으로 데이터를 빨아옵니다.<br/>
반면 Paging 방식에는 한번에 10개(혹은 개발자가 지정한 PageSize)만큼 데이터를 가져옵니다.<br/>

2개 방식의 구현체는 다음과 같습니다.
+ Cursor 기반 ItemReader 구현체
    + JdbcCursorItemReader
    + HibernateCursorItemReader
    + StoredProcedureItemReader
    
+ Paging 기반 ItemReader 구현체
    + JdbcPagingItemReader
    + HibernatePagingItemReader
    + JpaPagingItemReader
    
  

# CursorItemReader

CursorItemReader는 Paging과 다르게 Streaming으로 데이터를 처리합니다.<br/>
쉽게 생각하시면 Database와 어플리케이션 사이에 통로를 하나 연결하고 하나씩 빨아들인다고 생각하시면 됩니다.<br/>
JSP나 Servlet으로 게시판을 작성해보신 분들은 ResultSet을 사용해서 next()로 하나씩 데이터를 가져왔던 것을 기억하시면 됩니다.



