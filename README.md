# java-libs
Java 자주 사용하는 공통 기능 커스텀 라이브러리

## Tutorial
### 1.  프로젝트 하위에 ‘libs’ 디렉토리를 추가

### 2. 추가한 ‘libs’ 라이브러리 하위에 사용할 라이브러리 파일(jar)을 위치

### 3. gradle 설정 파일 dependency에 공통 라이브러리 정보를 추가

- build.gradle.kts
    
    ```kotlin
    dependencies {
    	implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    	...
    }
    ```
    

## 공통 라이브러리 기능
- commons-data-export-*.jar
    - excel/csv export 파일 포맷 정의 라이브러리
    - 버전: 1.0 (23.12.01 기준)
    - util 정보
        - `public static void exportCsv(Writer writer, String[] header, List<List<String>> body) {}`
        - `public static SXSSFWorkbook exportExcel (ExportDataDto exportData, Boolean isNumberData) {}`
    - 사용 방법
        - `exportCsv(Writer writer, String[] header, List<List<String>> body)`
        - 예시
            
            ```java
            @GetMapping(value = "/export/csv", produces = "text/csv")
            public void exportUserCsv(HttpServletResponse response) throws IOException {
            	String[] header = {"아이디", "고객명", "핸드폰 번호", "이메일", "권한명"};
            	List<List<String>> body = usersService.findBody();
            
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/csv; charset=UTF-8");
            
              response.setHeader("Content-disposition", "attachment;filename="
                          + LocalDate.now() + "_"
            			        + URLEncoder.encode("사용자 목록", StandardCharsets.UTF_8).replaceAll("\\+", "%20") + ".csv");
            
               ExportUtil.exportCsv(response.getWriter(), header, body);
            }
            ```
            
            - Writer writer: response 객체의 writer 값 (csv 생성 위해 필요)
            - String[] header: 테이블 헤더 정보 배열
            - List<List<String>> body: 테이블 데이터 정보 리스트
        - `exportExcel(ExportDataDto exportData, Boolean isNumberData)`
        - 예시
            
            ```java
            String sheetName = ExportDataInfo.USER.getSheetName();
            String[] header = ExportDataInfo.USER.getHeader();
            List<List<String>> body = findUsersBody();
            Boolean isNumberData = false;
            
            exportData = new ExportDataDto(sheetName, header, body);
            
            SXSSFWorkbook workbook = ExportUtil.exportExcel(exportData, isNumberData);
            ```
            
            - String sheetName: 시트명
            - String[] header: 테이블 헤더 정보 배열
            - List<List<String>> body: 테이블 데이터 정보 리스트
            - Boolean isNumberData: 숫자 데이터 포함 여부 (천단위 콤마 및 소숫점 자리수 처리 위해)
    
- commons-redis-*.jar
    - redis get/set/delete 함수 라이브러리
    - 버전: 2.0 (23.12.01 기준)
    - 업데이트 내용: redis config 를 포함하여 redis config 를 프로젝트에 추가하지 않아도 됨
    - util 정보
        - `public String getValue(String key) {}`
        - `public void setValue(String key, String value) {}`
        - `public void setValue(String key, String value, Long validityInMilliSeconds) {}`
        - `public Boolean delValue(String key) {}`
    - 사용 방법
        - `getValue(String key)`
            
            ```java
            @RequiredArgsConstructor
            class Test {
            	private final RedisUtil redisUtil;
            	...
            	
            	String userToken = redisUtil.getValue(userUuid);
            
            }
            ```
            
        - `setValue(String key)`
            
            ```java
            @RequiredArgsConstructor
            class Test {
            	private final RedisUtil redisUtil;
            	...
            	
            	redisUtil.setValue(userUuid, "value");
            }
            ```
            
        - `setValue(String key, String value, Long validityInMilliSeconds)`
            
            ```java
            @RequiredArgsConstructor
            class Test {
            	private final RedisUtil redisUtil;
            	...
              Long accessTokenValidityInSeconds = 3600;
            	Long validityInMilliSeconds = accessTokenValidityInSeconds * 1000;
            	redisUtil.setValue(userUuid, "value", validityInMilliSeconds);
            }
            ```
            
        - `delValue(String key)`
            
            ```java
            @RequiredArgsConstructor
            class Test {
            	private final RedisUtil redisUtil;
            	...
              Boolean isDelete = redisUtil.delValue(userUuid);
            }
            ```
            
- commons-paging-*.jar
    - `Pageable` 및 `Pagination` 등 페이징 데이터 포맷 정의 라이브러리
    - 버전: 1.0 (23.12.01 기준)
    - util 정보
        - `public static Pageable toPageable(int page, int limit) {}`
        - `public static Pageable toPageable(int page, int limit, int add) {}`
        - `public static Pagination toPagination(int page, int limit) {}`
        - `public static Pagination toPagination(int page, int limit, int add) {}`
    - 사용 방법
        - Spring Data JPA + Pageable
            - `toPageable(int page, int limit)`
                
                ```java
                Pageable paging = PagingUtil.toPageable(0, 15);
                ```
                
            - `toPageable(int page, int limit, int add)`
                
                ```java
                Pageable paging = PagingUtil.toPageable(0, 15, 2);
                ```
                
        - QueryDsl + Pagination
            - `toPagination(int page, int limit)`
                
                ```java
                Pagination paging = PagingUtil.toPagination(0, 15);
                ```
                
            - `toPageable(int page, int limit, int add)`
                
                ```java
                Pagination paging = PagingUtil.toPagination(0, 15, 2);
                ```
                
- commons-response-*.jar
    - Response 데이터 포맷 정의 라이브러리
    - 버전: 1.0 (23.12.01 기준)
    - util 정보
        - `public static ResponseEntity SUCCESS () {}`
        - `public static <T>ResponseEntity SUCCESS(T data) {}`
        - `public static <T>ResponseEntity ERROR (int code, T data) {}`
        - `public static <T>ResponseEntity ERROR (int code, T data, String message) {}`
    - 사용 방법
        - `SUCCESS () {}`
            
            ```java
            // Controller return 값이 없는 경우 사용
            return ResponseUtil.SUCCESS();
            ```
            
        - `SUCCESS(T data) {}`
            
            ```java
            // Controller return 값 있는 경우
            DataResponseDto responseData = dataService.saveJob(jobUnitCd, executorId);
            
            return ResponseUtil.SUCCESS(responseData);
            
            ```
            
        - `ERROR (int code, T data) {}`
            
            ```java
            // Custom Exception 인 경우
            try {}
            catch (DataException e) {
            	return ResponseUtil.ERROR(HttpServletResponse.SC_BAD_REQUEST, new ErrorResponseDto(e.getErrorCode()));
            }
            
            ```
            
        - `ERROR (int code, T data, String message) {}`
            
            ```java
            // System Exception 인 경우
            try {} 
            catch (RuntimeException e) {
                return ResponseUtil.ERROR(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, new ErrorResponseDto(ErrorCode.SERVER_ERROR), e.getMessage());
            }
            
            ```


