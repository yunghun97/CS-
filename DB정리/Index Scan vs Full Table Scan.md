# 😦 Index Scan vs Full Table Scan

## Index Scan
### 인덱스는 큰 테이블에서 소량의 데이터를 검색할 때 사용한다. 온라인 트랜젝션 처리 시스템에서는 소량 데이터를 주로 검색하므로 인덱스 튜닝이 무엇보다 중요하다.
### 인덱스를 튜닝한다는 것은 테이블 엑세스를 줄이는 것이다.
![image](https://user-images.githubusercontent.com/71022555/166112306-9d2ab0da-717c-4e4a-a0c1-8fd30da3d02d.png)  
  

1. 인덱스(Index)를 구성하는 컬럼의 값을 기반으로 데이터를 추출하는 액세스 기법이다.
2. 인덱스에 존재하지 않는 컬럼의 값이 필요한 경우에는 현재 읽은 레코드의 식별자를 이용하여 테이블을 액세스 해야한다. 그러나 SQL문에서 필요로 하는 모든 컬럼이 인덱스 구성 컬럼에 포함된 경우 테이블에 대한 액세스는 발생하지 않는다.
3. 인덱스 구성 컬럼이 A+B인 경우 컬럼 A로 정렬 되고 컬럼 A의 값이 동일할 경우 컬럼 B로 정렬된다. 그리고 컬럼 B까지 모두 동일하면 레코드 식별자로 정렬된다.
4. 인덱스가 구성 컬럼으로 정렬되어 있기 때문에 인덱스를 경유하여 데이터를 읽으면 결과 또한 정렬 되어 반환된다. -> 따라서 인덱스 순서와 동일한 정렬 순서를 사용 시 정렬 작업을 하지 않아도 된다.

### 동작 방식
1. 검색할 데이터를 기준으로 인덱스 데이터에 접근을 한다.
2. 인덱스 자료구조에 접근하여 id값을 통해 실질적으로 원하는 데이터를 추출한다.(트리 구조이기 떄문에 평균적으로 logN과정)

## Full Table Scan
![image](https://user-images.githubusercontent.com/71022555/166112625-e8a4ec97-da89-4816-80b1-1540332eedd4.png)  
  
1. 테이블에 존재하는 모든 데이터를 읽어 가면서 조건에 맞으면 결과로서 추출하고 조건에 맞지 않으면 버리는 방식이다.
2. 그림 처럼 일반적으로 블록들은 서로 인적해 있기 때문에 Full Table Scan은 한 번에 I/O에 여러 블록을 옮겨 온다. 즉 한번의 I/O에 데이터를 다중 블록 단위로 메모리에 가져오기 때문에, Row 당 소요되는 입출력 비용이 인덱스 스캔에 비해 적다.
  
### 옵티마이저가 Full Table Scan을 선택하는 경우
|상황|설명|
|:---|:---|
|적용 가능한 인덱스가 없는 경우|1. 결합 인덱스의 선두 컬럼이 존재하지 않을 때.  2. 적용할 인덱스가 있지만 컬럼을 가공, 연산하여 해당 인덱스를 사용할 수 없을 때|
|넓은 범위의 데이터 액세스|인덱스를 적용 가능하더라도 처리 범위가 넓어 Full Table Scan이 더 효과적을 경우 Full Table Scan 적용 가능|
|소량의 테이블 액세스|고수위 마크(HWM)내에 있는 블록이 DB_FILE_MULTIBLOCK_REWAD_COUNT(한번에 I/O 읽어들이는 최대 블럭 수)이내에 있다면 Full Table Scan이 일어날 수 있다.|
|병렬처리 액세스|병렬처리는 Full Table Scan을 더욱 효과적으로 수행하게 되므로 병렬처리로 수행되는 실행계획을 수립할 때는 항상 Full Table Scan을 선택한다.|
|Full 힌트를 적용한 경우|Full 힌트를 사용했을 때 Full 힌트가 적절하지 않는 경우 옵티마이저가 이를 무시 할 수 있다.|
  
> DB_FILE_MULTIBLOCK_READ_COUNT : 이 값이 크면 더 많은 수의 블록을 읽어오므로 Full Table Scan시에는 성능이 증가한다. 하지만 특정 블록만 필요한 경우 낭비가 될 수 있으며 Buffer pool을 많이 차지함으로써 자주 쓰이는 데이터를 밀어내어 Cache효율을 떨어뜨리는 결과를 가져올 수도 있다.

  
## 🤔 뭐가 더 효율적인가요?
### 가정
1. row 개수 : 10,000
2. 1 block = 10 row
3. Index Column의 분포도는 10%
4. 인덱스 row에 해당하는 테이블의 row는 각 block에 하나씩만 존재(최악의 경우)
> 분포도 : 데이터 별 평균 row 수 / 테이블의 총 row 수 * 100
= 1 / 컬럼 값의 종류 * 100
ex) 5개의 사업장을 가진 회사의 '사업장' 컬럼의 종류는 5이므로
사업장 컬럼의 분포도는 1 / 5 * 100 -> 20%

### Full Table Scan 방식
> Scan Cost : 테이블내의 총 row 수 / 블록 내의 row 수
1. DB 테이블 첫 번째 블록부터 순서대로 접근
2. Sacn Cost = 10,000/ 10 -> 1,000

### Index Scan 방식  
1. 해당 인덱스 row에 있는 RowID 정보를 이용하여 대응되는 실제 테이블 row에 접근한다.
2. 그 다음 인덱스 row가 끝날 때까지 인덱스 row를 차례대로 스캔하며 대응되는 실제 테이블의 row를 찾는 작업을 반복한다.
> 총 인덱스 수 = 테이블의 총 row 수 * 인덱스 column의 분포도 -> = 1,000개 -> 인덱스에 대응하는 테이블의 row가 각 block당 1개이므로, 1,000개의 블록 접근 필요
4. 위 계산을 통해, 두 경우 모두 1,000개의 블록 접근이 필요하다.
5. 따라서 **인덱스 사용 손익 분기점은 10%**임을 알 수 있으며 컬럼의 분포도가 10~15%인 경우 인덱스의 RowID를 통한 Random Access 비용 때문에 Table Full Scan이 더 좋은 것을 알 수 있다.  

> 분포도 공식 : 1 : 1/해당 컬럼의 distinct 데이터 개수 * 100 2 : 데이터별 평균 Rows / 테이블 전체 Rows * 100

### 1/해당 컬럼의 distinct 데이터 개수 * 100(MariaDB)
```sql
select (1/cnt)*100
from (
	select count(distinct code) cnt /* select 9 rows */
	from XXX
) a
```
### 데이터별 평균 Rows / 테이블 전체 Rows * 100(MariaDB)
```sql
select ((avg(tot)/(select count(1) from XXX))*100) selectivity
from(
	select count(code) tot
	from XXX
	group by code
) a
```
### 결론
인덱스는 대부분 적은 양의 데이터를 조회할 때 유리하고, 이 외의 경우 Full Scan이 더 유리하다.


참고 자료  

---
https://hoon93.tistory.com/53  
https://techgoeasy.com/how-to-find-which-sid-is-doing-full_30/  
https://jdm.kr/blog/169