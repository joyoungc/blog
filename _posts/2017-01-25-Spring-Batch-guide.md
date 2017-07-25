---
layout: post
title:  "스프링 배치 개발가이드"
date:   2017-01-25 14:00:00
categories: spring
tags: spring batch java
author: Joyoungc
---

# 스프링 배치 개발가이드
## 1. 개요 및 스피링 배치 설명

### 스프링 배치
대용량 배치처리에 필요한 구조와 기능들을 제공하는 프로젝트로 배치업무에 특화된 프로그래밍 모델을 제공함

### 주요기능
- 프로그래밍 모델(Chunk based processing)
- 실행관리 (트랜잭션, 시작/중단/재시작, 재시도/건너뛰기)
- 관리기능 (로깅/추적, 통계, 모니터링)


![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/spring-batch-reference-model_20170118.png)

### 주요개념
- #### Job
  - 하나의 배치작업을 정의하는 개념
  - Step의 집합
![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/job-heirarchy_20170118.png)


- #### Job Instance
  - Job의 실행에 대한 논리적인 개념
  - JobInstance는 다수의 JobExecution을 가진다.
  - 동일한 JobParameters를 가진 JobInstance는 생성불가

- #### Job Execution
  - Job Instance에 대한 한번의 실행시도를 의미
  - JobExecution은 Job에 대한 시도 결과(FAILED,COMPLETED)를 가진다.

- #### Job Parameters
  - Job을 시작하는데 사용되는 파라미터의 집합 (Map 형식)
  - JobInstance = Job + JobParameters
![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/job-stereotypes-parameters_20170118.png)
  
- #### Step
  - 배치 작업(Job)에 대한 순차적인 처리 단계를 의미함
  - 단순하거나 복잡(병렬,파티션,플로우 방식 등)하게도 구성할 수 있다.
![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/jobHeirarchyWithSteps_20170118.png)
  
  
- #### Step Execution
  - 한번의 Step 실행 시도를 의미함
  - Step이 실행될 때, 새로운 StepExecution이 생성된다.
  
- #### Tasklet
  - Step에서 실행되는 작업을 의미함
  - Chunk Oriented Processing Tasklet을 기본으로 제공
  - Tasklet을 별도로 구현하여 사용하는 것이 가능
  
- #### Chunk-Oriented Processing
Data를 한번에 하나씩 읽고 처리하며 트랜잭션 범위 내에서 'Chunk'를 만든 후 한번에 쓰는 방식이다. 

![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/chunk-oriented-processing_20170118.png)
> Chunk 단위는 트랜잭션 Commit 단위

- #### Item Reader
  - Step의 처리대상이 되는 데이터를 조회하는 역할
  
- #### Item Writer
  - Step의 처리결과를 기록(저장)하는 역할
  
- #### Item Processor
  - Step의 ItemReader를 통해서 전달된 하나의 아이템에 대한 비즈니스로직을 처리하는 역할 

- #### Job Launcher
  - JobParameters를 이용하여 Job을 시작함
  - 실행 결과로 JobRepository와 실행 중인 Job에서 JobExecution 정보를 제공함
  
- #### Job Repository
  - Job의 실행정보(JobInstance, JobExecution, JobExecutionParams, JobExecutionContext, StepExecution, StepExecutionContext)에 대한 CRUD 오퍼레이션 처리를 담당
  
- #### Execution Context
  - Step, Job Execution 구동 중 발생하는 정보를 저장하는 컬렉션
  - Job 재시작시에 사용되는 정보를 담는다. 
  - Json 형태로 저장이 되면 최대 2500 byte를 저장하므로 단순한 값만 저장한다.   

## <a name="springbatch2"></a>2. 개발 표준
### 배치 프로젝트 구조
```
├── src/main/java
│   └── {업무레벨1}.{업무레벨2}
│       ├── batch
│       │   ├── tasklet         * tasklet 패키지
│       │   ├── scheduler       * scheduler 패키지    
│       │   └── item            * ItemReader, Writer, Processor 패키지
│       ├── service             * Service 인터페이스 패키지
│       ├── dao                 * Dao 인터페이스 패키지
│       └── model               * Model 클래스 패키지
├── src/main/resources
│   ├── config
│   │   ├── log                 * Logback 설정파일 폴더
│   │   ├── spring              * Spring 설정파일 폴더
│   │   └── batch
│   │       └── job             * Job 설정파일 폴더
│   ├── message                 * 메시지 프로퍼티 파일폴더
│   └── sql                     * Mybatis 설정 파일 및 SqlMapper.xml 폴더
└── src/test/java
    └── {업무레벨1}.{업무레벨2}    * Job 테스트 클래스 패키지
```

### 명명 규칙

| 대상 | 표준 |
| ----- | ----- |
| Scheduler 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.scheduler.{배치업무명}Scheduler.java |
| Job 설정 파일 | config/batch/job/{업무카테고리}/job-{배치업무명}.xml |
| Tasklet 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.tasklet.{명사}Tasklet.java |
| ItemReader 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemReader.java |
| ItemWriter 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemWriter.java |
| ItemProcessor 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemProcessor.java |
| Job 테스트 클래스 | {프로젝트 패키지}.{업무카테고리}.{Job}Test.java |
| Job 아이디 | {배치업무명}Job |
| Step 아이디 | {명사}Step |



## 3. 배치작업 정의 절차
스프링 배치에서 제공하는 xml문법에 따라 배치작업을 정의함. (annotation 방식은 차후 포스팅 예정)

### 배치작업 파일 생성
아래 형식의 xml 파일을 생성한다. (위치 및 파일 이름은 [2.개발표준](#springbatch2) 참고)

```xml
<beans:beans xmlns="http://www.springframework.org/schema/batch"
     xmlns:beans="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/batch
           http://www.springframework.org/schema/batch/spring-batch-2.2.xsd">
           
</beans:beans>
```

### Job 정의
Job에 대한 아이디와 Step 처리 흐름을 정의한다. 

### Step 흐름 유형
- Sequential Flow : Step을 정해진 순서대로 실행함

![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/sequential-flow_20170206.png)

```xml
<job id="job">
    <step id="stepA" parent="s1" next="stepB" />
    <step id="stepB" parent="s2" next="stepC"/>
    <step id="stepC" parent="s3" />
</job>
```

- Conditional Flow : StepExecution 내의 ExitStatus 값을 이용하여 흐름을 분기 실행함
![http://docs.spring.io/spring-batch/trunk/reference/html/domain.html]({{ site.url }}/images/Conditional-Flow_20170206.png)

```xml
<job id="job">
    <step id="stepA" parent="s1">
        <next on="*" to="stepB" />
        <next on="FAILED" to="stepC" />
    </step>
    <step id="stepB" parent="s2" next="stepC" />
    <step id="stepC" parent="s3" />
</job>
```

- Split Flow : 흐름을 분할하고 병행 실행해야 하는 흐름

```xml
<split id="split1" next="step4">
    <flow>
        <step id="step1" parent="s1" next="step2"/>
        <step id="step2" parent="s2"/>
    </flow>
    <flow>
        <step id="step3" parent="s3"/>
    </flow>
</split>
<step id="step4" parent="s4"/>
```

>(1) Job 정의 
 - id : Job아이디
 - restartable : Job에 대해서 재시작 가능 여부(default : true) 
>(2) Step 정의
 - id : Step의 아이디
 - next : 다음에 실행되어야 하는 Step의 아이디 (Sequential Flow인 경우에 한하여 설정함) 
>(3) ExitStatus 값에 따라서 분기 실행되어야 하는 Step 정의 
 - on : ExitStatus 값
 - to : 다음에 실행되어야 하는 Step 
>(4) Job 실행 중지 정의 (ExitStatus에 따라서 결정)
 - on : ExitStatus 값
 - restart : Job을 재시작하는 경우 실행되는 Step
>(5) Job 실행 완료 정의 (ExitStatus에 따라서 결정)
 - on : ExitStatus값 
>(6) Job 실행 실패 (ExitStatus에 따라서 결정)
 - on : ExitStatus값
 
 
## 4. Step 정의 절차
Step의 Tasklet 종류와 설정

### Chunk Tasklet
기본적인 형태의 Tasklet으로 Chunk Oriented Processing 방식에 기반한 Tasklet

```xml
<job id="sampleJob">
    <step id="step1">
        <tasklet>
            <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
        </tasklet>
    </step>
</job>
```
>(1) ItemReader (필수)
>(2) ItemProcessor (옵션)
>(3) ItemWriter (필수)
>(4) commit-interval - transaction commit 크기

### 개발자 정의 Tasklet
개발자의 필요에 의해서 만들어진 Tasklet을 실행하도록 설정함

```xml
<step id="step1">
    <tasklet ref="myTasklet"/>
</step>
```
> (1) Spring bean으로 등록된 개발자 정의 Tasklet의 bean id를 설정한다.


## 5. 배치 작업 실행 클래스 개발
Step 내에서 실행되는 ItemReader, ItemWriter, ItemProcessor와 CustomTasklet 개발방법 설명

### ItemReader
처리대상의 데이터를 한건씩 리턴하여 다 소모될때까지 수행되는 클래스 

```java
@Component  <-- (1)
@Scope("step")  <-- (2)
public class SampleItemReader implements ItemStreamReader<CodeGroup> <-- (3) {
    @Override
    public CodeGroup read() throws Exception, UnexpectedInputException, ParseException, 
    NonTransientResourceException { <-- (4)
        return new CodeGroup();
    }
    void open(ExecutionContext executionContext) throws ItemStreamException {}
    void update(ExecutionContext executionContext) throws ItemStreamException {}
    void close() throws ItemStreamException {}    
}
```

>(1) 스프링 빈으로 등록하기 위해 설정

>(2) 스프링 빈의 범위를 step으로 설정

>(3) org.springframework.batch.item.ItemStreamReader&lt;T&gt; 인터페이스를 구현

>(4) read() 메서드 구현, 처리대상 데이터를 한건씩 조회하여 리턴함.

>주의사항 : 병렬처리 대상이 되는 경우, Thread Safe에 대한 구현이 필요

>※ 스프링배치에서 미리 구현된 다양한 ItemReader 구현체가 존재함. (http://docs.spring.io/spring-batch/trunk/reference/html/listOfReadersAndWriters.html)


### ItemWriter
ItemWriter는 대상 타입에 관계없이 한번에 항목의 묶음(Chunk)을 쓰는 행위에 대한 클래스

```java
@Component  <-- (1)
@Scope("step")  <-- (2)
public class SampleItemWriter implements ItemStreamWriter<ResultCodeGroup> <-- (3) {
    @Override
    public void write(List<? extends ResultCodeGroup> items) throws Exception { <-- (4)
        // 파라미터로 넘어온 결과를 저장하는 로직
    }
    void open(ExecutionContext executionContext) throws ItemStreamException {} <-- (5)
    void update(ExecutionContext executionContext) throws ItemStreamException {} <-- (6)
    void close() throws ItemStreamException {}   <-- (7) 
}
```
>(1) 스프링 빈으로 등록하기 위해 설정
>(2) 스프링 빈의 범위를 step으로 설정
>(3) org.springframework.batch.item.ItemStreamWriter&lt;T&gt; 인터페이스를 구현
>(4) write() 메서드 구현, 파라미터로 넘어온 결과를 저장함
>(5) ItemWriter가 처음 동작할때 설정
>(6) ItemWriter가 write 메서드가 실행된 이후 실행
>(7) ItemWriter가 종료될때 실행
>※ 스프링배치에서 미리 구현된 다양한 ItemWriter 구현체가 존재함. (http://docs.spring.io/spring-batch/trunk/reference/html/listOfReadersAndWriters.html#itemWritersAppendix)


### ItemProcessor
ItemReader가 리턴한 한 건의 데이터에 대해서 비즈니스 로직을 실행하여 그 결과를 리턴하는 클래스

```java
@Component  <-- (1)
@Scope("step")  <-- (2)
public class SampleItemProcessor implements ItemProcessor<CodeGroup, ResultCodeGroup> <-- (3) {
    @Override
    public ResultCodeGroup process(CodeGroup item) throws Exception { <-- (4)
        // 파라미터로 넘어온 결과를 처리하는 로직
        return item;
    }
}
```
>(1) 스프링 빈으로 등록하기 위해 설정
>(2) 스프링 빈의 범위를 step으로 설정
>(3) org.springframework.batch.item.ItemProcessor&lt;T&gt; 인터페이스를 구현
>(4) process 메서드 구현, 파라미터로 넘어온 값을 처리하여 리턴


### ItemStream
재시작 가능한 Job을 구성하기 위해서 필요한 메서드의 인터페이스, 재시작에 필요한 정보를 ExecutionContext에 넣기 위해서 선언됨.
ItemReader, ItemWriter에 함께 구현하여 ItemReader, ItemWirter의 구현클래스를 만든다. 

```java
public interface ItemStream {
    void open(ExecutionContext executionContext) throws ItemStreamException {}  <-- (1)
    void update(ExecutionContext executionContext) throws ItemStreamException {}    <-- (2)
    void close() throws ItemStreamException {}  <-- (3) 
}
```
>(1) ItemReader, ItemWriter가 생성된 이후 실행됨
>(2) ItemReader.read(), ItemWriter.wirte() 메서드가 실행된 이후 실행됨
>(3) ItemReader, ItemWriter의 모든 실행이 완료된 이후에 실행됨


### 개발자정의 Tasklet
ItemReader가 리턴한 한 건의 데이터에 대해서 비즈니스 로직을 실행하여 그 결과를 리턴하는 클래스

```java
@Component  <-- (1)
@Scope("step")  <-- (2)
public class CustomTasklet implements Tasklet {  <-- (3)
    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception { <-- (4)
        // 파라미터로 넘어온 결과를 처리하는 로직
        return RepeatStatus.FINISHED;
    }
}
```
>(1) 스프링 빈으로 등록하기 위해 설정
>(2) 스프링 빈의 범위를 step으로 설정
>(3) org.springframework.batch.core.step.tasklet.Tasklet 인터페이스를 구현
>(4) execute메서드를 구현, 비즈니스 로직을 실행하는 코드를 작성, 리턴값으로 RepeatStatus.FINISHED를 사용해야 함


## 6. 테스트
효과적인 배치의 개발, 테스트를 위하여 JUnit 기반의 테스트 클래스를 추가

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
    "classpath:/test/spring/test-context.xml",
    "classpath:/config/batch/sample/job-sample.xml" <-- (1)
    })
public class SampleTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    public void launchJob() throws Exception {
    
        JobParametersBuilder builder = new JobParametersBuilder(); <-- (2)
		
		builder.addString("startDate", "2016-09-01 00:00");
		builder.addString("endDate", "2016-09-19 19:00");
		builder.addLong("currentTime", System.currentTimeMillis());

        JobExecution jobExecution = jobLauncherTestUtils.launchJob(builder.toJobParameters());  <-- (3)

       //JobExecution jobExecution = jobLauncherTestUtils.launchStep("step1",builder.toJobParameters()); <-- (4)

        assertEquals(BatchStatus.COMPLETED, jobExecution.getStatus());   <-- (5)

    }
}
```
>(1) 테스트 대상 Job 설정파일을 추가해야 함
>(2) Job 실행시 필요한 파라미터들을 세팅
>(3) Job을 실행
>(4) step별로 테스트 필요시 실행
>(5) Job이 정상적으로 종료되었는지 확인


## 7. 스프링 배치 테이블 정보
영속성을 가져야 하는 실행정보들을 저장하는 테이블

![http://docs.spring.io/spring-batch/trunk/reference/html/metaDataSchema.html]({{ site.url }}/images/meta-data-erd_20170207.png)
>※ 테이블 상세정보 : http://docs.spring.io/spring-batch/trunk/reference/html/metaDataSchema.html

