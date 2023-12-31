# Chapter 02. 실전용 SQL 미리 맛보기

INSERT 데이터 입력 UPDATE 데이터 수정 DELETE 데이터 삭제 SELECT 데이터 조회

WHERE: SELECT문에서 특정 조건을 조회할 때 사용하는 구문

## 2-3. 데이터베이스 개체

데이터베이스에는 테이블, 인덱스, 뷰, 스토어드 프로시저, 트리거, 함수, 커서 등의 개체가 필요하다.

### 인덱스

데이터를 조회할 때 테이블에 데이터가 적다면 결과가 금방 나오지만 데이터가 많아질수록 결과가 나오는 시간이 많이 소요된다. 인덱스는 이런 경우 결과가 나오는 시간을 대폭 줄여준다.

### 뷰

뷰는 테이블과 상당히 동일한 성격의 데이터베이스 개체이다. 뷰를 활용하면 보안도 강화하고, SQL 문도 간단하게 사용할 수 있다.

뷰를 한마디로 정의하면 가상의 테이블이라고 할 수 있다. 일반 사용자의 입장에서는 테이블과 뷰를 구분할 수 없다. 일반 사용자는 테이블과 동일하게 뷰를 취급하면 된다. 다만 뷰는 실제 데이터를 가지고 있지 않으며, 진짜 테이블에 링크된 개념이라고 생각하면 된다.

뷰는 윈도우의 ‘바로가기 아이콘’과 비슷한 개념이다. 실체는 없으며 테이블과 연결되어 있는 것뿐이다. 뷰의 실체는 SELECT문이다.

뷰를 사용하면 보안에 도움이 되고, 긴 SQL 문을 간략하게 만들 수 있다.

### 스토어드 프로시저

스토어드 프로시저를 통해 SQL 안에서도 일반 프로그래밍 언어처럼 코딩을 할 수 있다.

MySQL에서 제공하는 프로그래밍 기능으로, 여러개의 SQL문을 하나로 묶어서 편리하게 사용할 수 있다. 그 외에 프로그래밍 언어에서 사용되는 연산식, 조건문, 반복문 등을 사용할 수도 있다.

```sql
DELIMITER //
**create procedure myProc()
 Begin
  select * from member where member_name = '나훈아';  
	select * from product where product_name = '삼각김밥'; 
end
DELIMITER ;**
```

<aside>
📌 데이터베이스 개체를 만들기 위해서는 **CREATE 개체종류 개체이름 ~~** 형식을 사용한다.  반대로 데이터베이스 개체를 삭제하기 위해서는 **DROP 개체종류 개체이름** 형식을 사용한다.

</aside>