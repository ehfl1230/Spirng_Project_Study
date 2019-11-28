# Part 3 각 영역의 Naming 명명 규칙

- xxxController   
    : 스프링 MVC에서 동작하는 Controller 클래스를 설계할 때 사용
- xxxService, Servicelmpl  
    : 비지니스 영역의 인터페이스는 Service, 인터페이스를 구현한 클래스는 Servicelmpl 사용
- xxxDao, xxxRepository  
    : 데이터 엑세스 영역을 DAO(Data-Access-Object)나 Repository라는 이름으로 명명, 이 책에서는 별도의 DAO를 구성하는 대신 MyBatis의 Mapper 인터페이스 활용
- VO,DTO  
    : VO와 DTO는 일반적으로 유사한 의미로 데이터를 담고 있는 객체를 의미한다는 공통점이 있다. 다만, VO는 주로 Read Only의 목적이 강하고, 데이터 자체도 Immutable(불변)하게 설계하는 것이 정석이다. DTO는 주로 데이터 수집의 용도가 더 강하다.

    ex) 웹 화면에서 로그인하는 정보를 DTO로 처리