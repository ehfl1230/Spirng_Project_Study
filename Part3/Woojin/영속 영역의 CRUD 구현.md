# 영속 영역(데이터 엑세스 계층)의 CRUD 구현

영속 영역은 테이블과 VO(DTO) 등 약간의 준비만으로도 비즈니스 로직과 무관하게 CRUD 작업을 자성할 수 있다.

MyBatis는 내부적으로 JDBC의 PreparedStatement를 활용하고 필요한 파라미터를 처리하는 '?'를 '#{속성}'을 이용해서 처리한다.

## **Create(insert) 처리**

tbl_board 테이블은 PK 칼럼으로 bno를 이용하고 시퀀스를 이용해서 자동으로 데이터가 추가될 때 번호가 만들어지는 방식을 사용한다.

이처럼 자동으로 PK값이 정해지는 경우에는 다음과 같은 2가지 방식으로 처리할 수 있다.

- insert문만 처리되고 생성된 PK값을 알 필요가 없는 경우
- insert문이 실행되고 생성된 PK값을 알아야 하는 경우

이와 같은 경우를 처리하기 위해 BoardMaper 인터페이스에  
- insert(생성된 PK값을 알 필요가 없는 경우)
- insertSelectKey(생성된 PK값을 알아야 하는 경우)  

메서드를 추가적으로 선언해준다.

```
org.zerock.mapper/BoardMapper

public interface BoardMapper {	
    // 추가되는 내용
    public void insert(BoardVO board);
	public void insertSelectKey(BoardVO board);

    // 어노테이션을 이용하면 아래와 같다.
    
    // @Insert("insert into tbl_board (bno,title,content,writer) values (seq_board.nextval, #{title}, #{content}, #{writer})")
	public void insert(BoardVO board);

    // bno값을 알야해서 select가 먼저 실행되어야 하는데 처리 방법을 모르겠음 
	public void insertSelectKey(BoardVO board);
}
```

BoardMapper.xml에는 다음과 같은 내용이 추가되어야 한다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.mapper.BoardMapper">

<insert id="insert">
insert into tbl_board (bno,title,content,writer) values (seq_board.nextval, #{title}, #{content}, #{writer})
</insert>

<insert id="insertSelectKey">
    <!-- bno 값을 구하는 코드 -->
	<selectKey keyProperty="bno" order="BEFORE" resultType="long">
	select seq_board.nextval from dual
	</selectKey>

insert into tbl_board (bno,title,content,writer) values (#{bno}, #{title}, #{content}, #{writer}) 
</insert>

</mapper>
```

insert()는 단순히 다음 시퀀스 값을 구해서 insert할 때 사용하므로 1번의 SQL 처리만으로 작업이 완료된다.

insertSelectKey()는 @SelectKey라는 MyBatis의 어노테이션을 이용해 PK값을 미리(before) SQL을 통해서 처리해 두고 특정한 이름으로 결과를 보관하는 방식을 이용해 이 처리된 결과로 insert문을 실행하는 방식을 이용한다.

아래는 위의 Mapper를 Test할 수 있는 Test 클래스 추가 코드이다.

```
src/test/java/org.zerock.mapper/BoardMapperTests

// insert문 Test
// bno가 null로 입력
@Test
public void testInsert() {
	BoardVO board = new BoardVO();
	board.setTitle("새로 작성하는 글");
	board.setContent("새로 작성하는 내용");
	board.setWriter("newbie");

	mapper.insert(board);
	log.info(board);
}

// insertSelectKey문 Test
@Test
public void testInsertSelectKey() {
	BoardVO board = new BoardVO();
	board.setTitle("새로 작성하는 글 select key");
	board.setContent("새로 작성하는 내용 select key");
	board.setWriter("newbie");

	mapper.insertSelectKey(board);
	log.info(board);
}
```
---
## **Read(Select) 처리**

Read 처리는 데이터를 조회하는 작업(Select)으로 PK를 이용해서 처리하므로 BoardMapper의 파라미터 역시 BoardVO 클래스의 bno 타입 정보를 이용해서 처리한다.

따라서, 아래와 같이 BoardMapper 인터페이스의 메서드가 bno를 파라미터로 가진다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
    public BoardVO read(Long bno);
}
```

MyBatis는 Mapper 인터페이스의 리턴 타입에 맞게 select의 결과를 처리하기 때문에 아래의 select문의 결과는 BoardVO에 각 속성에 맞는 값으로 처리된다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<select id="read" resultType="org.zerock.domain.BoardVO">
    select * from tbl_board where bno = #{bno}
</select>
```

위 Read 인터페이스를 테스트 할 코드는 아래와 같다

```
src/test/java/org.zerock.mapper/BoardMapperTests
//Read 처리 Test
@Test
public void testRead() {
    // read함수로 bno값이 5인 board를 찾아서 출력
    BoardVO board = mapper.read(5L);
    log.info(board);
}
```

## **Delete 처리**

Delete 처리 역시 특정한 데이터를 삭제하는 것으로 PK를 이용해서 처리한다. 등록, 삭제, 수정과 같은 DML 작업은 '몇 건의 데이터가 삭제(혹은 수정) 되었는지를' 반환할 수 있다.

read와 같이 bno를 파라미터로 가진다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
	public int delete(long bno);
}
```

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<delete id="delete">
	delete from tbl_board where bno = #{bno}
</delete>
```

위 Delete 인터페이스를 테스트 할 코드는 아래와 같다

```
src/test/java/org.zerock.mapper/BoardMapperTests
// Delete 처리 Test
@Test
public void testDelete() {
	// delete함수로 bno값이 3인 board를 찾아서 삭제 후 삭제 count 출력
	log.info("DELETE COUNT: " + mapper.delete(3L));
}
```

## **Update 처리**

Update는 게시물등의 제목, 내용, 작성자를 수정하는 것으로 업데이트할 때에 최종 수정시간을 데이터베이스 내 현재 시간으로 수정해줘야 한다. Update 처리는 delete와 마찬가지로 '몇 개의 데이터가 수정되었는가'를 처리할 수 있게 int타입으로 메서드를 설계할 수 있다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
	public int update(BoardVO board);
}
```

수행할 SQL문은 updateDate가 현재 시간으로 수정되고, regdate 칼럼은 최초 생성 시간이므로 건드리지 않는 것이 중요하다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<update id="update">
	update tbl_board
	set title = #{title},
	content = #{content},
	writer = #{writer},
	updateDate = sysdate
	where bno = #{bno}
</update>
```

위 Update 인터페이스를 테스트 할 코드는 아래와 같다.  
테스트 코드는 read()를 이용해서 가져온 BoardVO 객체의 일부를 수정하거나, 새로운 객체를 생성해서 처리할 수 있다. 예제는 객체를 생성해서 테스트를 진행한다.

```
src/test/java/org.zerock.mapper/BoardMapperTests
// Update 처리 Test
@Test
public void testUpdate() {
	BoardVO board = new BoardVO();
	// 실행전 수정할 번호가 존재하는 지 확인해줄 것
	board.setBno(5L);
	board.setTitle("수정된 제목");
	board.setContent("수정된 내용");
	board.setWriter("user00");
	
	int count = mapper.update(board);
	log.info("UPDATE COUNT: " + count);
}
```