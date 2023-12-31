# Chapter 06. 인덱스

## 06-1 인덱스 개념

### 시작하기 전에

인덱스는 데이터를 빠르게 찾을 수있도록 도와주는 도구로, 실무에서는 인덱스 없이 데이터베이스 운영이 불가능하다.

인덱스에는 클러스터형 인덱스와 보조 인덱스가 있다. 클러스터형 인덱스는 기본 키로 지정하면 자동 생성되며 테이블에 1개만 만들 수 있다. 기본 키로 지정한 열을 기준으로 자동 정렬된다. 보조 인덱스는 고유 키로 지정하면 자동 생성되며 여러 개를 만들 수도 있지만 자동 정렬되지는 않는다.

### 인덱스의 개념

인덱스는 데이터를 빠르게 찾을 수 있도록 해주는 도구이다. 실무에서 윤영하는 테이블에서는 인덱스의 사용 여부에 따라 성능 차이가 날 수 있다. 대용량의 테이블일 경우에는 더욱 그러하며, 이것이 인덱스를 사용하는 이유이다.

**인덱스의 문제점**

인덱스에도 문제점이 있다. 필요없는 인덱스를 만들 경우 데이터베이스가 차지하는 공간만 더 늘어나고, 인덱스를 이용해서 데이터를 찾는 것이 전체 테이블을 찾아보는 것보다 느려진다.

<aside>
💡 데이터베이스에 인덱스를 생성해 놓아도, 인덱스를 사용해서 검색하는 것이 빠를지 아니면 전체 테이블을 검색하는 것이 빠를지 MySQL이 알아서 판단한다. 만약 인덱스를 사용하지 않는다면 쓸데없이 공간을 낭비한 것이다
</aside>
<br></br>

**인덱스의 장점과 단점**

인덱스를 SELECT에서 즉각적인 효과를 내는 빠른 방법 중 한 가지이다. 적절한 인덱스를 생성하고 인덱스를 사용하는 SQL을 만든다면 기존보다 아주 빠른 응답 속도를 얻을 수 있다.

인덱스의 장점은 다음과 같다.

- SELECT 문으로 검색하는 속도가 매우 빨라진다
- 전체 시스템의 성능이 향상된다

인덱스의 단점은 다음과 같다

- 인덱스도 공간을 차지해서 데이터베이스 안에 추가적인 공간이 필요하다.(테이블 크기의 약 10%)
- 처음에 인덱스를 만드는 데 시간이 오래 걸릴 수 있다.
- SELECT가 아닌 데이터의 변경이 자주 일어나면 오히려 성능이 나빠질  수 있다.

### 인덱스의 종류

MySQL에서 사용되는 인덱스의 종류는 크게 두 가지로 나뉘는데, 클러스터형 인덱스와 보조 인덱스이다. 이 두 개를 쉽게 비교하면 클러스터형 인덱스는 사전과 같고, 보조 인덱스는 책의 뒤에 찾아보기가 있는 일반적인 책과 같다.

보조 인덱스는 책과 같이 찾아보기가 별도로 있고, 찾아보기에서 해당 언어를 찾은 후에 옆에 표시된 페이지를 펼쳐야 실제 찾는 내용이 있는 것을 말한다. 클러스터형 인덱스는 영어사전처럼 책의 내용이 이미 알파벳 순서대로 정렬되어 있는 것이다. 그래서 별도의 찾아보기가 없다. 책 자체가 찾아보기이다.

**자동으로 생성되는 인덱스**

인덱스는 테이블의 컬럼 단위에 생성되며, 하나의 열에는 하나의 인덱스를 생성할 수 있다.

기본 키로 지정하면 자동으로 컬럼에 클러스터형 인덱스가 생성된다. 그런데 기본 키는 테이블에 하나만 지정할 수 있다. 결국 클러스터형 인덱스는 테이블에 한 개만 만들 수 있다는 것이다.

한번 확인해 보자. table1을 생성하고 아래와 같이 실행하면 된다.

```sql
show index from table1;
```

![Untitled](/assets/혼공sql19.png)

 Key_name이 PRIMARY이다.  이것은 기본 키로 설정해서 자동으로 생성된 인덱스라는 의미이다. 이것이 바로 클러스터형 인덱스이다. Column_name이 col1로 설정되어 있다는 것은 col1 열에 인덱스가 만들어져 있다는 말이다. Non_unipue는 고유하지 않다라는 뜻이다. 즉 중복이 허용되냐는 뜻이다. 0이라는 것은 False, 1은 True의 의미이다. 결론적으로 이 인덱스는 중복이 허용되지 않는 인덱스이다.

기본 키와 더불어 고유 키도 인덱스가 자동으로 생성된다. 고유 키로 생성되는 인덱스는 보조 인덱스이다.

```sql
create table table2(
	col1 int primary key,
	col2 int unique,
	col3 int unique
);
show index from table2;
```

![Untitled](/assets/혼공sql20.png)

Key_name에 col2, col3가 써져 있다. 이렇게 열 이름이 써 있는 것은 보조 인덱스라고 보면 된다. 고유 키 역시 중복값을 허용하지 않는다. 또 고유 키를 여러 개 지정할 수 있듯이 보조 인덱스도 여러개 만들 수 있다.

**자동으로 정렬되는 클러스터형 인덱스**

클러스터형 인덱스는 기본 키로 지정하면 자동 생성된다. 그리고 테이블에 1개만 생선도니다. 이번에는 클러스터형 인덱스의 특징을 파악해보자.

클러스터형 인덱스를 영어사전과 비교해서 설명했다. 영어사전의 가장 큰 특징은 단어가 알파벳 순서로 정렬되어 있다는 것이다. 마찬가지로 어떤 열을 기본 키로 지정하면(클러스터형 인덱스가 생성되면) 그 열을 기준으로 자동 정렬된다.

먼저 테이블을 생성하고 값을 입력한 후, 어떤 열을 기본 키로 설정하면 해당 열에 클러스터형 인덱스가 생성되어 해당 열을 기준으로 정렬된다.

추가로 데이터를 입력하면 알아서 기준에 맞춰서 정렬된다.

<aside>
💡 이미 대용량의 데이터가 있는 상태에서 기본 키를 지정하면 시간이 엄청 오래 걸릴 수 있다. 노트에 단어가 4개라면 정렬하는 데 금방이지만, 노트에 단어가 4만개라면 그것을 정렬하는 데는 오랜 시간이 소요된다.

</aside><br></br>

**정렬되지 않는 보조 인덱스**

고유 키로 지정하면 보조 인덱스가 생성된다. 그리고 보조 인덱스는 테이블에 여러 개 설정할 수 있다. 고유 키를 테이블에 여러 개 지정할 수 있는 것과 마찬가지이다.

보조 인덱스를 생성해도 데이터의  순서는 변경되지 않고 별도로 인덱스를 만드는 것이다. 데이터를 추가로 입력해도 제일 뒤에 추가된다. 보조 인덱스는 여러 개를 만들 수 있다. 하지만 보조 인덱스를 만들 때마다 데이터베이스의 공간을 차지하게 되고, 전반적으로 시스템에 오히려 나쁜 영향을 미치게 된다. 그러므로 꼭 필요한 열에만 적절히 보조 인덱스를 생성하는 것이 좋다.

## 06-2 인덱스의 내부 작동

클러스터형 인덱스와 보조 인덱스는 모두 내부적으로 균형 트리로 만들어진다. 균형 트리(Balanced tree, B-tree)는 자료구조에 나오는 범용적으로 사용되는 데이터의 구조이다.

### 인덱스의 내부 작동 원리

인덱스의 내부 작동 원리를 이해하면, 인덱스를 사용해야 할 경우와 사용하지 말아햐 할 경우를 선택할 때 도움이 된다.

**균형 트리의 개념**

균형 트리 구조에서 데이터가 저장되는 공간을 노드라고 한다. 루트 노드는 노드의 가장 상위 노드를 말한다. 리프 노드는 제일 마지막에 존재하는 노드를 말한다. 중간에 끼인 노드들은 중간 노드라 부른다. MySQL에서는 페이지라고 부른다. 균형 트리는 데이터를 검색할 때 아주 뛰어난 성능을 발휘한다.

균형 트리는 무조건 루트 페이지부터 검색한다. 모든 데이터는 정렬되어 있기 때문에 더 적은 페이지를 읽고 데이터를 검색할 수 있게 된다.

**균형 트리의 페이지 분할**

인덱스를 구성하면 데이터 변경 작업시 성능이 나빠진다. 특히 INSERT 작업이 일어날 때 더 느리게 입력될 수 있다. 이유는 페이지 분할이라는 작업이 발생하기 때문이다. 페이지 분할이란 새로운 페이지를 준비해서 데이터를 나누는 작업을 말한다. 페이지 분할이 일어나면 MySQL이 느려지고, 너무 자주 일어나면 성능에 큰 영향을 준다.

### 인덱스의 구조

인덱스 구조를 통해 인덱스를 생성하면 왜 데이터가 정렬되는지, 어떤 인덱스가 더 효율적인지 살펴보자.

**클러스터형 인덱스 구성하기**

테이블을 생성하고 데이터를 입력한 뒤  기본 키를 지정하면 클러스터형 인덱스가 생성되면서 데이터 페이지가 정렬되고 균형 트리 형태의 인덱스가 형성된다. 클러스터형 인덱스는 루트 페이지와 리프 페이지(중간 페이지가 있다면 중간 페이지도 포함)로 구성되어 있다. 또한 인덱스 페이지의 리프 페이지는 데이터 그 자체이다. 리프 페이지 = 데이터 페이지

**보조 인덱스 구성하기**

고유 키를 지정하면 보조 인덱스가 생성된다.  보조 인덱스는 데이터 페이지를 건드리지 않는다. 그리고 별도의 장소에 인덱스 페이지를 생성한다. 책의 찾아보기가 별도의 공간에 만들어진 것처럼 보조 인덱스가 별도의 공간에 만들어진다.

우선 인덱스 페이지의 리프 페이지에 인덱스로 구성한 열을 정렬한다. 그리고 실제 데이터가 있는 위치를 준비한다. 데이터의 위치는 **페이지 번호 +#위치**로 구성되어 있다.

**인덱스에서 데이터 검색하기**

이제 데이터를 검색해보자.  인덱스 페이지는 루트 페이지와 리프 페이지로 구성된다고 가정하자.

클러스터형 인덱스는 루트 페이지를 읽고, 리프페이지, 즉 데이터 페이지를 읽으면 데이터를 찾을 수 있다. 총 2개 페이지를 읽어서 찾은것이다. 보조 인덱스는 루트 페이지를 읽고, 리프 페이지를 읽은 후 해당 데이터 페이지를 읽어서 데이터를 찾아낸다. 총 3개의 페이지를 읽어서 찾은 것이다. 두 인덱스 모두 검색이 빠르기는 하지만 클러스터형 인덱스가 조금 더 빠르다.
## 06-3 인덱스의 실제 사용

인덱스를 생성하기 위해서는 CREATE INDEX 문을 사용하고, 제거하기 위해서는 DROP INDEX 문을 사용한다.

```sql
CREATE [UNIQUE] INDEX 인덱스_이름
	ON 테이블_이름 (열_이름) [ASC | DESC]
```

```sql
DROP INDEX 인덱스_이름 ON 테이블_이름
```

### 인덱스 생성과 제거 문법

**인덱스 생성 문법**

직접 인덱스를 생성하려면 CREATE INDEX 문을 사용해야 한다.

UNIQUE는 중복이 안 되는 고유 인덱스를 만드는 것인데, 생략하면 중복이 허용된다. CREATE UNIQUE로 인덱스를 생성할려면 기존에 입력된 값들에 중복이 있으면 안된다. 그리고 인덱스를 생성한 후에 입력되는 데이터와도 중복될 수없으니 신중해야 한다.

예를 들어, 회원 이름을 UNIQUE로 지정하면 향후에는 같은 이름의 회원은 입력할 수 없게 된다.

ASC 또는 DESC는 인덱스를 오름차순 또는 내림차순으로 만들어준다. 기본은 ASC로 만들어지며, DESC로 만드는 경우는 거의 없다.

**인덱스 제거 문법**

DROP INDEX로 제거한다. 주의할 점은 기본 키, 고유 키로 자동 생성된 인덱스는 DROP INDEX로 제거하지 못한다. ALTER TABLE 문으로 기본 키나 고유 키를 제거하면 자동으로 생성된 인덱스도 제거할 수 있다.

> 하나의 테이블에 클러스터형 인덱스와 보조 인덱스가 모두 있는 경우, 인덱스를 제거할 때는 보조 인덱스부터 제거하는 것이 더 좋다. 클러스터형 인덱스부터 제거하면 내부적으로 데이터가 재구성되기 때문이다. 또한, 인덱스가 많이 생성되어 있는 테이블의 경우 사용하지 않는 인덱스는 제거해 주는 것이 좋다.
> 

### 인덱스 생성과 제거 실습

```sql
show index from member //member 테이블의 인덱스 확인
show table status like 'member'  //테이블에 생성된 인덱스의 크기 확인 가능
```

```sql
create index idx_member_addr
		on member(addr);      //member의 addr에 중복을 허용하는 단순 보조 인덱스 생성
```

> 클러스터형 인덱스와 보조 인덱스가 동시에 있다는 것은 영어사전이면서 동시에 찾아보기도 존재한다는 것이다. 영어사전에 추가로 한글 동물, 한글 식물 단어를 찾아보기로 정리해 놓은 것으로 생각하면 된다. 예를 들어 영어 단어를 찾을 때는 본문(클러스터형 인덱스)을 찾으면 되고, 한글 단어(동물, 식물)를 찾을 때는 찾아보기를 이용하면 된다.
> 

보조 인덱스가 추가되었으므로 전체 인덱스의 크기를 다시 확인해보자. Index_length 부분이 보조 인덱스의 크기인데, 이상하게 크기가 0으로 나왔다.

생성된 인덱스를 실제로 적용시키려면 ANALYZE TABLE 문으로 먼저 테이블을 분석/처리해줘야 한다. 이제 보조 인덱스가 생성된 것이 확인된다.

중복된 값이 있는 컬럼에 중복을 허용하지 않는 고유 보조 인덱스를 생성할 수 없다.

고유 보조 인덱스가 생성된 컬럼에는 중복된 값을 입력할 수없다. 업무상 절대로 중복되지 않는 열에만 UNIQUE 옵션을 사용해서 인덱스를 생성해야 한다.

> 중복된 데이터가 많은 열도 종종 있다. 중복된 데이터가 많은 열에 인덱스를 생성하는 것은 의미도 없고 오히려 성능에 나쁜 영향을 미친다.
> 

**인덱스의 활용 실습**

생성된 인덱스를 활용해보자. SELECT를 사용해서 인덱스가 있으면 인덱스를 통해서 결과를 출력하고, 인덱스가 없으면 전체 테이블을 찾아서 결과를 출력했다. 일반 사용자 입장에서는 결과의 차이는 없으며 빠르게 결과를 보느냐, 느리게 결과를 보느냐의 차이뿐이다. 하지만 인덱스를 이해하지 못하면 빠른 결과를 내야 할 때 제대로 활용하지 못할 수도 있다. 현재 mem_id, mem_name, addr 열에 인덱스가 생성되어 있다.

이번에는 전체를 조회해보자. 그런데 이 SQL은 인덱스와 아무런 상관이 없다. 인덱스를 사용하려면 인덱스가 생성된 열 이름이 SQL 문에 있어야 한다.

인덱스를 사용했는지 여부는 결과중 [Execution Plan] 창을 확인하면 된다. 전체 테이블이 검색을 한 것이 확인된다. 책과 비교하면 첫 페이지부터 끝 페이지까지 넘겨본 것이다.

이번에는 인덱스가 있는 열을 조회해 보자.

조회는 잘 되지만 [Execution Plan] 창을 확인해 보면 이번에도 전체 테이블 검색을 했다. 열 이름이 SELECT 다음에 나와도 인덱스를 사용하지 않는다.

이번에는 인덱스가 생성된 mem_name 값이 ‘에이핑크’인 행을 조회해보자. 결과에 에이핑크가 나왔을 것이다.

```sql
select mem_id, mem_name, addr
		from member
		where mem_name = '에이핑크';
```

[Execution Plan] 창을 확인해 보면 Single Row(constant)라고 되어 있다. 인덱스를 사용해서 결과를 얻었다는 의미이다.

> Full Table Scan을 제외하고, 나머지는 모두 인덱스를 사용했다는 의미이다. 인덱스를 사용하는 방법이 여러 개라서 다양한 용어로 표현된다.
> 

이번에는 숫자의 범위로 조회해보자. 먼저 숫자로 구성된 인원수(mem_number)로 단순 보조 인덱스를 만들어보자

```sql
create index idx_member_mem_number
		on member(mem_number);
analyze table member;
```

인원수가 7명 이상인 그룹의 이름과 인원수를 조회해보자. 결과는 4건이 나왔을 것이다.

```sql
select mem_name, mem_number
from member
where mem_number >= 7;
```

[Execution Plan] 창을 확인해 보면 인덱스를 사용한 것을 확인할 수 있다.

**인덱스를 사용하지 않을 때**

인덱스가 있고 WHERE 절에 열 이름이 나와도 인덱스를 사용하지 않는 경우가 있다. 인원수가 1명 이상인 회원을 조회해보자. 회원은 1명 이상이므로 10건 모두 조회된다.

이제 [Execution plan] 창을 살펴보자. 전체 테이블 검색을 했다. 앞에서 7명 이상일 때는 인덱스를 사용해는데, 1명 이상으로 설정하니 전체 테이블 검색을 했다 .인덱스가 있더라도 MySQL 이 인덱스 검색보다는 전체 테이블 검색이 낫겠다고 판단했기 때문이다. 이 경우에는 대부분의 행을 가져와야 하므로 인덱스를 왔다갔다 하는 것보다는 차라리 테이블을 차례대로 읽는 것이 효율적이다.

> 인덱스가 있어도 MySQL이 판단해서 사용하지 않을 수 있다
> 

또 다른 경우를 살펴보자

```sql
select mem_name, mem_number
	from member
where mem_number*2 >= 14;
```

where 문에서 열에 연산이 가해지면 인덱스를 사용하지 않는다. 이런 경우에는 다음과 같이 수정하면 된다.

```sql
select mem_name, mem_number
	from member
where mem_number >= 14/2;
```

where 절에 나온 열에는 아무런 연산을 하지 않는 것이 좋다.

**인덱스 제거 실습**

클러스터형 인덱스와 보조 인덱스가 섞여 있을 때는 보조 인덱스를 먼저 제거하는 것이 좋다. 클러스터형 인덱스를 먼저 제거해도 되지만, 데이터를 쓸데없이 재구성해서 시간이 더 오래 걸린다.

```sql
drop index idx_member_mem_name on member;
drop index idx_member_addr on member;
drop index idx_member_mem_number on member;
```

마지막으로 기본 키 지정으로 자동 생성된 클러스터형 인덱스를 제거하면 된다. Primary Key에 설정된 인덱스는 DROP INDEX 문으로 제거되지 않고 ALTER TABLE 문으로만 제거할 수 있다.

```sql
alter table member
	drop primary key;
```

오류가 발생했다. 이유는 member의 mem_id 열을 buy가 참조하고 있기 때문이다. 그러므로 기본 키를 제거하기 전에 외래 키 관계를 제거해야 한다.

테이블에는 여러 개의 외래 키가 있을 수 있다. 그래서 먼저 외래 키의 이름을 알아내야 한다.

information__schema 데이터베이스의 referential_constraints 테이블을 조회하면 외래 키의 이름을 알 수 있다.

```sql
select table_name, constraint_name
	from information_schema.referential_constraints
    where constraint_schema = 'market_db';
```

외래 키의 이름이 buy_ibfk_1인것을 알았다. 외래 키를 먼저 제거하고 기본 키를 제거하면 된다.

### 인덱스를 효과적으로 사용하는 방법

**인덱스는 열 단위에 생성된다**

하나의 열에 하나의 인덱스를 생성할 수 있다. 사실 하나의 열에 2개 이상의 인덱스를 만들 수도 있고, 2개 이상의 열을 묶어서 하나의 인덱스로 만들 수도 있다. 하지만 이런 경우는 드물기 때문에 하나의 열에 하나의 인덱스를 만드는 것이 가장 일반적이다.

**WHERE 절에서 사용되는 열에 인덱스를 만들어야 한다**

SELECT 문을 사용할 때, WHERE 절의 조건에 해당 열이 나와야 인덱스를 사용한다.

```sql
select mem_id, mem_name, mem_number, addr
	from member
	where mem_name = '에이핑크'
```

만약 member 테이블에는 단지 이 SQL만 사용한다고 가정하면, 이 SQL에서 mem_id, mem_number, addr 열에는 인덱스를 생성해도 전혀 사용하지 않는다. where 절에 있는 mem_name 열만 인덱스를 사용한다. 그러므로 다른 열에 인덱스를 만드는 것은 낭비가 된다.

**WHERE 절에 사용되더라도 자주 사용해야 가치가 있다**

만약 mem_name 열에 인덱스를 생성해서 효율이 아주 좋아진다고 하더라도, 이 select 문은 1년에 ㅎ1번 정도만 사용되고 member 테이블에는 주로 insert 작업만 일어난다면 어떨까? 인덱스는 insert의 성능을 나쁘게 한다. 그러므로 오히려 인덱스로 인해서 성능이 나빠지는 결과가 될 것이다.

**데이터의 중복이 높은 열은 인덱스를 만들어도 별 효과가 없다**

열에 들어갈 데이터의 종류가 몇 가지 되지 않으면 인덱스가 큰 효과를 내지 못한다. 예를 들어 성별, 연락처 국번, 주로 사용하는 교통 수단 등 종류가 제한된 것에는 인덱스를 만들어도 효과가 없다.

**클러스터형 인덱스는 테이블당 하나만 생성할 수 있다**

클러스터형 인덱스는 데이터 페이지를 읽는 수가 보조 인덱스보다 적기 때문에 성능이 더 우수하다. 그러므로 하나밖에 지정하지 못하는 클러스터형 인덱스(기본 키)는 조회할 때 가장 많이 사용되는 열에 지정하는 것이 효과적이다.

**사용하지 않는 인덱스는 제거한다**

실제로 사용되는 SQL을 분석해서 WHERE 조건에서 사용되지 않는 열의 인덱스는 제거할 필요가 있다. 그러면 공간을 확보할 뿐 아니라 데이터 입력 시 발생되는 부하도 많이 줄일 수 있다.