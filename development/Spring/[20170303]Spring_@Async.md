# Spring 비동기 Service 구현 방법

## @Async

- 도입하게 된 이유 : 10만건 이상의 엑셀 다운로드를 진행해야 하는데 
어드민 페이지에서 엑셀 다운로드 시도시 request에 대한 응답이 1분이 넘어가면 502 bad gateway가 발생
- 처리 방법 : 응답은 먼저 알려주고 특정 Service에서 엑셀 파일 생성 후 파일 생성이 완료되면 다운로드 버튼이 활성화 되게 한다. 

### 지켜야할 내용
- public 메소드에만 적용
- self invocation (같은 클래스안에서 async 메소드를 호출)은 동작하지 않음
- 비동기 설정을 해줘야 함 
  1. annotation : @EnableAsync
  2. xml 설정 : task 네임스페이스를 이용
```xml
	<task:annotation-driven executor="taskExecutor" />		 
	<task:executor id="taskExecutor" pool-size="10" />
```

### 주의할 점

- @Async가 적용된 메소드에서는 Http Session은 공유해서 사용할 수 없다. Thread가 분리되기 때문 (관련 포스트 링크)


### 예제

#### StatisticsDownloadServiceImpl.java
```java
/***
 * async 메소드 실행
 */
@Override
public String selectStatisticsDataExcel(RequestStatistics requestStatistics, 
Pageable pageable, HttpServletResponse response) {

	logger.info("===== Start Excel Data [ selectStatisticsDataExcel ] ");
		
	String fileName = "data_" + DateTime.now().toString("yyyyMMddHHmmssSSS") + ".xlsx";
		
	Future<String> result = asyncService.asyncMakeBaseDataExcel(requestStatistics, pageable, fileName);
	httpSession.setAttribute(requestStatistics.getDownloadSequenceNumber(), result);
		
	return fileName;		
}
```
#### AsyncService.java
```java
@Async
public Future<String> asyncMakeBaseRawDataExcel(RequestStatistics requestStatistics,
 Pageable pageable, String fileName) {

	SXSSFWorkbook workbook = new SXSSFWorkbook(100);

	/*........ 엑셀파일 생성 ........*/
		
	logger.info("===== End Excel Data [ getStatisticsBaseDataExcel ] ");
	String result = "COMPLETE";
		
	// 엑셀 파일 생성이 완료되면 Future 객체를 이용하여 성공여부 return
	return new AsyncResult<String>(result);	
}
```    