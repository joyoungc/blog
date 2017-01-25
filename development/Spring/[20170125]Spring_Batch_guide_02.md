## 2. 개발 표준
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
대상 | 표준
------------ | -------------
Scheduler 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.scheduler.{배치업무명}Scheduler.java
Job 설정 파일 | config/batch/job/{업무카테고리}/job-{배치업무명}.xml
Tasklet 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.tasklet.{명사}Tasklet.java
ItemReader 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemReader.java
ItemWriter 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemWriter.java
ItemProcessor 클래스 | {프로젝트 패키지}.{업무카테고리}.batch.item.{Step아이디}ItemProcessor.java
Job 테스트 클래스 | {프로젝트 패키지}.{업무카테고리}.{Job}Test.java
Job 아이디 | {배치업무명}Job
Step 아이디 | {명사}Step

## 3. 배치작업 정의 절차
스프링 배치에서 제공하는 xml문법에 따라 배치작업을 정의함. (annotation 방식은 차후 포스팅 예정)

#### 배치작업 파일 생성
아래 형식의 xml 파일을 생성한다. (위치 및 파일 이름은 [2.개발표준](## 2. 개발 표준) 참고)
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
