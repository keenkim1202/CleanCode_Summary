# Clean Code 3장
: 함수

프로그래밍 초창기에는 시스템을 루틴과 하위 루틴으로 나눴고, 포트란과 PL/1 시절에는 시스템을 프로그램, 하위 프로그램, 함수로 나눴다.

지금은 `함수`만 살아남았다.
어떤 프로그램이든 가장 기본적인 단위가 `함수`이다.
이 장은 `함수를 잘 만드는 법`을 소개한다.

</br>

아래의 코드를 3분을 줄테니 살펴보자.  
[FitNesse](www.fitnesse.org)에서 긴 함수를 찾기는 어렵다.  
길이가 길 뿐만 아니라 중복 된 코드에, 괴상한 문자열에, 낯설고 모호한 자료 우형과 API가 많다.

</br>

(swift에는 StringBuffer 타입이 존재하지 않아 StringBuilder.swfit 파일을 참고링크로 남깁니다.)
- [StringBuilder.swift](https://gist.github.com/kristopherjohnson/1fc55e811d944a430289)
- [java에서 StringBuffer 에 대하여](https://wikidocs.net/276)

```swift
// HtmlUtl.java (FitNesse 20070619) 를 Swift로 의역
// * FitNesse : 오픈 소스 테스트 도구이다.

func testableHtml(pageData: PageData, includeSuiteSetup: Bool) throws -> String {
    var wikiPage: WikiPage = pageData.getWikiPage()
    var buffer: StringBuffer = StringBuffer()

    if pageData.hasAttribute("Test") {
        if includeSuiteSetup {
            let suiteSetup: WikiPage = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage)

            if suiteSetup != nil {
                let pagePath: WikiPagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup)
                let pagePathName: String = PathParser.render(pagePath)

                buffer.append("!include -setup .")
                buffer.append(pagePathName)
                buffer.append("\n")
            }
        }

        let setup: WikiPage = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage)

        if setup != nil {
            let setupPath: WikiPagePath = wikiPage.getPageCrawler().getFullPath(setup)
            let setupPathName: String = PathParser.render(setupPath)
            buffer.append("!include -setup .")
            buffer.append(setupPathName)
            buffer.append("\n")
        }
    }

    buffer.append(pageData.getContent())

    if pageData.hasAttribute("Test") {
        let teardown: WikiPage = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage)

        if teardown != nil {
            let tearDownPath: WikiPagePath = wikiPage.getPageCrawler().getFullPath(teardown)
            let tearDownPathName: String = PathParser.render(tearDownPath)
            buffer.append("\n")
            buffer.append("!include -teardown .")
            buffer.append(tearDownPathName)
            buffer.append("\n")
        }

        if includeSuiteSetup {
            let suiteTeardown: WikiPage = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage)

            if suiteTeardown != nil {
                let pagePath: WikiPagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown)
                let pagePathName: String = PathParser.render(pagePath)
                buffer.append("!include -teardown .")
                buffer.append(pagePathName)
                buffer.append("\n")
            }
        }
    }

    pageData.setContent(buffer.toString())
    return pageData.getHtml()
}
```

3분이 지났다. 코드를 이해하겠는가?  
추상화 수준도 너무 다양하고, 코드도 너무 길다.  
두 겹으로 중첩된 if문은 이상한 플래그를 확인하고, 이상한 문자열을 사용하며, 이상한 함수를 호출한다.

</br>

메서드 몇 개를 추출하고, 이름 몇 개를 변경하고 ,구조를 조금 변경했더니 아래와 같은 코드가 되었다.  
함수 의도를 코드 9줄로 표현했다. 자, 다시 3분을 줄 테니 코드를 살펴보자.
```swift
func renderPageWithSetupAndTeardowns(pageData: PageData, isSuite: Bool) throws -> String {
    let isTestPage: WikiPage = PageData.hasAttributes("Test")

    if isTestPage {
        let testPage: WikiPage = pagData.getWikiPage()
        let newPageContent: StringBuffer = StringBuffer()

        includeSetupPages(testPage, newPageContent, isSuite)
        newPageContent.append(pageData.getContent())

        includeTeardownPages(testPage, newPageContent, isSuite)
        pageData.setContent(newPageContent.toString())
    }
    
    return pageData.getHtml()
}
```
FitNesse에 익숙하지 않은 이상 코드를 100% 이해하기는 어려우리라.  
그래도   
- 함수가 설정페이지(setup)와 해제페이지(teardown)를 테스트 페이지에 넣은 후 해당 테스트 페이지를 HTML로 렌더링한다는 사실은 짐작하리라.
- JUnit에 익숙하다면 이 함수가 일종의 웹 기반 테스트 프레임워크에 속한다는 사실도 짐작하리라.
여러분 짐작이 옳다.  
첫번째 코드에서는 파악하기 어려웠던 정보가 두번쨰 코드에서는 쉽게 드러난다.

</br>

그렇다면 두번째 코드 속 함수가 
- 읽기 쉽고 이해하기 쉬운 이유는 무엇일까?
- 의도를 분명히 표현하는 함수를 어떻게 구현할 수 있을까?
- 함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

</br>

----

</br>

## 작게 만들어라
> 블록과 들여쓰기

</br>

----

## 한 가지만 해라!
> 함수 내 섹션

</br>

----

## 함수 당 추상화 수준은 하나로!
> 위에서 아래로 코드 읽기: 내려가기 규칙

</br>

----

## Switch 문

</br>

----
## 서술적인 이름을 사용하라!

</br>

----
## 함수 인수
> 많이 쓰는 단항 형식 

> 플래그 인수

> 이항 함수

> 삼항 함수

> 인수 객체

> 인수 목록

> 동사와 키워드

</br>

----
## 부수 효과를 일으키지 마라!
> 출력 인수

</br>

----
## 명령과 조회를 분리하라!

</br>

----
## 오류 코드보다 예외를 사용하라!
> try / catch 블록 뽑아내기

> 오류 처리도 한 가지 작업이다

> Error.swift 의존성 자석

</br>

----
## 반복하지 마라!

</br>

----

## 구조적 프로그래밍

</br>

----
## 함수를 어떻게 짜죠?

</br>

----
## 결론
