# Chapter 04. SQL 고급 문법

# 04-1 데이터 형식

### 정수형

정수형은 소수점이 없는 숫자, 즉 인원 수, 가격, 수량 등에 많이 사용한다. 정수형에는 TINYINT, SMALLINT, INT, BIGINT가 있다.

값의 범위가 0부터 시작되는 UNSIGNED 예약어를 사용할 수 있다. TINYINT는 -128~127 이고, TINYINT UNSIGNED는 0~255까지 이다.

### 문자형

문자형은 글자를 저장하기 위해 사용하며, 입력할 최대 글자의 개수를 지정해야 한다. 대표적인 문자형은 다음과 같다

| 데이터 형식 | 바이트 수 |
| --- | --- |
| CHAR(개수) | 1~255 |
| VARCHAR(개수) | 1~16383 |

CHAR는 문자를 의미하는 Character의 약자로, 고정길이 문자형이라고 부른다. 즉, 자릿수가 고정되어 있다. 예를 들어 CHAR(10)에 ‘가나다’ 3글자만 저장해도 10자리를 모두 확보한 후에 앞에 3자리를 사용하고 뒤의 7자리는 낭비하게 된다. 이와 달리 VARCHAR는 가변길이 문자형으로 3자리만 사용한다.

VARCHAR가 CAHR보다 공간을 효율적으로 운영할 수 있지만 MySQL 내부적으로 성능면에서는 CHAR로 설정하는 것이 조금 더 좋다.

### 대량의 데이터 형식

문자열인 CHAR는 최대 255자까지, VARCHAR는 최대 16383자까지 지정이 가능하다. 더 큰 데이터를 TEXT,LONGTEXT, BLOB, LONGBLOB를 사용한다

TEXT는 최대 65535자, LONGTEXT는 최대 약 42억자까지 저장된다. BLOB는 Binary Long Objext의 약자로 글자가 아닌 이미지, 동영상 등의 데이터이다 LONGTEXT 및 LONGBLOB로 설정하면 각 데이터는 최대 4GB까지 입력할 수 있다.

### 실수형

FLOAT는 4바이트, 소수점 아래 7자리까지, DOUBLE은 8바이트, 소수점 아래 15자리까지 표현한다.

### 날짜형

날짜형은 날자 및 시간을 저장할 때 사용한다.

| 데이터 형식 | 바이트 수 | 설명 |
| --- | --- | --- |
| DATE | 3 | 날짜만 저장. YYYY-MM-DD  |
| TIME | 3 | 시간만 저장. HH:MM:SS |
| DATETIME | 8 | 날짜 및 시간을 저장  YYYY-MM-DD HH:MM:SS |

## 변수의 사용

SQL도 다른 일반적인 프로그래밍 언어처럼 변수를 선언하고 사용할 수 있다. 변수의 선언과 값의 대입은 다음 형식을 따른다.

```sql
SET @변수이름 = 변수의 값;
SELECT @변수이름;
```

변수는 MySQL 워크벤치를 재시작할 때까지는 유지되지만, 종료하면 없어진다. 그러므로 임시로 사용한다고 생각하면 된다.

```sql
USE market_db;
SET @myVar1 = 5;
SET @myVar2 = 4.25;

SELECT @myVar1 ;
SELECT @myVar1 + @myVar2 ;

SET @txt = '가수이름==> ' ;
SET @hight = 166;
SELECT @txt, mem_name FROM member WHERE height > @height;
```

결과

![Untitled](/assets/혼공sql1.png)

![Untitled](/assets/혼공sql2.png)

![Untitled](/assets/혼공sql3.png)

SELECT 문에서 행의 개수를 제한하는 LIMIT에는 변수를 사용할 수 없다. 이를 해결하는 것이 PREPARE와 EXECUTE이다. PREPARE는 실행하지 않고 SQL 문만 준비해 놓고 EXECUTE에서 실행하는 방식이다.

```sql
SET @count = 3;
PREPARE mySQL FROM 'SELECT mem_name, height FROM member ORDER BY height LIMIT ?';
EXECUTE mySQL USING @count;
```

1. @count 변수에 3을 대입한다
2. PREPARE는 ‘SELECT ~ FROM ~ LIMIT ?’ 문을 실행하지 않고 mySQL이라는 이름으로 준비만 해 놓는다. ?는 ‘현재는 모르지만 나중에 채워짐’ 같은 의미이다.
3. EXECUTE로 mySQL에 저장된 select 문을 실행할 때, USING으로 ?에 @count 변수의 값을 대입한다.

## 데이터 형 변환

직접 함수를 사용해서 변환하는 명시적인 변환과 별도의 지시 없이 자연스럽게 변환되는 암시적인 변환이 있다.

### 함수를 이용한 명시적인 변환

CAST(), CONVERT() 함수를 사용한다. 형식만 다를 뿐 동일한 기능을 한다.

```sql
CAST(값 AS 데이터_형식 [(길이)])
CONVERT(값, 데이터_형식 [(길이)])
```

```sql
SELECT AVG(price) AS '평균 가격' FROM buy;
```

결과가 실수로 나온다. 가격은 실수보다 정수로 표현하는 것이 보기 좋을 것이다.

```sql
SELECT CAST(AVG(price) AS SIGNED) '평균 가격' FROM buy;

SELECT CONVERT(AVG(price) , SIGNED) '평균 가격' FROM buy;
```

SIGNED는 부호가 있는 정수, UNSIGNED는 부호가 없는 정수이다.

```sql
SELECT CAST('2022$12$12' AS DATE);
SELECT CAST('2022%12%12' AS DATE);
SELECT CAST('2022/12/12' AS DATE);
SELECT CAST('2022@12@12' AS DATE);
```

다양한 구분자를 날짜형으로 변경할 수도 있다.

SQL의 결과를 원하는 형태로 표현할 때도 사용할 수 있다. 가격과 수량을 곱한 실제 구매액을 표시하는 SQL을 다음과 같이 작성할 수 있다.

```sql
SELECT num, CONCAT(CAST(price AC CHAR), 'X', CAST(amount AS CHAR), '=')
			'가격X수량', price*amount '구매액'
			FROM buy;
```

가격과 수량은 정수지만,  CAST() 함수를 통해 문자로 바꿨다. CONCAT() 함수는 문자를 이어주는 역할을 한다.

### 암시적인 변환

암시적인 변환은 함수를 사용하지 않고도 자연스럽게 형이 변환되는 것이다.

```sql
SELECT CONCAT(100,'200');
SELECT 100 + '200';
```

실행결과 100200과 300이 나온다. 숫자 100이 문자 ‘100’으로 변환되었고, 문자 ‘200’이 숫자 200으로 변환되었다.

# 04-2 두 테이블을 묶는 조인

조인이란 두 개의 테이블을 서로 묶어서 하나의 결과를 만들어 내느 것을 말한다. 두 테이블을 엮어야만 원하는 형태가 나오는 경우도 많다.

### 내부 조인

두 테이블을 연결할 때 가장 많이 사용되는 것이 내부 조인이다. 그냥 조인이라 부르면 내부 조인을 의마하는 것이다.

**일대다 관계의 이해**

두 체이블의 조인을 위해서는 테이블이 일대다 관계로 연결되어야 한다. 먼저 일대다 관계에 대해서 알아보자

데이터베이스의 테이블은 하나로 구성되는 것보다는 여러 정보를 주제에 따라 분리해서 저장하는 것이 효율적이다. 이 분리된 테이블은 서로 관계를 맺고 있다. 이러한 대표적인 사례가 인터넷 마켓 데이터베이스의 회원 테이블과 구매 테이블이다.

market_db에서 회원 테이블의 아이디와 구매 테이블의 아이디는 일대다 관계이다. 일대다 관계란 한쪽 테이블에는 하나의 값만 존재해야 하지만, 연결된 다른 테이블에는 여러개의 값이 존재할 수 있는 관계를 말한다.

예를 들어, 회원 테이블에서 블랙핑크의 아이디는 ‘BLK’로 1명밖에 없다. 그래서 회원 테이블의 아이디를 기본키(PK)로 지정했다. 구매 테이블의 아이디에서는 3개의 BLK를 찾을 수 있다. 즉, 회원은 1명이지만 이 회원은 구매를 여러번 할 수 있는 것이다. 그래서 구매 테이블의 아이디는 기본키가 아닌 외래 키(FK)로 설정했다.

<aside>
💡 일대다 관계는 주로 PK와 FK 관계로 맺어져 있다. 그래서 일대다 관계를 ‘PK-FK 관계’라고 부르기도 한다

</aside>

**내부 조인의 기본**

조인은 3개 이상의 테이블로도 할 수 있지만 대부분은 2개로 조인한다.

내부 조인의 형식은 다음과 같다

```sql
SELECT <열 목록>
FROM <첫 번째 테이블>
		INNER JOIN <두 번째 테이블>
		ON <조인될 조건>
[WHERE 검색 조건]
```

- INNER JOIN을 그냥 JOIN이라고만 써도 INNER JOIN으로 인식한다.

구매 테이블에는 물건을 구매한 회원의 아이디와 물건 등의 정보만 있다. 이 물건을 배송하기 위해서는 구매한 회원의 주소 및 연락처를 알아야 한다. 이 회원의 주소, 연락처를 알기 위해 정보가 있는 회원 테이블과 결합하는 것이 내부 조인이다.

구매 테이블에서 GRL이라는 아이디를 가진 사람이 구매한 물건을 발송하기 위해 다음과 같이 조인해서 이름/주소/연락처 등을 검색할 수 있다.

```sql
USE market_db;
select *
	from buy
    inner join member
    on buy.mem_id = member.mem_id
where buy.mem_id = 'GRL';
```

![Untitled](/assets/혼공sql4.png)

1. 구매 테이블의 mem_id(buy.mem_id)인 ‘GRL’을 추출한다
2. ‘GRL’과 동일한 값을 회원 테이블의 mem_id(member.mem_id) 열에서 검색한다
3. ‘GRL’이라는 아이디를 찾으면 구매 테이블과 회원 테이블의 두 행을 결합(JOIN)한다.

만약 WHERE 절을 생략하면 어떻게 될까? 원래는 구매 테이블의 7번째에 있는 GRL에 대해서만 결합했지만, 생략하면 구매 테이블의 모든 행이 회원 테이블과 결합한다.

**내부 조인의 간결한 표현**

결합된 결과가 열이 너무 많아 복잡해 보이므로 이번에는 필요한 아이디/이름/구매 물품/주소/연락처만 추출해 보겠다.

```sql
SELECT mem_id, mem_name, prod_name, addr, CONCAT(phone1,phone2) '연락처'
	from buy
	inner join member
	on buy.mem_id = member.mem_id;
```

- Error Code: 1052. Column 'mem_id' in field list is ambiguous

열 이름인 mem_id가 불확실하다는 에러가 발생한다. 즉, 회원 아이디는 회원 테이블과 구매 테이블에 모두 들어 있어서 어느 테이블의 mem_id인지 헷갈린다는 의미이다.

이럴 때는 어느 테이블의 mem_id를 추출할지 정확하게 작성해야 한다. 지금은 구매 테이블을 기준으로 하는 것이므로 buy.mem_id가 논리적으로 더 맞을 것 같다.

```sql
SELECT buy.mem_id, mem_name, prod_name, addr, CONCAT(phone1,phone2) '연락처'
	from buy
	inner join member
	on buy.mem_id = member.mem_id;
```

![Untitled](/assets/혼공sql5.png)

코드를 간결하게 표현하기 위해서 다음과 같이 from 절에 나오는 테이블의 이름 뒤에 별칭을 줄 수 있다.

```sql
SELECT B.mem_id, M.mem_name, B.prod_name, M.addr, CONCAT(M.phone1,M.phone2) '연락처'
	from buy B
	inner join member M
	on B.mem_id = M.mem_id;
```

**내부 조인의 활용**

이번에는 전체 회원의 아이디/이름/구매한 제품/주소를 출력하겠다. 결과는 보기 쉽게 회원 아이디 순으로 정렬한다.

```sql
SELECT B.mem_id, M.mem_name, B.prod_name, M.addr
	from buy B
		inner join member M
		on B.mem_id = M.mem_id;
	order by M.mem_id;
```

![Untitled](/assets/혼공sql6.png)

구매 테이블의 목록이 12건이었으므로 이상 없이 잘 나왔다. 결과는 이상이 없지만, 조금 전에 말했던 ‘전체 회원’과는 차이가 있다. 결과는 ‘전체 회원’이 아닌 ‘구매한 기록이 있는 회원들’의 목록이다.

결과에 한 번도 구매하지 않은 회원의 정보는 없다. 우리가 원하는 결과는 구매한 회원의 구매 기록과 더불어 구매하지 않은 회원의 이름/주소가 같이 검색되도록 하는 것이다.

지금까지 사용한 내부 조인은 두 테이블에 모두 있는 내용만 조인되는 방식이다. 만약, 양쪽 중에 한곳이라도 내용이 있을 때 조인하려면 외부 조인을 사용해야 한다.

### 외부 조인

**외부 조인의 기본**

외부 조인은 두 테이블을 조인할 때 필요한 내용이 한쪽 테이블에만 있어도 결과를 추출할 수 있다. 외부 조인의 형식은 다음과 같다

```sql
SELECT <열 목록>
FROM <첫 번째 테이블(LEFT 테이블)>
		<LEFT | RIGHT | FULL> OUTER JOIN <두 번째 테이블(RIGHT 테이블)>
		ON <조인될 조건>
[WHERE 검색 조건];
```

내부 조인보다는 조금 복잡해 보이지만 사용 방법은 거의 비슷하다. 먼저 내부 조인에서 해결하지 못한 전체 회원의 구매 기록 출력을 외부 조인으로 만들어보자.

```sql
SELECT B.mem_id, M.mem_name, B.prod_name, M.addr
	from member M
		left outer join buy B
		on B.mem_id = M.mem_id
	order by M.mem_id;
```

LEFT OUTER JOIN 문의 의미를 ‘왼쪽 테이블(member)의 내용은 모두 출력되어야 한다’ 정도로 해석하면 기억하기 쉽다.

**외부 조인의 활용**

이번에는 회원으로 가입만 하고, 한 번도 구매한 적이 없는 회원의 목록을 추출해보자

```sql
SELECT distinct M.mem_id, M.mem_name, B.prod_name, M.addr
	from member M
		left outer join buy B
		on B.mem_id = M.mem_id
	where B.prod_name is null
	order by M.mem_id;
```

![Untitled](/assets/혼공sql7.png)

FULL OUTER JOIN은 왼쪽 외부 조인과 오른쪽 외부 조인이 합쳐진 것이라고 생각하면 된다. 왼쪽이든 오른쪽이든 한쪽에 들어 있는 내용이면 출력한다.

### 기타 조인

내부 조인이나 외부 조인처럼 자주 사용되지는 않지만 가끔 유용하게 사용되는 조인으로 상호 조인과 자체 조인도 있다 .

**상호 조인**

상호 조인은 한쪽 테이블의 모든 행과 다른 쪽 테이블의 모든 행을 조인시키는 기능을 말한다. 그래서 상호 조인 결과의 전체 행 개수는 두 테이블의 각 행의 개수를 곱한 개수가 된다.

```sql
SELECT *
	from buy
		cross join member;
```

![Untitled](/assets/혼공sql8.png)

회원 테이블의 첫 행은 구매 테이블의 모든 행과 조인된다. 나머지 행도 마찬가지다. 즉 회원 테이블의 모든 행이 구매 테이블의 모든 행과 결합되므로 회원 테이블의 10개 행과 구매 테이블의 12개 행을 곱해서 총 120개의 결과가 생성되는 것이다.

상호 조인은 다음과 같은 특징을 갖는다.

- on 구문을 사용할 수 없다
- 결과의 내용은 의미가 없다. 랜덤으로 조인하기 때문이다
- 상호 조인의 주 용도는 테스트하기 위해 대용량의 데이터를 생성할 때이다.

**자체 조인**

내부 조인, 외부 조인, 상호 조인은 모두 2개의 테이블을 조인했다. 자체 조인은 자신이 자신과 조인한다는 의미이다. 그래서 자체 조인은 1개의 테이블을 사용한다. 또, 별도의 문법이 있는 것은 아니고 1개로 조인하면 자체 조인이 되는 것이다.

하나의 테이블에 같은 데이터가 있지만 2개 이상의 열로 존재할 때 자체 조인을 할 수 있다.

# 04-3 SQL 프로그래밍

스토어드 프로시저는 MySQL에서 프로그래밍 기능이 필요할 때 사용하는 데이터베이스 개체이다. SQL 프로그래밍은 기본적으로 스토어드 프로시저 안에 만들어야 한다. 

스토어드 프로시저는 다음과 같은 구조를 가진다.

```sql
DELIMITER $$
CREATE PROCEDURE 스토어드_프로시저_이름()
BEGIN
		이 부분에 SQL 프로그래밍 코딩
END $$
DELIMITER;
CALL 스토어드_프로시저_이름();
```

<aside>
💡 일반적으로 구분문자(DELIMITER)는 $$를 많이 사용하지만, 원한다면 /,&,@ 등을 사용해도 상관없다. 다른 기호와 중복될 수 있으므로 기호 2개를 연속해서 사용하는 것이 좋다.

</aside>

## IF 문

### IF 문의 기본 형식

IF 문은 조건식이 참이라면 ‘SQL문장들’을 실행하고 그렇지 않으면 그냥 넘어간다.

```sql
IF <조건식> THEN
				SQL 문장들
END IF;
```

‘SQL문장들’이 한 문장이라면 그 문장만 써도 되지만, 두 문장 이상이 처리되어야 할 때는 BEGIN ~ END로 묶어줘야 한다.

```sql
DROP PROCEDURE IF EXISTS ifProc1;
DELIMITER $$
CREATE PROCEDURE ifProc1()
BEGIN
	IF 100 = 100 THEN
		SELECT '100은 100과 같습니다.';
	END IF;
END $$
DELIMITER ;
CALL ifProc1();
```

1. 만약 기존에 ifProc1()을 만든 적이 있다면 삭제한다.
2. 스토어드 프로시저의 시작과 끝을 구분하기 위해 $$를 사용한다
3. 스토어드 프로시저의 이름을 ifProc1()으로 지정한다
4. 조건식으로 100과 100이 같은지 비교한다. 참이므로 다음 행이 실행된다.
5. CALL로 ifProc1()을 호출한다.

<aside>
💡 SQL은 같다의 조건으로 =을 사용한다. 그리고 SELECT 뒤에 문자가 나오면 그냥 화면에 출력해준다.

</aside>

### IF~ELSE 문

```sql
DROP PROCEDURE IF EXISTS ifProc2;
DELIMITER $$
CREATE PROCEDURE ifProc2()
BEGIN
	DECLARE myNum INT;
	SET myNum = 200;
	IF myNum = 100 THEN
		SELECT '100입니다.';
	ELSE
		SELECT '100이 아닙니다.';
	END IF;
END $$
DELIMITER ;
CALL ifProc2();
```

1. DECLARE 예약어를 사용해서 myNum 변수를 선언한다. 변수의 데이터 형식은 INT로 지정한다.
2. SET 예약어로 변수에 200을 대입한다.
3. 변수가 100인지 아닌지를 구분한다.

### IF 문의 활용

기존 테이블과 함께 IF 문을 활용해보자. 아이디가 APN인 회원의 데뷔 일자가 5년이 넘었는지 확인해보고 5년이 넘으면 축하 메세지를 출력해보자.

```sql
DROP PROCEDURE IF EXISTS ifProc3;
DELIMITER $$
CREATE PROCEDURE ifProc3()
BEGIN
	DECLARE debutDate DATE;
    DECLARE curDate DATE;
    DECLARE days INT;
    SELECT debut_date INTO debutDate
		FROM market_db.member
		WHERE mem_id = 'APN';
	SET curDate = current_date();
    SET days = datediff(curDate, debutDate);
    IF(days/365) >= 5 THEN
		SELECT CONCAT('데뷔한 지', days, '일이나 지났습니다. 축하합니다') ;
	else
		SELECT CONCAT('데뷔한 지', days, '일밖에 안됐습니다.');

	END IF;
END $$
DELIMITER ;
CALL ifProc3();
```

1. 변수 3개로 데뷔일자, 오늘날짜, 데뷔 일자부터 오늘까지 몇일이 지났는지를 각각 debutDate, curDate, days에 저장한다.
2. APN의 데뷔 일자를 추출한다. SELECT debut_date INTO debutDate와 같이 INTO 변수를 사용하면 결과를 변수에 저장한다. 결국 APN의 데뷔 일자가 debutDate에 저장된다.
3. current_date() 함수로 현재 날짜를 curDate에 저장한다.
4. datediff() 함수로 데뷔 일자부터 현재 날짜까지 일수를 days에 저장한다.
5. 조건문을 거쳐 출력한다.

<aside>
💡 MySQL은 날짜와 관련된 함수를 여러 개 제공해준다.
**CURRENT_DATE()** : 오늘 날짜를 알려준다.
**CURRENT_TIMESTAMP()** : 오늘 날짜 및 시간을 함께 알려준다.
**DATEDIFF(날짜1, 날짜2)** : 날짜2부터 날짜1까지 몇일인지 알려준다.

</aside>

## CASE 문

여러 가지 조건 중에서 선택해야 하는 경우도 있다. 이럴 때 CASE 문을 사용해서 조건을 설정할 수 있다.

### CASE 문의 기본 형식

CASE 문은 2가지 이상의 여러 가지 경우일 떄 처리가 가능하므로 ‘다중 분기’라고 부른다.

```sql
CASE
	WHEN 조건1 THEN
		SQL 문장들1
	WHEN 조건2 THEN
		SQL 문장들2
	WHEN 조건3 THEN
		SQL 문장들3
	ELSE
		SQL 문장들4
END CASE;
```

예시로 시험 점수에 따른 학점을 부여하는 코드를 보자

```sql
DROP PROCEDURE IF EXISTS caseProc;
DELIMITER $$
CREATE PROCEDURE caseProc()
BEGIN
	DECLARE point int;
    DECLARE credit CHAR(1);
	SET point = 88;
    
    case
		when point >= 90 then
			set credit = 'A';
		when point >= 80 then
			set credit = 'B';
		when point >= 70 then
			set credit = 'C';
		when point >= 60 then
			set credit = 'D';
		else
			set credit = 'F';
    end case;
    select concat('취득점수==>', point) '취득점수', concat('학점==>', credit) '학점';
END $$
DELIMITER ;
CALL caseProc();
```

### CASE 문의 활용

인터넷 마켓 데이터베이스의 회원들은 물건을 구매한다. 회원들의 총 구매액을 계산해서 회원의 등급을 다음과 같이 4단계로 나누려 한다.

![Untitled](/assets/혼공sql9.png)

먼저 구매 테이블에서 회원별로 총 구매액을 구해보자. GROUP BY를 이용해서 다음과 같이 만들 수 있다.

```sql
select mem_id, sum(price*amount) '총 구매액'
	from buy
	group by mem_id;
```

![Untitled](/assets/혼공sql10.png)

추가로 order by를 사용해서 총 구매액이 많은 순서대로 정렬한다.

```sql
select mem_id, sum(price*amount) '총 구매액'
	from buy
	group by mem_id
	order by sum(price*amount) desc;
```

회원의 이름도 출력한다. 그런데 회원의 이름은 회원 테이블에 있으므로 구매 테이블과 조인시켜야 한다.

```sql
use market_db;

select B.mem_id, M.mem_name, sum(price*amount) '총 구매액'
	from buy B
		inner join member M
        on B.mem_id = M.mem_id
	group by B.mem_id
    order by sum(price*amount) desc;
```

![Untitled](/assets/혼공sql11.png)

이번에는 구매하지 않은 회원의 아이디와 이름도 출력해 보겠다. 내부 조인 대신에 외부 조인을 시키면 된다. 그리고 구매 테이블에는 구매한 적이 없어도 회원 테이블에 있는 회원은 모두 출력해야 하므로 inner join을 right outer join으로 변경한다.

```sql
use market_db;

select B.mem_id, M.mem_name, sum(price*amount) '총 구매액'
	from buy B
		right outer join member M
        on B.mem_id = M.mem_id
	group by M.mem_id
    order by sum(price*amount) desc;
```

![Untitled](/assets/혼공sql12.png)

주의할 점은 구매 테이블에는 4명만 구매했으므로, 나머지 6명에 대한 아이디 등의 정보가 없다. 그래서 select에서 회원 테이블의 아이디인 M.mem_id를 조회하고 group by도 M.mem_id로 변경했다.

이제 계획한 대로 총 구매액에 따라 회원 등급을 구분하자. case 문을 사용하면 다음과 같다

```sql
case
	when (총 구매액 >= 1500) then '최우수고객'
	when (총 구매액 >= 1000) then '우수고객'
	when (총 구매액 >= 1) then '일반고객'
	else '유령고객'
end
```

이제 이 case 문을 새로운 열로 추가하면 된다. 다음과 같이 콤마로 구분해서 열의 마지막에 추가한다. 열 이름에 별칭도 지정했다.

```sql
use market_db;

select M.mem_id, M.mem_name, sum(price*amount) '총 구매액',
case
	when ( sum(price*amount) >= 1500) then '최우수고객'
	when ( sum(price*amount) >= 1000) then '우수고객'
	when ( sum(price*amount) >= 1) then '일반고객'
	else '유령고객'
end '회원등급'
	from buy B
		right outer join member M
        on B.mem_id = M.mem_id
	group by M.mem_id
    order by sum(price*amount) desc;
```

![Untitled](/assets/혼공sql13.png)

## WHILE 문

### WHILE 문의 기본 형식

```sql
WHILE <조건식> DO
		SQL 문장들
END WHILE;
```

1에서 100까지의 값을 모두 더하는 간단한 기능을 WHILE 문으로 구현해보자.

```sql
drop procedure if exists whileProc;
DELIMITER $$
create procedure whileProc()
begin
	  declare i int;
    declare sum int;
    set i = 1;
    set sum = 0;
    
    while(i <= 100) do
		set sum = sum + i;
        set i = i + 1;
	end while;
    select '1부터 100까지의 합 ==>', sum;
end $$
delimiter ;
call whileProc();
```

![Untitled](/assets/혼공sql14.png)

### while 문의 응용

1에서 100의 합계에서 4의 배수를 제외시키려면 어떻게 해야할까? 추가로 숫자를 더하는 중간에 합계가 100이 넘으면 더하는 것을 그만두고, 100이 넘는 순간의 숫자를 출력한 후 프로그램을 종료하고 싶다면 어떻게 해야 할까? 이런 경우에는 ITERATE 문과 LEAVE 문을 활용할 수 있다.

- ITERATE[레이블] : 지정한 레이블로 가서 계속 진행한다.
- LEAVE[레이블] : 지정한 레이블을 빠져나간다. 즉 WHILE 문이 종료된다.

<aside>
💡 ITERATE 문을 프로그래밍 언어의 CONTINUE와, LEAVE 문은 BREAK와 비슷한 역할을 한다.

</aside>

```sql
drop procedure if exists whileProc;
DELIMITER $$
create procedure whileProc()
begin
	declare i int;
    declare sum int;
    set i = 1;
    set sum = 0;
    
    mywhile:
    while(i <= 100) do
		if(i%4 = 0) then
			set i = i + 1;
            iterate mywhile;
		end if;
		set sum = sum + i;
        if(sum > 1000) then
			leave mywhile;
		end if;
        set i = i + 1;
	end while;
    select '1부터 100까지의 합(4의 배수 제외), 1000 넘으면 종료 ==>', sum;
end $$
delimiter ;
call whileProc();
```

![Untitled](/assets/혼공sql16.png)

## 동적 SQL

SQL 문은 내용이 고정되어 있는 경우가 대부분이다. 하지만 상황에 따라 내용 변경이 필요할 때 동적 SQL을 사용하면 변경되는 내용을 실시간으로 적용시켜 사용할 수 있다.

### PREPARE와 EXECUTE

PREPARE는 SQL 문을 실행하지는 않고 미리 준비만 해놓고, EXECUTE는 준비한 SQL 문을 실행한다. 그리고 실행 후에는 DEALLOCATE PREPARE로 문장을 해제해주는 것이 바람직하다.

```sql
use market_db;
prepare myQuery from 'select * from member where mem_id = "BLK"';
execute myQuery;
deallocate prepare myQuery;
```

![Untitled](/assets/혼공sql18.png)

이렇게 미리 SQL을 준비한 후에 나중에 실행하는 것을 동적 SQL이라고 부른다.

### 동적 SQL의 활용

PREPARE 문에서는 ?로 향후에 입력될 값을 비워 놓고, EXECUTE에서 USING으로 ?에 값을 전달할 수 있다. 그러면 실시간으로 필요한 값들을 전달해서 동적으로 SQL이 실행된다. 다음 예제로 살펴보자

보안이 중요한 출입문에서는 출입한 내역을 테이블에 기록해 놓는다. 이때 출입증을 태그하는 순간의 날짜와 시간이 INSERT 문으로 만들어져서 입력되도록 해야 한다.

```sql
drop table if exists gate_table;
create table gate_table(id int AUTO_INCREMENT PRIMARY KEY, entry_time DATETIME);

set @curDate = current_timestamp();

prepare myQuery from 'insert into gate_table values(NULL, ?)';
execute myQuery using @curDate;
deallocate prepare myQuery;

select * from gate_table;
```

![Untitled](/assets/혼공sql17.png)

1. 출입용 테이블을 간단히 만들었다. 아이디는 자동으로 증가되도록 하고, 출입하는 시간을 DATETIME형으로 준비했다.
2. 현재 날짜와 시간을 @curDate 변수에 넣는다
3. ?를 사용해서 entry_time에 입력할 값을 비워 놓는다
4. using 문으로 앞에서 준비한 @curDate 변수를 넣은 후에 실행된다. 결국 이 SQL을 실행한 시점의 날짜와 시간이 입력된다.