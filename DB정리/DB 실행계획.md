# 😮 DB 실행계획(MySQL)

### DB 튜닝에서 가장 중요한 것은 쿼리와 스키마 최적회이다. 스키마 설계는 한번 진행되면 그 테이블을 사용하는 모든 쿼리에 영향을 주기 때문에 변경하기는 힘들다. 하지만 쿼리는 수정만 하면 되므로 변경하기 쉽다. **Slow Query**를 없애는 것은 성능을 향상 시키기 위한 매우 중요한 수단이다.

성능 진단의 가장 첫걸음은 실행한 SQL이 DB에서 어떻게 처리되는 지를 파악하는 것이다. 
1. SQL문을 실행하도록 DB에 명령
2. DB는 내부적으로 SQL을 파싱 (문법 체크 및 DB에서 실행하기 위한 형태로 변환)
3. 옵티마이징 (데이터를 찾는 가장 빠른 방법을 찾아내는 단계)을 거친 후 데이터를 찾는다.

## MySQL Explain
- MySQL Explain이란 DB가 데이터를 찾아가는 일련의 과정을 사람이 알아 보기 쉽게 DB 결과 셋으로 보여주는 것  
- MySQL Explain을 활용하여 기존의 쿼리를 튜닝할 수 있을 뿐만 아니라 성능 분석, 인덱스 전략 수립등과 같이 성능 최적화에 대한 전반적인 업무를 처리할 수 있다.

### 사용 방법
```sql
EXPLAIN [EXTENDED] SELECT ... FROM ... WHERE ...
```

## MySQL EXPLAIN 결과의 각 항목 별 의미
|구분|설명|
|:---|:---|
id|select 아이디로 SELECT를 구분하는 번호|
|table|참조하는 테이블|
|select_type|select에 대한 타입|
|type|조인 혹은 조회 타입|
|possible_keys|데이터를 조회할 때 DB에서 사용할 수 있는 인덱스 리스트|
|key|실제로 사용할 인덱스|
|key_len|실제로 사용할 인덱스의 길이|
|ref|Key안의 인덱스와 비교하는 컬럼(상수)|
|rows|쿼리 실행 시 조사하는 행 수립|
|extra|추가 정보|

### id
행이 어떤 SELECT 구문을 나타내는 지를 알려주는 것으로 구문에 서브 쿼리나 UNION이 없다면 SELECT는 하나밖에 없기 때문에 모든 행에 대해 1이란 값이 부여되지만 이외의 경우에는 원 구문에서 순서에 따라 각 SELECT 구문들에 **순차적**으로 번호가 부여된다.

### table
행이 어떤 테이블에 접근하는 지를 보여주는 것으로 대부분의 경우 테이블 이름이나 SQL에서 지정된 별명 같은 값을 나타낸다.

### select_type
|구분|설명|
|:---|:---|
|SIMPLE|단순 SELECT (Union 이나 Sub Query 가 없는 SELECT 문)|
|PRIMARY|Sub Query를 사용할 경우 Sub Query의 외부에 있는 쿼리(첫번째 쿼리) UNION 을 사용할 경우 UNION의 첫 번째 SELECT 쿼리|
|UNION|UNION 쿼리에서 Primary를 제외한 나머지 SELECT|
|DEPENDENT_UNION|UNION 과 동일하나, 외부쿼리에 의존적임 (값을 공급 받음)|
|UNION_RESULT|UNION 쿼리의 결과물|
|SUBQUERY|Sub Query 또는 Sub Query를 구성하는 여러 쿼리 중 첫 번째 SELECT문|
|DEPENDENT_SUBQUERY|Sub Query 와 동일하나, 외곽쿼리에 의존적임 (값을 공급 받음)|
|DERIVED|SELECT로 추출된 테이블 (FROM 절 에서의 서브쿼리 또는 Inline View)|
|UNCACHEABLE SUBQUERY|Sub Query와 동일하지만 공급되는 모든 값에 대해 Sub Query를 재처리. 외부쿼리에서 공급되는 값이 동이라더라도 Cache된 결과를 사용할 수 없음|
|UNCACHEABLE UNION|UNION 과 동일하지만 공급되는 모든 값에 대하여 UNION 쿼리를 재처리|

### type
|구분|설명|
|:---|:---|
|system|테이블에 단 한개의 데이터만 있는 경우|
|const|쿼리의 일부를 상수로 대체시킬 수 있을 때 사용한다. const 테이블은 한번 밖에 읽혀지지 않기 때문에 빠르다. SELECT에서 Primary Key 혹은 Unique Key를 상수로 비교할 때 사용|
|eg_ref|조인을 할 때 Primary Key|
|ref|조인을 할 때 Primary Key 혹은 Unique Key가 아닌 Key로 매칭하는 경우|
|ref_or_null|ref 와 같지만 null 이 추가되어 검색되는 경우|
|index_merge|두 개의 인덱스가 병합되어 검색이 이루어지는 경우|
|unique_subquery|다음과 같이 IN 절 안의 서브쿼리에서 Primary Key가 오는 특수한 경우
SELECT *
FROM tab01
WHERE col01 IN (SELECT Primary Key FROM tab01);|
|index_subquery|unique_subquery와 비슷하나 Primary Key가 아닌 인덱스인 경우
SELECT *
FROM tab01
WHERE col01 IN (SELECT key01 FROM tab02);|
|range|특정 범위 내에서 인덱스를 사용하여 원하는 데이터를 추출하는 경우로, 데이터가 방대하지 않다면 단순 SELECT 에서는 나쁘지 않음|
|index|인덱스를 처음부터 끝까지 찾아서 검색하는 경우로, 일반적으로 인덱스 풀스캔이라고 함|
|all|테이블을 처음부터 끝까지 검색하는 경우로, 일반적으로 테이블 풀스캔이라고 함|

### possible_keys
쿼리에서 접근하는 컬럼 들과 사용된 비교 연산자들을 바탕으로 어떤 인덱스를 사용할 수 있는 지를 표시해준다.

### key
테이블에 접근하는 방법을 최적화 하기 위해 어떤 인덱스를 사용하기로 결정했는 지를 나타낸다.

### key_len
MySQL이 인덱스에 얼마나 많은 바이트를 사용하고 있는 지를 보여준다. MySQL에서 인덱스에 있는 컬럼들 중 일부만 사용한다면 이 값을 통해 어떤 컬럼들이 사용되는 지를 계산할 수 있다.

### ref
키 컬럼에 나와 있는 인덱스에서 값을 찾기 위해 선행 테이블의 어떤 컬럼이 사용되었는 지를 나타낸다.

### rows
원하는 행을 찾기 위해 얼마나 많은 행을 읽어야 할 지에 대한 예측값을 의미한다.
  
### extra
Extra 필드는 옵티마이저가 동작하는데 대해서 우리에게 알려주는 힌트다. 이 필드는 EXPLAN을 사용해 옵티마이저의 행동을 파악할때 아주 중요하다.
|구분|설명|
|:---|:---|
|Using where|접근 방식을 설명한 것으로, 테이블에서 행을 가져온 후 추가적으로 검색조건을 적용해 행의 범위를 축소한 것을 표시한다.|
|Using index|테이블에는 접근하지 않고 인덱스에서만 접근해서 쿼리를 해결하는 것을 의미한다. 커버링 인덱스로 처리됨 index only scan이라고도 부른다|
|Using index for group-by|Using index와 유사하지만 GROUP BY가 포함되어 있는 쿼리를 커버링 인덱스로 해결할 수 있음을 나타낸다|
|Using filesort|ORDER BY 인덱스로 해결하지 못하고, filesort(MySQL의 quick sort)로 행을 정렬한 것을 나타낸다.|
|Using temporary|암묵적으로 임시 테이블이 생성된 것을 표시한다.|
|Using where with pushed|엔진 컨디션 pushdown 최적화가 일어난 것을 표시한다. 현재는 NDB만 유효|
|Using index condition|인덱스 컨디션 pushdown(ICP) 최적화가 일어났음을 표시한다. ICP는 멀티 칼럼 인덱스에서 왼쪽부터 순서대로 칼럼을 지정하지 않는 경우에도 인덱스를 이용하는 실행 계획이다.|
|Using MRR|멀티 레인지 리드(MRR) 최적화가 사용되었음을 표시한다.|
|Using join buffer(Block Nested Loop)|조인에 적절한 인덱스가 없어 조인 버퍼를 이용했음을 표시한다.|
|Using join buffer(Batched Key Access)|Batched Key Access(BKAJ) 알고리즘을 위한 조인 버퍼를 사용했음을 표시한다.|

  
참고 자료  

---
https://www.mysql.com  
https://nomadlee.com/mysql-explain-sql/  
https://cheese10yun.github.io/mysql-explian/  