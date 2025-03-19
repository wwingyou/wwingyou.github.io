---
layout: post
tags: Spring
title: NamedParameterJdbcTemplate
---

파라미터 바인딩의 순서를 꼭 맞춰줘야 하는 [[JdbcTemplate]]과는 다르게 파라미터의 이름을 명시하여 바인딩할 수 있는 JdbcTemplate이다. 

# 설정

`JdbpTemplate`과 마찬가지로 `spring-jdbc`라이브러리에 기본으로 포함되어 있으므로 스프링을 이용해 JDBC를 사용한다면 별도의 설정 없이 바로 사용할 수 있다.

# 사용

파라미터 바인딩에 `?`를 사용하는 `JdbcTemplate`과는 다르게 `:paramName` 형태의 명시적 바인딩을 사용한다. 

### 준비

생성자에 `DataSource` 구현체를 넘겨주어 `NamedParameterJdbcTemplate`을 생성한다.

```java
NamedParameterJdbcTemplate template = new NamedParameterJdbcTemplate(dataSource);
```

### Select

바인딩할 파라미터를 순서에 맞게 넘겨주면 됐던 `JdbcTemplate` API와는 다르게 파라미터를 `Map<String, Object>` 형태로 넘겨주어야 한다. 

```java
String sql = "select id, item_name, price from item where id = :id"
RowMapper<Item> rowMapper = BeanPropetyRowMapper.newInstance(Item.class);
Map<String, Object> paramMap = Map.of("id", id);

try {
	Item item = template.queryForObject(sql, paramMap, rowMapper);
	// do something with item
} catch (EmptyResultDataAccessException e) {
	// catch when there is no data
}
```

또다른 차이점은 `RowMapper<T>` 대신에 `BeanPropertyRowMapper`를 사용했다는 것이다. `BeanPropertyRowMapper`는 자바빈 클래스에 맞게 자동으로 결과를 바인딩해주는 `RowMapper<T>`이다. 

> [!info] 관례의 불일치
> 자바빈 규칙에서 프로퍼티는 카멜케이스로 표기하지만, 데이터베이스 칼럼 이름은 보통 스네이크케이스로 표기된다. `BeanPropertyRowMapper`는 이런 관례의 불일치를 자동으로 해결해준다.(예: item_name -> setItemName() 호출)
> 만약 칼럼 이름이 자바빈 프로퍼티와 완전히 이름이 다르다면 SQL에서 AS 키워드로 칼럼 이름을 알맞게 변환시켜주면 된다.

### Insert

insert는 저장할 빈 객체의 속성들을 모두 파라미터로 바인딩해줘야 하는 경우가 많다. select처럼 파라미터 맵을 만들어서 바인딩을 할 수도 있지만, `BeanPropertySqlParameterSource`를 사용하면 [[JavaBean]]에서 파라미터를 가져오는 코드를 훨씬 간소화시킬 수 있다.

```java
String sql = "insert into item(item_name, price) " + 
			"values (:itemName, :price)";
SqlParameterSource param = new BeanPropertySqlParameterSouce(item);
KeyHolder keyHolder = new GeneratedKeyHolde();
template.update(sql, param, keyHolder);

Long key = keyHolder.getKey().longValue();
// do something with key
```

이 경우 자동생성 키를 받아오는 훨씬 간편한 API를 사용할 수 있으므로 일반 `JdbcTemplate`을 사용하는 것 보다 좀더 간편하고 좋은 방식이다.

하지만 이것보다 더 간편한 방식도 있다. [[SimpleJdbcInsert]]를 이용하면 sql을 작성할 필요 없이 빈 오브젝트의 값을 인서트하고 자동생성 키 값도 받아올 수 있다.

### Update

insert에서 사용한 `BeanPropertySqlParameterSource`는 자바 빈에서 파라미터를 자동으로 가져오는 `SqlParameterSource`의 구현체이다. 하지만 값을 업데이트하는 경우에는 이 구현체를 사용하기가 조금 어려운데, 보통 업데이트는 `WHERE`문을 포함하여 조건 파라미터를 함께 넘겨줘야 하는 경우가 많기 때문이다. 

이런 경우에는 `MapSqlParameterSource`를 사용하여 직접 파라미터값을 지정해주면 된다.

```java
String sql = "update item " + 
			"set item_name=:itemName, price=:price" + 
			"where id=:id";
SqlParameteSource param = new MapSqlParameterSource()
							.addValue("itemName", updateParam.getItemName())
							.addValue("price", updateParam.getPrice())
							.addValue("id", id);
template.update(sql, param);
```

