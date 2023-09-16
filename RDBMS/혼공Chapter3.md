# Chapter 03. SQL 기본 문법

## 03-1. SELECT ~ FROM ~ WHERE

SELECT 바로 다음에는 열 이름이, FROM 다음에는 테이블 이름이 나온다. WHERE 다음에는 조건식이 나온다.

### 기본 조회하기: Select ~ FROM

**USE문**

SELECT 문을 실행시키려면 먼저 사용할 데이터베이스를 지정해야 한다. 현재 사용하는 데이터베이스를 지정 또는 변경하는 형식은 다음과 같다

```sql
USE 데이터베이스_이름:
```

이렇게 지정해 놓은 후에 다시 USE 문을 사용하거나 다른 DB를 사용하겠다고 명시하지 않으면 앞으로 모든  SQL 문은 market_db에서 수행된다.

**SELECT문의 기본 형식**

```sql
SELECT 열_이름
	FROM 테이블_이름
	WHERE 조건식
	GROUP BY 열_이름
	HAVING 조건식
	ORDER BY 열_이름
	LIMIT 숫자
```

### 특정한 조건만 조회하기: SELECT ~ FROM ~ WHERE

WHERE는 필요한 것들만 골라서 결과를 보는 효과를 갖는다.

**기본적인WHERE절**

WHERE 절은 조회하는 결과에 특정한 조건을 추가해서 원하는 데이터만 보고 싶을 때 사용한다. 형식은 다음과 같다

```sql
SELECT 열_이름 FROM 테이블_이름 WHERE 조건식;
```

**관계 연산자, 논리 연산자의 사용**

숫자로 표현된 데이터는 범위를 지정할 수 있다. 예를 들어 평균 키가 162 이하인 회원을 검색하려면 다음과 같이 관계 연산자를 사용해서 조회할 수 있다.

```sql
SELECT mem_id, mem_name 
	FROM member
	WHERE height <= 162;
```

논리 연산자도 사용할 수 있다.

**BETWEEN ~ AND**

범위에 있는 값을 구하는 경우에는 BETWEEN ~ AND를 사용해도 된다.

```sql
SELECT *
	FROM member
	WHERE height BETWEEN 163 AND 165;
```

**IN()**

평균 키와 같이 숫자로 구성된 데이터는 크다/작다의 범위를 지정할 수 있으므로 BETWEEN ~ AND 를 사용할 수 있지만, 주소와 같은 데이터는 문자로 표현되기 때문에 어느 범위에 들어 있다고 표현할 수 없다. 만약 경기/전남/경남 중 한 곳에 사는 회원을 검색하려면 OR로 일일이 표현해 주어야 한다.

IN() 을 사용하면 코드를 간결하게 작성할 수 있다.

```sql
SELECT *
	FROM member
	WHERE addr IN('경기','전남','경남');
```

**LIKE**

문자열의 일부 글자를 검색하려면 LIKE를 사용한다. 예를들어 이름(mem_name)의 첫 글자가 ‘우’로 시작하는 회원은 다음과 같이 검색할 수 있다. 이 조건은 제일 앞 글자가 ‘우’이고 그 뒤는 무엇이든(%) 허용한다는 의미이다.

```sql
SELECT *
	FROM member
	WHERE mem_name LIKE '우%';
```

한 글자와 매치하기 위해서는 언더바를 사용한다. 다음은 앞 두글자는 상관없고 뒤는 ‘핑크’인 회원을 검색한다

```sql
SELECT *
	FROM member
	WHERE mem_name LIKE '__핑크';
```

### 서브 쿼리

SELECT 안에는 또 다른 SELECT가 들어갈 수 있다. 이것을 서브 쿼리 또는 하위 쿼리라고 부른다. 이름(mem_name_이 ‘에이핑크’인 회원의 평균 키(height)보다 큰 회원을 검색하고 싶다고 가정해 보겠다.

우선 에이핑크의 평균 키를 알아내야 한다.

```sql
SELECT height FROM member WHERE mem_name = '에이핑크';
```

평균 키가 164인 것을 알아냈고, 이제 평균 키가 164보다 큰 회원을 조회하면 된다.

```sql
SELECT mem_name, height FROM member WHERE heigth > 164;
```

SQL 문 2개를 사용해서 결과를 얻었다. 이 두 SQL을 하나로 만들 수 있다.

```sql
SELECT mem_name, height FROM member
	WHERE height > (SELECT height FROM member WHERE mem_name = '에이핑크');
```

## 03-2 좀 더 깊게 알아보는 SELECT 문

SELECT 문에서는 결과의 정렬을 위한 ORDER BY, 결과의 개수를 제한하는 LIMIT, 중복된 데이터를 제거하는 DISTINCT 등을 사용할 수 있다. 

그리고 GROUPY BY 절은 지정한 열의 데이터들을 같은 데이터끼리는 묶어서 결과를 추출한다. 주로 그룹으로 묶는 경우는 합계, 평균, 개수 등을 처리할 때 사용하므로 집계 함수와 함께 사용된다. GROUP BY 절에서도 HAVING 절을 통해 조건식을 추가할 수 있다. HAVING 절은 WHERE 절과 비슷해 보이지만, GROUP BY 절과 함께 사용되는 것이 차이점이다.

### ORDER BY 절

ORDER BY 절은 결과의 값이나 개수에 대해서는 영향을 미치지 않지만, 결과가 출력되는 순서를 조절한다.

```sql
SELECT *
	FROM member
	ORDER BY debut_date;
```

이렇게 하면 데뷔 일자(debut_date)가 빠른 순서대로 출력된다. 늦은 순서대로 정렬하려면 제일 뒤에 DESC라고 붙여주면 된다. 기본값은 ASC이다.

ORDER BY 절과 WHERE 절은 함께 사용할 수 있다. ORDER BY 절은 WHERE 절 다음에 나와야 한다.

```sql
SELECT *
	FROM member
	WHERE height >= 164
	ORDER BY height;
```

정렬 기준을 1개 열이 아니라 여러 개 열로 지정할 수 있다. 우선 첫 번째 지정 열로 정렬한 후에, 동일할 경우에는 다음 지정 열로 정렬할 수 있다. 즉, 평균 키가 큰 순서대로 정렬하되, 평균 키가 같으면 데뷔 일자가 빠른 순서로 정렬한다.

```sql
SELECT *
	FROM member
	WHERE height >= 164
	ORDER BY height DESC, debut_date;
```

### 출력의 개수를 제한: LIMIT

LIMIT는 출력하는 개수를 제한한다. 예를 들어, 회원 테이블을 조회하는데 전체 중 앞에서 3건만 조회할 수 있다.

```sql
SELECT *
	FROM member
	LIMIT 3;
```

이렇게 아무런 기준 없이 앞에서 3건만 뽑는 경우는 별로 없다. 먼저 정렬한 후 앞에서 몇 건을 추출하는 것이 대부분이다. ORDER BY와 함께 사용할 수 있다.

LIMIT 형식은 LIMIT 시작, 개수 이다. 지금과 같이 LIMIT 3만 쓰면 LIMIT 0,3 과 동일하다.

```sql
SELECT *
	FROM member
	ORDER BY debut_date
	LIMIT 3;
```

### 중복된 결과를 제거: DISTINCT

DISTINCT는 조회된 결과에서 중복된 데이터를 1개만 남긴다.

```sql
SELECT DISTINCT addr FROM member;
```

### GROUP BY 절

GROUP BY 절은 말 그대로 그룹으로 묶어주는 역할을 한다. 집계 함수는 주로 GROUP BY 절과 함께 쓰이며 데이터를 그룹화해주는 역할을 한다.

집계 함수

주로 사용되는 집계 함수는 다음 표와 같다.

| 함수명 | 설명 |
| --- | --- |
| SUM() | 합계 |
| AVG() | 평균 |
| MIN() | 최소값 |
| MAX() | 최대값 |
| COUNT() | 행의 개수를 센다 |
| COUNT(DISTINCT) | 행의 개수를 센다(중복은 1개만 인정) |

각 회원(mem_id)별로 구매한 개수를 합쳐서 출력하기 위해서는 집계 함수인 SUM()과 GROUP BY 절을 사용하면 된다.

```sql
SELECT mem_id, SUM(amount) FROM buy GROUP BY mem_id;
```

각 회원이 한 번 구매 시 평균 몇 개를 구매했는지 알아보자

```sql
SELECT mem_id, AVG(amount) FROM buy GROUP BY mem_id;
```

회원 테이블(member)에서 연락처가 있는 회원의 수를 카운트해보자.

```sql
SELECT COUNT(*) FROM member;
```

이렇게 하면 전체 회원 수인 10이 나온다.

연락처가 있는 회원만 카운트하려면 국번(phone1) 또는 전화번호(phone2)의 열 이름을 지정해야 한다. 그러면 NULL 값인 항목은 제외하고 카운트하여 연락처가 있는 회원의 인원만 나온다.

```sql
SELECT COUNT(phone1) "연락처가 있는 회원" FROM member;
```

### HAVING 절

HAVING은 WHERE와 비슷한 개념으로 조건을 제한하는 것이지만, 집계 함수에 대해서 조건을 제한하는 것이다. 그리고 HAVING 절은 꼭 GROUP BY 절 다음에 나와야 한다.

```sql
SELECT mem_id "회원 아이디", SUM(price*amount) "총 구매 금액"
	FROM buy
	GROUP BY mem_id
	HAVING SUM(price*amount) > 1000
	ORDER BY SUM(price*amount) DESC;
```

## 03-3 데이터 변경을 위한 SQL 문

데이터베이스와 테이블을 만든 후에는 데이터를 변경하는, 즉 입력/수정/삭제하는 기능이 필요하다. 입력은 INSERT, 수정은 UPDATE, 삭제는 DELETE 문을 사용한다

### 데이터 입력: INSERT

**INSERT 문의 기본 문법**

INSERT는 테이블에 데이터를 삽입하는 명령이다. 기본적인 형식은 다음과 같다

```sql
INSERT INTO 테이블 [(열1, 열2, ...)] VALUES (값1, 값2, ...)
```

테이블 이름 다음에 나오는 열은 생략이 가능하다. 열 이름을 생략할 경우에 VALUES 다음에 나오는 값들의 순서 및 개수는 테이블을 정의할 때의 열 순서 및 개수와 동일해야 한다.

```sql
USE market_db;
CREATE TABLE hongong1(toy_id INT, toy_name CHAR(4), age INT);
INSERT INTO hongong1 VALUES(1, '우디', 25);
```

테이블의 열이 3개이므로 입력할 때도 차례에 맞춰서 3개를 입력했다.

이 예제에서 아이디와 이름만 입력하고 나이는 입력하고 싶지 않다면 다음과 같이 테이블 이름 뒤에 입력할 열의 이름을 써줘야 한다. 이 경우 생략한 나이 열에는 NULL 값이 들어간다.

```sql
INSERT INTO hongong1(toy_id, toy_name) VALUES(2, '버즈');
```

열의 순서를 바꿔서 입력하고 싶을 때는 열 이름과 값을 원하는 순서에 맞춰 써주면 된다.

**자동으로 증가하는 AUTO_INCREMENT**

AUTO_INCREMENT는 열을 정의할 때 1부터 증가하는 값을 입력해 준다. INSERT에서는 해당 열이 없다고 생각하고 입력하면 된다. 단, 주의할 점은 AUTO_INCREMENT로 지정하는 열은 꼭 PK로 지정해줘야 한다.

```sql
CREATE TABLE hongong2(
	toy_id INT AUTO_INCREMENT PRIMARY KEY,
	toy_name CHAR(4),
	age INT);
```

테이블에 데이터를 입력한다. 자동 증가하는 부분은 NULL 값으로 채워 놓으면 된다.

```sql
INSERT INTO hongong2 VALUES(NULL,'A',25);
INSERT INTO hongong2 VALUES(NULL,'B',23);
INSERT INTO hongong2 VALUES(NULL,'C',21);
SELECT * FROM hongong2;
```

계속 입력하다 보면 현재 어느 숫자까지 증가되었는지 확인이 필요하다.

```sql
SELECT LAST_INSERT_ID();
```

만약 AUTO_INCREMENT로 입력되는 다음 값을 100부터 시작하도록 변경하고 싶다면 다음과 같이 실행한다. ALTER TABLE 뒤에는 테이블 이름을 입력하고, 자동 증가를 100부터 시작하기 위해 AUTO_INCREMENT를 100으로 지정한다.

```sql
ALTER TABLE hongong2 AUTO_INCERMENT=100;
INSERT INTO hongong2 VALUES(NULL,'D','23');
```

ALTER TABLE은 테이블을 변경하라는 의미이다. 테이블의 열 이름 변경, 새로운 열 정의, 열 삭제 등의 작업을 한다.

이번에는 처음부터 입력되는 값을 1000으로 지정하고, 다음 값은 3씩 증가하도록 설정하겠다.

이런 경우에는 시스템 변수인 @@auto_increment_increment를 변경시켜야 한다.

```sql
CREATE TABLE hongong3(
	toy_id INT AUTO_INCREMENT PRIMARY KEY,
	toy_name CHAR(4),
	age INT);
ALTER TABLE hongong3 AUTO_INCREMENT=1000;
SET @@auto_increment_inrement=3;
```

<aside>
📌 시스템 변수란 MySQL에서 자체적으로 가지고 있는 설정값이 저장된 변수를 말한다. 주로 MySQL의 환경과 관련된 내용이 저장되어 있다. 시스템 변수는 앞에 @@가 붙는 것이 특징이며, 시스템 변수의 값을 확인하려면 SELECT @@시스템 변수를 실행하면 된다. 만약 전체 시스템 변수의 종류를 알고 싶다면 SHOW GLOBAL VARIABLES를 실행하면 된다.

</aside>

다음과 같이 여러 줄로 입력되는 INSERT 문을 1줄로 입력할 수 있다.

```sql
INSERT INTO 테이블_이름 VALUES(값1, 값2, ...), (값3, 값4,...), (값5,값6,...);
```

**다른 테이블의 데이터를 한번에 입력하는 INSERT INTO ~ SELECT**

많은 양의 데이터를 직접 타이핑해서 입력하려면 오랜 시간이 걸린다. 다른 테이블에 이미 데이터가 입력되어 있다면 이 구문을 사용해 해당 테이블의 데이터를 가져와서 한 번에 입력할 수 있다.

```sql
INSERT INTO 테이블_이름 (열_이름1, 열_이름2, ...)
	SELECT 문;
```

주의할 점은 SELECT 문의 열 개수는 INSERT할 테이블의 열 개수와 같아야 한다.

DESC 명령으로 테이블 구조를 확인할 수 있다. DESC는 Describe의 약자로 테이블의 구조를 출력해주는 기능을 한다.

```sql
DESC world.city;
```

**데이터 수정: UPDATE**

UPDATE는 기존에 입력되어 있는 값을 수정하는 명령이다. 기본적인 형식은 다음과 같다.

```sql
UPDATE 테이블_이름
	SET 열1=값1, 열2=값2, ...
	WHERE 조건;
```

**WHERE가 없는 UPDATE 문**

UPDATE 문에서 WHERE 절은 문법상 생략이 가능하지만, WHERE 절을 생략하면 테이블의 모든 행의 값이 변경된다. 일반적으로 전체 행의 값을 변경하는 경우는 별로 없으므로 주의해야 한다.

다음 SQL을 이용해서 모든 인구 열을 한번에 10,000으로 나눌 수 있다.

```sql
UPDATE city_popul
	SET popul = popul/10000;
SELECT * FROM city_popul LIMIT 5;
```

데이터 삭제: DELETE

테이블의 행 데이터를 삭제해야 하는 경우도 발생한다. 예를 들어 회원이 탈퇴한 경우에 해당 회원의 정보를 삭제해야 한다.

DELETET도 UPDATE와 거의 비슷하게 사용할 수 있다. DELETE는 행 단위로 삭제하며, 형식은 다음과 같다.

```sql
DELETE FROM 테이블이름 WHERE 조건;
```

city_popul 테이블에서 ‘New’로 시작하는 도시를 삭제하기 위해 다음과 같이 실행한다.

```sql
DELETE FROM city_popul
	WHERE city_name LIKE 'New%';
```

### 대용량 테이블의 삭제

만약 몇억 건의 데이터가 있는 대용량 테이블이 더 이상 필요 없다면 어떻게 삭제하는 것이 좋을까?

우선 대용량 테이블을 3개 준비한다

```sql
CREATE TABLE big_table1 (SELECT * FROM world.city, sakila.country);
CREATE TABLE big_table2 (SELECT * FROM world.city, sakila.country);
CREATE TABLE big_table3 (SELECT * FROM world.city, sakila.country);
SELECT COUNT(*) FROM big_table1;
```

이제 동일한 내용의 대용량 테이블 3개를 DELETE, DROP, TRUNCATE 각각 다른 방법으로 삭제한다.

우선 DELETE 문은 삭제가 오래 걸린다. DROP 문은 테이블 자체를 삭제한다. 그래서 순식간에 삭제된다. TRUNCATE 문도 DELETE와 동일한 효과를 내지만 속도가 빠르다. DROP은 테이블이 아예 없어지지만, DELETE와 TRUNCATE는 빈 테이블을 남긴다.

```sql
DELETE FROM big_table1;
DROP TABLE big_table2;
TRUNCATE TABLE big_table3;
```

### 마무리

- INSERT 문은 테이블에 데이터를 입력하는 명령이다
- AUTO_INCREMENT는 1부터 증가하는 값을 자동으로 입력해준다. 해당 열은 PRIMARY KEY로 지정해야 한다
- INSERT INTO ~ SELECT는 다른 테이블의 데이터를 가져와서 한 번에 대량으로 입력한다.
- UPDATE는 기존에 입력되어 있는 값을 수정하며 주로 WHERE와 함께 사용한다.
- DELETE는 행 단위로 삭제하며 WHERE가 없으면 전체 행이 삭제된다.