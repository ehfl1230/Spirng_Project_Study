# 프레젠테이션(웹)계층의 CRUD 구현

## **Controller의 작성**

Controller는 하나의 클래스 내에서 여러 메서드를 작성하고, @RequestMapping 등을 이용해서 URL을 분기하는 구조로 작성할 수 있기 때문에 하나의 클래스에서 필요한 만큼 메서드의 분기를 이용하는 구조로 작성한다.

또한, 작성하기 전에 반드시 현재 원하는 기능을 호출하는 방식에 대해 다음과 같이 테이블로 정리한 후 코드를 작성하는 것이 좋다.(**명세서를 작성하는 것이 좋다.**)


여기에서는 BoardController 하나를 작성하여 URL 분기를 한다.

## **BoardController의 작성**

BoardController에서는 @Controller 어노테이션을 추가해서 스프링의 빈으로 인식할 수 있게 표시해두고, @RequestMapping을 통해서 '/board'로 시작하는 모든 처리를 BoardController가 하도록 지정한다.
이후에 root-context.xml에 표시된 Service가 Spring에서 인식할 수 있도록 base-package로 Service 패키지를 등록해준다. (Context항목이 없으면 Namespace에서 추가)


Controller 패키지는 보통 프로젝트 생성 시에 base-package로 지정해주기 때문에 되어있다면 안해줘도 된다.

```
@Controller
@Log4j
@RequestMapping("/board/*")
public class BoardController {

}
```

```
root-context.xml

<context:component-scan base-package="org.zerock.controller"></context:component-scan>
```

## **목록에 대한 처리와 테스트**

BoardController는 BoardService 타입의 객체와 같이 연동해야 하므로 의존성에 대한 처리도 같이 진행한다.

```
org.zerock.controller/BoardController

@Controller
@RequestMapping("/board/*")
@Log4j
@AllArgsConstructor
public class BoardController {
	
	// service 의존성 주입
	private BoardService service;
	
	@GetMapping("/list")
	public void list(Model model) {
		
		log.info("list");
		
		// 게시물의 목록을 전달해야 하므로 Model을 파라미터로 지정하고 결과를 담아 전달
		model.addAttribute("list", service.getList());
	}
}
```

@AllArgsConstructor로 BoardService에 대해서 의존성을 주입해준다(만일 생성자를 만들지 않을 경우는 @Setter(onMethod_ = {@Autowired})를 이용해서 처리)

**Controller Test 코드**

Controller의 Test코드는 이제까지와는 좀 다르게 진행된다. 그 이유는 웹을 개발할 때 매번 URL을 테스트하기위해 Tomcat과 같은 WAS(웹 어플리케이션 서버)를 실행하는 것이 불편하기 때문이다.

WAS를 실행하지 않기 위해서는 약간의 추가적인 코드가 필요하지만 서버 실행의 불필요와 오류 수정 단계를 줄여줄 수 있어 한 번쯤 고려해 볼 만한 방식이다.

```
src/test/java/org.zerock.controller/BoardControllerTests

@RunWith(SpringJUnit4ClassRunner.class)
// WebApplicationContext를 사용하기 위한 어노테이션
@WebAppConfiguration
@ContextConfiguration({
	"file:src/main/webapp/WEB-INF/spring/root-context.xml",
	"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml"
})
@Log4j
public class BoardControllerTests {
	
	// Servletcontext 사용
	@Setter(onMethod_ = {@Autowired})
	private WebApplicationContext ctx;
	
	// 가짜 mvc로 가짜로 URL과 파라미터 등을 브라우저에서 사용하는 것처럼 만들어서 Test가능
	private MockMvc mockMvc;
	
	// Test전에 매번 실행
	@Before
	public void setup() {
		this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
	}
	
	// get 방식으로 MockMvc 테스트
	@Test
	public void testList() throws Exception{
		log.info(
				mockMvc.perform(MockMvcRequestBuilders.get("/board/list"))
				.andReturn()
				.getModelAndView()
				.getModelMap()
				);
		
	}
}
```

테스트 클래스의 선언부에는 WebApplicationContext를 이용하기 위해 @WebAppConfiguration 어노테이션으로 Servlet의 ServletContext를 적용한다.

@Before 어노테이션이 적용된 setUp()에서는 import 할 때 JUnit을 이용해야 한다. @Before가 적용된 메서드는 모든 테스트 전에 매번 실행되는 메서드가 된다.

MockMvc는 말 그대로 '가짜 mvc'이다. 가짜로 URL과 파라미터 등을 브라우저에서 사용하는 것처럼 만들어서 Controller를 실행해 볼 수 있다.

테스트 코드인 testList()는 MockMvcRequestBuilders라는 존재를 이용해 GET방식의 호출을 한다. 이후에는 BoardController의 getList()에서 반환된 결과를 이용해서 Model에 어떤 데이터들이 담겨있는지 확인한다.

## **등록 처리와 테스트**

등록은 주로 Post 형식으로 처리한다.

```
org.zerock.controller/BoardController

@PostMapping("/register")
public String register(BoardVO board, RedirectAttributes rttr) {
    Log.info("register: " + board);
    
    service.register(board);
    
    // 등록된 게시물 번호를 전달
    rttr.addFlashAttribute("result", board.getBno());
    
    return "redirect:/board/list";
}
```

register 메서드는 등록 작업이 끝난 후 다시 목록 화면으로 이동하기 위해 다른 메서드와 다르게 String을 리턴 타입으로 지정하고, 추가적으로 새롭게 등록된 게시물의 번호를 같이 전달하기 위해 RedirectAttributes를 파라미터로 받는다. 리턴 시에는 `redirect:` 접두어를 사용해 스프링 MVC가 내부적으로 response.sendRedirect()를 처리하게 한다.

등록 처리의 Test 코드는 post 방식이므로 MockMvcRequestBuilder의 post를 이용하고 .param으로 전달해야 하는 파라미터들을 지정한다. (<input> 태그와 유사)

```
src/test/java/org.zerock.controller/BoardControllerTests

// post 방식의 register 테스트
@Test
public void testRegister() throws Exception{
    String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/register")
            .param("title", "테스트 새글 제목")
            .param("content", "테스트 새글 내용")
            .param("writer", "user00")
            ).andReturn().getModelAndView().getViewName();
    log.info(resultPage);
}
```

## **조회 처리와 테스트**

특별한 경우가 아니라면 주로 Get 형식으로 처리한다.

```
org.zerock.controller/BoardController

@GetMapping("/get")
public void get(@RequestParam("bno") Long bno, Model model) {
    log.info("/get");
    
    model.addAttribute("board", service.get(bno));
}
```

@RequestParam을 이용해서 좀 더 명시적으로 bno 값이 필요하다고 알리고, 화면 쪽에 해당 번호의 게시물을 전달해야 하므로 model을 파라미터로 받는다.


조회 처리의 Test 코드는 다음과 같다.

```
src/test/java/org.zerock.controller/BoardControllerTests

// Get 방식의 조회(Get) 테스트
@Test
public void testGet() throws Exception{
    log.info(
            mockMvc.perform(MockMvcRequestBuilders
                    .get("/board/get")
                    // bno파라미터를 추가해서 실행
                    .param("bno", "2"))
            .andReturn()
            .getModelAndView()
            .getModelMap()
            );
}
```

## **수정 처리와 테스트**

수정 작업은 등록과 유사하게 변경된 내용을 수집해서 처리하므로 BoardVO를 파라미터로 받는다. 또한 수정 작업을 시작하는 화면은 GET 방식으로 접근하지만 실제 작업은 POST 방식으로 동작하므로 @PostMapping을 이용해서 처리한다.

service.modify()는 수정 여부를 boolean으로 처리하고 그 결과를 RedirectAtrributes에 추가하기 위해 파라미터로 받는다.

```
org.zerock.controller/BoardController

@PostMapping("/modify")
public String modify(BoardVO board, RedirectAttributes rttr) {
    Log.info("modify:" + board);
    
    // 수정 성공시 success 반환
    if(service.modify(board)) {
        rttr.addFlashAttribute("result", "success");
    }
    return "redirect:/board/list";
}
```

수정 처리의 Test 코드는 다음과 같다.

```
src/test/java/org.zerock.controller/BoardControllerTests

// post 방식의 수정(modify) 테스트
@Test
public void testModify() throws Exception{
    String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/modify")
            .param("bno", "1")
            .param("title", "수정된 테스트 새글 제목")
            .param("content", "수정된 테스트 새글 내용")
            .param("writer", "user00")
            ).andReturn().getModelAndView().getViewName();
    
    log.info(resultPage);
}
```

## **삭제 처리와 테스트**

삭제 처리는 반드시 POST 방식으로만 처리한다.

```
org.zerock.controller/BoardController

@PostMapping("/delete")
public String remove(@RequestParam("bno") Long bno, RedirectAttributes rttr) {
    Log.info("remove..." + bno);
    
    if(service.remove(bno)) {
        rttr.addFlashAttribute("result", "success");
    }
    return "redirect:/board/list";
}
```

삭제 처리의 Test 코드는 다음과 같다.

```
src/test/java/org.zerock.controller/BoardControllerTests

// post 방식의 삭제(delete) 테스트
@Test
public void testDelete() throws Exception{
    String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/remove")
            .param("bno", "25")
            ).andReturn().getModelAndView().getViewName();
    
    log.info(resultPage);
}
```