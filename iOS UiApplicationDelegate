### applicationWillEnterForeground:와 appicationDidBecomeActive:의 차이

클래스업의 inputbar 구현 중 Swipe 뷰가 나오면 inputbar가
사라지고 들어가면 다시 보여야 했다. 마찬가지로 백그라운드 상태에서
다시 앱으로 들어왔을 때 항상 inputbar가 위에 있어야 했다.

그런데 전화가 와서 전화를 받고 다시 보았더니 inputbar가
사라져 있었다.

그래서 UIApplicationDelegate에 대한 상황별 쓰임을 자세히
찾게 되었다. 여기서는 active, foreground 상태를 고민했다.

### applicationDidEnterBackground:와 appicationWillResignActive:의 차이

클래스업에 App Search를 적용 중, 앱에 백그라운드 상태로 갈 때,
search에 관련된 데이터를 만들어 주는게 최상이라고 판단되어
정확하게 찾아보았다.

결국 클래스업에는 applicationDidEnterBackground:에
관련 코드를 작성해야 할 것 같다.

**[StackOverflow](https://stackoverflow.com/a/9860393/1496211)**에 정확하게 설명해 놓은 글이 있었다.
