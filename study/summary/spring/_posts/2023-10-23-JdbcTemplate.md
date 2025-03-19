---
layout: post
tags: Spring
title: JdbcTemplate
---

순수 **JDBC** 인터페이스를 활용하여 데이터에 접근하다보면 다음 코드들이 매번 반복된다.

- 커넥션 조회, 커넥션 동기화
- `PreparedStatement` 생성 및 파라미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생 시 예외 변환
- 리소스 종료

**Spring**에서는 이 반복 작업을 효과적으로 처리할 수 있도록 `JdbcTemplate`을 제공한다. 

# 설정

`JdbpTemplate`은 `spring-jdbc`라이브러리에 기본으로 포함되어 있으므로 스프링을 이용해 JDBC를 사용한다면 별도의 설정 없이 바로 사용할 수 있다.

# 사용

### 준비

`DataSource` 구현체를 생성자의 파라미터로 넘겨주어 `JdbcTemplate` 인스턴스를 생성한다. 

```java
JdbcTemplate template = new JdbcTemplate(dataSource);
```

### Select

`JdbcTemplate.query*` 형태의 여러가지 함수들이 존재하여 쿼리 결과를 다양한 방식으로 처리할 수 있다. 그중 하나의 방법은 `RowMapper<T>` 클래스와 `JdbcTemplate.queryForObject` 메소드를 활용하는 방식이다. 

```java
String sql = "select * from item where id = ?"
RowMapper<Item> rowMapper = (rs, rowNum) -> {
	Item item = new Item();
	item.setId(rs.getLong("id"));
	item.setItemName(rs.getString("item_name"));
	item.setPrice(rs.getInt("price"));
	return item;
}

try {
	Item item = template.queryForObject(sql, rowMapper, id);
	// do something with item
} catch (EmptyResultDataAccessException e) {
	// catch when there is no data
}
```

`RowMapper<T>`는 쿼리 결과를 담은 `ResultSet`과 각 레코드의 인덱스를 파라미터로 받아 `T`타입의 오브젝트를 반환하는 함수이다. 

쿼리 실행결과 결과값이 없으면 `EmptyResultDataAccessException`가 발생한다. 이 예외는 `SQLException`과 다르게 `DataAccessException`을 상속하는 런타임 예외이므로 반드시 처리해야 하는 것은 아니다. 

### Insert

간단한 insert 작업은 `JdbcTemplate.update()` 메소드를 사용하면 된다.

```java
String sql = "insert into member(member_id, money) values(?, ?)";
int count = template.update(sql, member.getMemberId(), member.getMoney());
```

sql과 파라미터 값들을 넘겨주고, 리턴값으로는 변경된 레코드 수를 반환한다.

##### 데이터베이스 자동 생성 키 확인

데이터베이스에서 키를 자동으로 생성해 주는 경우에는 `KeyHolder`를 사용해야 한다. 

```java
String sql = "insert into item values (?, ?)";
KeyHolder keyHolder = new GeneratedKeyHolder();

template.update(connection -> {
	PreparedStatement pstmt = connection.prepareStatement(sql, new String[]{"id"});
	ps.setString(1, item.getItemName());
	ps.setInt(2, item.getPrice());
	return ps;
}, keyHolder);

long key = keyHolder.getKey().longValue();
// use key
```

`PreparedStatementCreator`를 사용하는 `update()`함수를 이용해서 `PreparedStatement`에 키 컬럼명을 지정해주고 `KeyHolder` 오브젝트를 파라미터로 같이 넣어주면 업데이트 후 `KeyHolder`에 키값이 저장된다. 

키는 내부적으로 `Statement.getGeneratedKeys()` 메소드를 호출하여 얻는다. 자세한 내용은 [[JDBC 생성 키값 확인하기]]를 참고하자.

> [!info] 자동생성 키를 얻는 좀더 쉬운 방법
> [[NamedParmeterJdbcTemplate]]를 사용하면 `PreparedStatement`를 직접 생성하지 않고도 자동생성 키를 얻는 API를 사용할 수 있다.

### update

insert와 동일한 방식으로 수행한다.

```java
String sql = "update item set price = ? where id = ?";
int count = template.update(sql, newPrice, item.id);
```

### 동적 쿼리 생성

`JdbcTempate`만으로 동적 쿼리를 생성하는 것은 상당히 복잡하다. 따라서 동적 쿼리를 생성해야 한다면 [[MyBatis]]를 사용하는 것을 고려해보자.

### 파라미터 바인딩에 이름 사용

`JdbcTemplate`의 메소드들은 파라미터를 바인딩할 때 sql의 바인딩 위치 순서대로 파라미터를 바인딩한다. 이 방식은 간편하지만 암시적이어서 실수가 발생할 가능성이 있다.

이를 해결하기 위해 파라미터의 이름을 명시적으로 선언할 수 있는 **NamedParmeterJdbcTemplate**이 있다.
