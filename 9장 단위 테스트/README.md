# 9장 단위 테스트
> 애자일과 TDD덕분에 단위 테스트를 자동화하는 프로그래머들이 이미 많아졌으며 점점 더 늘어나는 추세다
> 테스트를 추가하려고 급하게 서두르는 와중에 제대로 된 테스트 케이스를 작성해야 한다는 좀 더 미묘한(그리고 더 중요한) 사실을 놓쳐버렸다

#### TDD 법칙 세가지
* **첫째 법칙** : 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다
* **둘째 법칙** : 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다
* **셋째 법칙** : 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다
* 위 3가지 법칙을 따르면 개발과 테스트를 빠르게 수행할수 있지만 테스트 케이스가 너무 많이 나와서 관리가 힘들 수 있다

<br>

#### 깨끗한 테스트 코드 유지하기
* 지저분한 테스트 코드는 실제 코드를 짜는 시간보다 테스트 케이스를 추가하는 시간이 더 걸린다
* 테스트 코드는 실제 코드 못지 않게 중요하다
* 테스트는 **유연성, 유지보수성, 재사용성**을 제공한다
  * 코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 **단위 테스트**다
  * 실제코드를 점검하는 자동화된 단위 테스트 슈트는 설계와 아키텍처를 최대한 깨끗하게 보존하는 열쇠다
  * 테스트 케이스가 있으면 변경이 쉬워지기 때문이다
  * 테스트 코드가 지저분할수록 실제 코드도 지저분해진다. 결국 테스트 코드를 잃어버리고 실제 코드도 망가진다

<br>

#### 깨끗한 테스트 코드
* 깨끗한 테스트 코드를 만들려면 **가독성**이 중요하다
  * 가독성을 높이려면 **명료성, 단순성, 풍부한 표현력**이 필요하다
* 테스트 코드는 최소의 표현으로 많은 것을 나타내야 한다
```
// FitNess에서 가져온 코드
public void testGetPageHieratchyAsXml() throws Exception {
    crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));
    
    request.setResource("root");
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response = (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();
    
    assertEquals("text/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
}
public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
  WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  PageData data = pageOne.getData();
  WikiPageProperties properties = data.getProperties();
  WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
  symLinks.set("SymPage", "PageTwo");
  pageOne.commit(data);

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
  assertNotSubString("SymPage", xml);
}
public void testGetDataAsHtml() throws Exception {
  crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

  request.setResource("TestPageOne"); request.addInput("type", "data");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("test page", xml);
  assertSubString("<Test", xml);
}
```
* addPage와 assertSubString을 부르느라 중복되는 코드가 매우 많다
* 위 코드는 읽는 사람을 고려하지 않는다. 불쌍한 독자들은 온갖 잡다하고 무관한 코드를 이해한 후에야 테스트 케이스를 이해한다
```
// 위의 코드를 개선한 코드
public void testGetPageHierarchyAsXml() throws Exception {
    makePages("PageOne", "PageOne.ChildOne", "PageTwo");
    
    submitRequest("root", "type:pages");
    
    assertResponseIsXml();
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
    WikiPage page = makePage("PageOne");
    makePages("PageOne.ChildOne", "PageTwo");
    
    addLinkTo(page, "pageTwo", "SymPage");
    
    submitRequest("root", "type:pages");
    
    assertResponseIsXml();
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
    assertResponseDoesNotContain("SymPage");
}
public void testGetDataAsXml() throws Exception {
    makePageWithContent("TestPageOne", "test page");
    
    submitRequest("TestPageOne", "type:data");
    
    assertResponseIsXml();
    assertResponseContains("test page", "<Test");
}
```
* BUILD-OPERATE-CHECK 패턴이 위와 같은 구조에 적합하다
* 각 테스트는
  * 테스트 자료를 만든다
  * 테스트 자료를 조작한다
  * 조작한 결과가 올바른지 확인한다
* 테스트 코드는 진짜 필요한 자료 유형과 함수만 사용한다
* 그러므로 코드를 읽는 사람은 온갖 잡다하고 세세한 코드에 주눅들고 헷갈릴 필요 없이 코드가 수행하는 기능을 재빨리 이해한다
* 도메인에 특화된 테스트 언어
  * 위의 코드들을 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다
  * 즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 **언어**다
  * **숙련된 개발자라면 자기 코드를 좀 더 간결하고 표현력이 풍부한 코드로 리팩터링해야 마땅하다**
* 이중 표준
  * 테스트 API코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다
  * 컴퓨터 자원과 메모리가 제한적일 가능성이 있지만 테스트 환경은 자원이 제한적일 가능성이 낮다(이중 표준의 본질)
  * 실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식이 있다(효율성보단 가독성에 투자하자)

<br>

#### 테스트당 assert 하나
* assert문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다
* 테스트를 분리하면 중복되는 코드가 많아지지만 TEMPLATE METHOD패턴을 사용하면 중복을 제거할 수 있다
* 가장 좋은 개념은
  * **개념 당 assert 문 수를 최소로 줄여라**
  * **테스트 함수 하나는 개념 하나만 테스트하라**

<br>

#### F.I.R.S.T
* 깨끗한 코드가 따르는 다섯 가지 규칙
* **빠르게(Fast)** : 테스트는 빨리 돌아야 자주 돌려 문제를 찾을 수 있다
* **독립적으로(Independent)** : 테스트가 서로 의존해서는 안된다, 이는 하나가 실패하면 나머지도 실패하므로 원인을 진단하기 힘들며 결함이 숨겨진다
* **반복가능하게(Repeatable)** : 테스트는 어떤 환경에서도 반복 가능해야 한다, 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다
* **자가검증하는(Self-Validating)** : 테스트는 부울(bool)값으로 내야한다, 성공 or 실패로 나와야 한다
* **적시에(Timely)** : 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다

<br>

#### 결론
* 테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문에 실제 코드보다 더 중요할 수 있다
* 테스트 API를 구현해 도메인 특화 언어(DSL)을 만들자, 그러면 그만큼 테스트 코드를 짜기가 쉬워진다
* 테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자