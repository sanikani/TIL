# Chapter 05. 테이블과 뷰

## 05-2 제약조건으로 테이블을 견고하게

테이블을 만들 때는 테이블의 구조에 필요한 제약조건을 설정해줘야 한다. 앞에서 확인한 기본 키와 외래 키가 대표적인 제약조건이다.

이메일, 휴대폰과 같이 중복되지 않는 열에는 고유 키를 지정할 수 있다. 회원의 평균 키는 200cm를 넘지 않을 것이기 때문에, 실수로 200을 입력하는 것을 방지하는 제약조건이 체크이다. 회원 테이블에 국적을 입력한다면 대부분은 대한민국일 것이다. 제약조건으로 기본값을 설정할 수 있다. 또한, 값을 꼭 입력해야 하는 NOT NULL 제약조건도 있다.

## 제약조건의 기본 개념과 종류

제약조건은 데이터의 무결성을 지키기 위해 제한하는 조건이다. 일단 데이터의 무결성이란 데이터에 결함이 없음이란 의미이다.

MySQL에서 제공하는 대표적인 제약조건은 다음과 같다

- PRIMARY KEY
- FOREIGN KEY
- UNIPUE
- CHECK
- DEFAULT 정의
- NULL 값 허용

## 기본 키 제약조건

데이터를 구분할 수 있는 식별자를 기본 키라고 부른다. 기본 키에 입력되는 값은 중복될 수 없으며, NULL 값이 입력될 수 없다.

<aside>
💡 많은 인터넷 쇼핑몰에서 기본 키를 아이디로 지정하고 있다. 하지만 설계 방법에 따라서 아이디가 아닌 다른 열을 기본 키로 지정할 수 있다. 기본 키는 중복되지 않고, 비어 있지 않으면 되기 때문에 최근에는 주민등록번호나 이메일 또는 휴대폰 번호 등으로 지정해서 회원을 구분하는 사이트도 많이 등장하고 있다.

</aside>

대부분의 테이블은 기본 키를 가져야 한다. 기본 키로 생성한 것은 자동으로 클러스터형 인덱스가 생성된다.

테이블은 기본 키를 1개만 가질 수 있다. 어떤 열에 설정해도 문법상 문제는 없으나 테이블의 특성을 가장 잘 반영하는 열을 선택하는 것이 좋다.

### CREATE TABLE에서 설정하는 기본 키 제약조건

```sql
CREATE TABLE member
( mem_id   CHAR(8) NOT NULL PRIMARY KEY,
  mem_name VARCHAR(10) NOT NULL,
  height   TINYINT UNSIGNED NULL
);
```

회원 테이블과 구매 테이블은 기본 키-외래 키로 연결되어 있다. 즉, 회원 테이블의 회원만 구매 테이블에 입력될 수 있다. 만약 구매 테이블이 있는데 회원 테이블을 삭제하면 어떻게 될까?

예를 들어, 구매 테이블의 BLK 이름과 연락처를 알고 싶어도 회원 테이블은 이미 삭제되었기 때문에 알 수 있는 방법이 없다. 그러므로 기본 키-외래 키 관계로 연결된 테이블은 외래 키가 설정된 테이블을 먼저 삭제해야 한다.

CREATE TABLE에서 기본 키를 지정하는 다른 방법은 제일 마지막 행에 PRIMARY KEY(mem_id)를 추가하는 것이다.

### ALTER TABLE에서 설정하는 기본 키 제약조건

이미 만들어진 테이블을 수정하는 ALTER TABLE 구문을 사용해서 제약조건을 설정할 수 있다.

```sql
ALTER TABLE member
			ADD CONSTRAINT
			PRIMARY KEY (mem_id);
```

1. member를 변경합니다.
2. 제약조건을 추가합니다
3. mem_id 열에 기본 키 제약조건을 설정합니다.

기본키에 이름은 다음과 같이 지정할 수 있다.

```sql
CONSTRAINT PRIMARY KEY PK_member_mem_id (mem_id)
```

## 외래 키 제약조건

외래 키 제약조건은 두 테이블 사이의 관계를 연결해주고, 그 결과 데이터의 무결성을 보장해주는 역할을 한다. 외래 키가 설정된 열은 꼭 다른 테이블의 기본 키와 연결된다.

기본 키가 있는 회원 테이블을 기준 테이블이라고 부르며, 외래 키가 있는 구매 테이블을 참조 테이블이라고 부른다.

구매 테이블의 아이디(FK)는 반드시 회원 테이블의 아이디(PK)로 존재한다. 그러므로 구매한 기록은 있으나 구매한 사람이 누구인지 모르는 심각한 일은 절대 발생하지 않는다. 구매 테이블의 데이터는 모두 누가 구매했는지 확실히 알 수 있는, 무결한 데이터가 되는 것이다.

또 하나 기억할 것은 참조 테이블이 참조하는 기준 테이블의 열은 반드시 기본 키나, 고유 키로 설정되어 있어야 한다.

### CREATE TABLE에서 설정하는 외래 키 제약조건

외래 키를 생성하는 방법은 CREATE TABLE 끝에 FOREIGN KEY 키워드를 설정하는 것이다.

```sql
CREATE TABLE buy
( ~~~
  FOREIGN KEY(mem_id) REFERENCES member(mem_id)
);
```

외래 키의 형식은 FOREIGN KEY(열_이름) REFERENCES 기준_테이블(열_이름)이다.

기준 테이블의 열 이름과 참조 테이블의 열 이름이 꼭 같아야 하는 것은 아니다.

### ALTER TABLE에서 설정하는 외래 키 제약조건

```sql
ALTER TABLE buy
		ADD CONSTRAINT
		FOREIGN KEY(mem_id)
		REFERENCES member(mem_id);
```

1. buy를 수정한다.
2. 제약조건을 추가한다
3. 외래 키 제약조건을 buy 테이블의 mem)id에 설정한다.
4. 참조할 기준 테이블의 member 테이블의 mem_id 열이다.

### 기준 테이블의 열이 변경될 경우

기준 테이블의 열이 변경되는 경우를 생각해 보자. 예를 들어 회원 테이블의 BLK가 물품을 2건 구매한 상태에서 회원 아이디를 PINK로 변경하면 어떻게 될까? 두 테이블의 정보가 일치하지 않게 된다.

기본 키-외래 키로 맺어진 후에는 기준 테이블의 열 이름이 변경되지 않는다. 열 이름이 변경되면 참조 테이블의 데이터에 문제가 발생하기 때문이다. 삭제 역시 불가능하다.

<aside>
💡 만약 참조 테이블에 데이터가 없다면 기준 테이블의 열을 변경할 수 있다.

</aside>

기준 테이블의 열을 변경하면 참조 테이블의 열도 자동으로 변경되도록 ON UPDATE CASCADE 문을 사용할 수 있다. ON DELETE CASCADE 문은 기준 테이블의 데이터가 삭제되면 참조 테이블의 데이터도 삭제되는 기능이다.

```sql
DROP TABLE IF EXISTS buy;
CREATE TABLE buy
(	num			INT AUTO_INCREMENT NOT NULL PRIMARY KEY,
	mem_id		CHAR(8) NOT NULL,
	prod_name	CHAR(6) NOT NULL
);
ALTER TABLE buy
	ADD CONSTRAINT
    FOREIGN KEY(mem_id) REFERENCES member(mem_id)
    ON UPDATE CASCADE
    ON DELETE CASCADE;
```

구매 테이블에 데이터를 다시 입력하고 회원 테이블의 BLK를 PINK로 변경해도 오류 없이 잘 변경된다.

PINK를 기준 테이블에서 삭제해도 오류 없이 잘 실행된다. 구매 테이블의 데이터를 확인하면 함께 삭제된 것을 확인할 수 있다.

## 기타 제약조건

### 고유 키 제약조건

고유 키(Unipue) 제약조건은 ‘중복되지 않는 유일한 값’을 입력해야하는 조건인다. 기본 키 제약조건과 거의 비슷하지만, 차이점은 NULL값을 허용한다는 것이다. NULL 값은 여러 개가 입력되어도 상관없다. 또, 기본 키는 테이블에 1개만 설정해야 하지만, 고유 키는 여러 개를 설정해도 된다.

만약 회원 테이블에 Email 주소가 있다면 중복되지 않으므로 고유 키로 설정할 수 있다.

### 체크 제약조건

체크 제약조건은 입력되는 데이터를 점검하느 기능을 한다. 예를 들어 평균 키에 마이너스 값이 입력되지 않도록 하는 것이다.

```sql
height TINYINT UNSIGNED NULL CHECK(height >= 100)
```

ALTER TABLE 문으로 제약조건을 추가해도 된다.

```sql
ALTER TABLE member
			ADD CONSTRAINT
			CHECK (phone1 IN('02','031','032','054','055','061'));
```

<aside>
💡 IN() 은 괄호 안에 있는 값중 하나와 같아야 참이 된다.

</aside>

### 기본값 정의

기본값 정의는 값을 입력하지 않았을 때 자동으로 입력될 값을 미리 지정해 놓는 방법이다.

```sql
height TINYINT UNSIGNED NULL DEFAULT 160
```

기본값이 설정된 열에 기본값을 입력하려면 default라고 써주고, 원하는 값을 입력하려면 해당 값을 써주면 된다.

```sql
INSERT INTO member VALUES('SPC', '우주소녀', default, default);
```

## 05-3 가상의 테이블: 뷰

뷰는 데이터베이스 객체중에 하나이다. 모든 데이터베이스 개체는 테이블과 관련이 있지만, 특히 뷰는 테이블과 아주 밀접하게 관련되어 있습니다. 뷰는 한번 생성해 놓으면 테이블이라고 생각하고 사용해도 될 정도로 사용자들의 입장에스는 테이블과 거의 동일한 개체로 취급한다.

뷰는 테이블처럼 데이터를 가지고 있지는 않다. 뷰의 실체는 SELECT 문으로 만들어져 있기 때문에 뷰에 접근하는 순간 SELECT가 실행되고 그 결과가 화면에 출력되는 방식이다. 뷰는 단순 뷰와 복합 뷰로 나뉘는데, 단순 뷰는 하나의 테이블과 연관된 뷰를 말하고, 복합 뷰는 2개 이상의 테이블과 연관된 뷰를 말한다.

### 뷰의 개념

**뷰의 기본 생성**

SELECT 문으로 회원 테이블에서 아이디, 이름 주소를 가져와서 출력한 결과를 보면 테이블의 모양을 가지고 있다. 즉 SELECT 문으로 실행해서 나온 결과를 mem_id, mem_name, addr 3개의 열을 가진 테이블로 봐도 된다.

뷰는 바로 이러한 개념이다. 그래서 뷰의 실체가 SELECT 문이 되는 것이다. 이 예제의 실행 결과를 v_member라고 부른다면, 앞으로는 v_member를 그냥 테이블이라고 생각하고 접근하면 된다.

<aside>
💡 뷰의 이름만 보고도 뷰인지 알아볼 수 있도록 이름 앞에 v_를 붙이는 것이 일반적이다.

</aside>

뷰를 만드는 형식은 다음과 같다

```sql
CREATE VIEW 뷰_이름
AS
	SELECT 문;
```

뷰를 만든 후에 뷰에 접근하는 방식은 테이블과 동일하게 SELECT 문을 사용한다. 전체에 접근할 수도 있고, 필요하면 조건식도 테이블과 동일하게 사용할 수 있다.

```sql
SELECT 열_이름 FROM 뷰_이름
	[WHERE 조건];
```

**뷰의 작동**

사용자는 뷰를 테이블이라고 생각하고 접근한다. 그러면 MySQL이 뷰 안에 있는 SELECT를 실행해서 그 결과를 사용자에게 보내주므로 사용자 입장에서는 뷰에서 모두 처리된 것으로 이해한다.

뷰는 수정이 가능할까? 기본적으로 읽기 전용으로 사용되지만, 뷰를 통해서 원본 테이블의 데이터를 수정할 수도 있다. 하지만 무조건 가능한 것은 아니고 몇 가지 조건을 만족해야 한다.

**뷰를 사용하는 이유**

**보안에 도움이 된다**

앞의 예에서 만든 v_member 뷰에는 사용자의 아이디, 이름, 주소만 있을 뿐 사용자의 중요한 개인정보인 연락처, 평균 키 등의 정보는 들어있지 않다. 이런 방식으로 사용자마다 테이블에 접근하는 권한에 차별을 둬서 처리하고 있으며, 사용자별 권한이 데이터베이스 보안의 중요한 주제 중 하나이다.

**복잡한 SQL을 단순하게 만들 수 있다.**

```sql
SELECT B,mem_id, M.mem_name, B.prod_name, M.addr,
								CONCAT(M.phone1, M.phone2) '연락처'
		FROM buy B
			INNER JOIN member M
			ON B,mem_id = M.mem_id;
```

내용이 길고 복잡하다. 이 SQL을 다음과 같이 뷰로 생성해 놓고 사용자들은 해당 뷰에만 접근하도록 하면 복잡한 SQL을 입력할 필요가 없어진다.

```sql
CREATE VIEW v_memberbuy
AS
		SELECT B.mem_id, M.mem_name, B.prod_name, M.addr,
								CONCAT(M.phone1, M.phone2) '연락처'
				FROM buy B
							INNER JOIN member M
							ON B.mem_id = M.mem_id;
```

이제부터는 v_memberbuy를 테이블이라 생각하고 접근하면 된다.
```sql
SELECT * FROM v_memberbuy WHERE mem_name = '블랙핑크';
```

### 뷰의 실제 작동

뷰의 단순한 형태에 대해 살펴보았는데, 실무에서는 좀 더 복잡하게 사용한다. 실제로 사용하는 방법을 익혀보자.

**뷰의 실제 생성, 수정, 삭제**

기본적인 뷰를 생성하면서 뷰에 사용될 열 이름을 테이블과 다르게 지정할 수도 있다. 기존에 배운 별칭을 사용하면 되는데, 중간에 띄어쓰기 사용이 가능하다. 별칭은 열 이름 뒤에 작은따옴표 또는 큰따옴표로 묶어주고, 형식상 AS를 붙여준다.

단, 뷰를 조회할 때는 열 이름에 공백이 있으면 백틱(`)으로 묶어줘야 한다.

```sql
USE market_db;
CREATE VIEW v_viewtest1
AS
			SELECT B.mem_id 'Member ID', M.mem_name AS 'Member Name',
					B.prod_name "Product Name",
											CONCAT(M.phone1, M.phone2) AS "office Phone"
					FROM buy B
								INNER JOIN member M
								ON B.mem_id = M.mem_id;
SELECT DISTINCT `Member ID`, `Member Name` FROM v_viewtest1; ->백틱 사용
```

뷰의 수정은 ALTER VIEW 구문을 사용하며, 열 이름에 한글을 사용해도 된다.

```sql
ALTER VIEW v_viewtest1
AS
			SELECT B.mem_id '회원 아이디', M.mem_name AS '회원 이름',
					B.prod_name "제품 이름",
											CONCAT(M.phone1, M.phone2) AS "연락처"
					FROM buy B
								INNER JOIN member M
								ON B.mem_id = M.mem_id;
SELECT DISTINCT `회원 아이디`, `회원 이름` FROM v_viewtest1; ->백틱 사용
```

<aside>
💡 열 이름에 한글을 사용하는 것은 권장하지 않는다.

</aside>

뷰의 삭제는 DROP VIEW를 사용한다.
**뷰의 정보 확인**

기존에 생성된 뷰에 대한 정보를 확인할 수 있다.

```sql
USE market_db;
CREATE OR REPLACE VIEW v_viewtest2
AS
			SELECT mem_id. mem_name, addr FROM member;
```

DESCRIBE 문으로 기존 뷰의 정보를 확인할  수 있다.

```sql
describe v_viewtest2;
```

뷰도 테이블과 동일하게 정보를 보여준다. 주의할 점은 PK 등의 정보는 확인되지 않는다.

SHOW CREATE VIEW 문으로 뷰의 소스 코드도 확인할 수있다.

```sql
SHOW CREATE VIEW v_viewtest2;
```

**뷰를 통한 데이터의 수정/삭제**

뷰를 통해서 테이블의 데이터를 수정할 수도 있다. v_member 뷰를 통해 데이터를 수정해보자.

```sql
UPDATE v_member SET addr = '부산' WHERE mem_id='BLK';
```

이번에는 데이터를 입력해보자.

```sql
insert into v_member(mem_id, mem_name, addr) values('BTS', '방탄소년단','경기');
```

Error Code: 1423. Field of view 'market_db.v_member' underlying table doesn't have a default value	

v_member(뷰)가 참조하는 member의 열 중에서 mem_number 열은 NOT NULL로 설정되어서 반드시 입력해줘야 한다. 하지만 뷰에서는 mem_number 열을 참조하고 있지 않으므로 값을 입력할 방법이 없다. 만약 v_member 뷰를 통해서 member 테이블에 값을 입력하고 싶다면 v_member에 mem_number 열을 포함하도록 뷰를 재정의하거나, 아니면 member에서 mem_number 열의 속성을 ULL로 바꾸거나, 기본값을 지정해야 한다.

이번에는 지정한 범위로 뷰를 생성해보자. 평균 키가 167 이상인 뷰를 생성한다.
```sql
create view v_height167
as
		select * from member where height >= 167;
	
select * from v_height167;
```

이번에는 v_height167 뷰에서 키가 167 미만인 데이터를 입력해보자.

```sql
insert into v_height167 values('TRA','티아라','6','서울',null,null,159,'2005-01-01');
```

일단 입력은 된다. 그런데 v_height167 뷰는 167 이상만 보이도록 만든 뷰인데, 167 미만인 데이터가 입력되었다. 뷰의 데이터를 확인해보면 방금 입력한 데이터는 보이지 않는다. 이를 보면 이 뷰에서 키가 159인 데이터를 입력한 것은 별로 바람직해 보이지 않는다. 즉, 예상치 못한 경로를 통해서 입력되면 안 되는 데이터가 입력된 듯한 느낌이다. 키가 167 이상인 뷰이므로 167 이상의 데이터만 입력되도록 하는 것이 논리적으로 바람직하다.

이럴 때 예약어 WITH CHECK OPTION을 통해 뷰에 설정된 값의 범위가 벗어나는 값은 입력되지 않도록 할 수 있다.

```sql
alter view v_height167
as
		select * from member where height >= 167
				with check option;
```

이제 키가 167 미만인 데이터는 입력되지 않고, 167 이상의 데이터만 입력된다. 이러한 방식이 좀 더 합리적이다.

**단순 뷰와 복합 뷰**

하나의 테이블로 만든 뷰를 단순 뷰라 하고, 두 개 이상의 테이블로 만든 뷰를 복합 뷰라고 한다. 복합 뷰는 주로 두 테이블을 조인한 결과를 뷰로 만들 때 사용한다. 복합 뷰의 예는 다음과 같다.

```sql
create view v_complex
as
		select B.mem_id, M.mem_name, B.prod_name, M.addr
			from buy B
				INNER JOIN member M
                ON B.mem_id = M.mem_id;
```

복합 뷰는 읽기 전용이다. 복합 뷰를 통해 테이블에 데이터를 입력, 수정, 삭제할 수 없다.

**뷰가 참조하는 테이블의 삭제**

뷰가 참조하는 테이블을 삭제해보자

```sql
drop table if exists buy, member;
```

현재 여러 개의 뷰가 두 테이블과 관련이 있는데도 테이블이 삭제되었다. 두 테이블 중 아무거나 연관되는 뷰를 다시 조회해보자.

```sql
select * from v_height167;
```

당연히 참조하는 테이블이 없기 때문에 조회할 수없다는 메시지가 나온다. 바람직하지는 않지만, 관련 뷰가 있더라도 테이블은 쉽게 삭제된다.

뷰가 조회되지 않으면 check table 문으로 뷰의 상태를 확인해볼 수 있다. 뷰가 참조하는 테이블이 없어서 오류가 발생하는 것을 확인할 수 있다.