# 배치란 무엇일까?
배치(Batch)는 **일괄처리**란 뜻을 가지고 있다.

### 배치 어플리케이션의 조건
+ 대용량 데이터 - 배치 어플리케이션은 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 있어야 합니다.
+ 자동화 - 배치 어플리케이션은 심각한 문제 해결을 제외하고는 **사용자 개입 없이 실행**되어야 합니다.
+ 견고성 - 배치 어플리케이션은 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 한다.
+ 신뢰성 - 배치 어플리케이션은 무엇이 잘못되었는지를 추적할 수 있어야 한다. (로깅, 알림)
+ 성능 - 배치 어플리케이션은 **지정한 시간 안에 처리를 완료**하거나 동시에 실행되는 **다른 어플리케이션을 방해하지 않도록 수행**되어야합니다.

## Batch vs Quartz?
**Quartz는 스케줄러의 역할**, Batch와 같이 **대용량 데이터 배치 처리에 대한 기능을 지원하지 않는다.**<br/>
Batch 역시 Quartz의 다양한 스케줄 기능을 지원하지 않아서 Quartz + Batch를 조합해서 사용한다.
<br/>
정해진 스케줄마다 Quartz가 Spring Batch를 실행하는 구조

```java
@Slf4j // log 사용을 위한 lombok 어노테이션
@RequiredArgsConstructor // 생성자 DI를 위한 lombok 어노테이션
@Configuration
public class SimpleJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory; // 생성자 DI 받음
    private final StepBuilderFactory stepBuilderFactory; // 생성자 DI 받음

    @Bean
    public Job simpleJob() {
        return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1())
                .build();
    }

    @Bean
    public Step simpleStep1() {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step1");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

+ @Configuration
    + Spring Batch의 모든 Job은 @Configuration으로 등록해서 사용
    
+ JobBuilderFactory.get("simpleJob")
    + simpleJob이란 이름의 Batch Job을 생성
    + job의 이름은 별도로 지정하지 않고, Builder를 통해 지정
 
+ stepBuilderFactory.get("simpleStep1")
    + simpleStep1이란 이름의 Batch Step을 생성
    + jobBuilderFactory.get("simpleJob")와 마찬가지로 Builder를 통해 이름을 지정
    
+ .tasklet((contribution, chunkContext))
    + Step안에서 수행될 기능들을 명시
    + Tasklet은 **Step안에서 단일로 수행될 커스텀한 기능**들을 선언할때 사용
    + 여기서는 Batch가 수행되면 log.info(">>>>> This is Step1")가 출력되도록 합니다.

 
Spring Batch에서 **Job은 하나의 배치 작업 단위** <br>
Job안에는 여러 Step이 존재, Step 안에 Tasklet 혹은 Reader & Processor & Writer 묶음이 존재 <br>
**Tasklet 하나와 Reader & Processor & Writer 한 묶음이 같은 레벨**

