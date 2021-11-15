---
layout: post
title:  "SwiftUI @State 알아보기"
date:   2021-11-10 20:11:11 +0900
categories: Swift
---

@State란 무엇일까요? 한마디로 정리하자면 struct안에서 값을 변경할 수 있게 해주는 Property Wrapper 입니다.
struct는 Value Type이기 때문에 기본적으로는 변경이 되지 않습니다.
기본적이라고 한 이유는 mutating이라는 키워드를 통해 새로운 객체를 만들어 다시 할당하는 방법으로 값을 변경하는 것처럼 보일 수 있고
Reference Type의 Property같은 경우엔 주소의 값이 변경되지 않으므로 값이 변경되지 않지만 그 안의 변수는 값의 수정이 자유롭기 떄문에
값이 수정 되는 것 처럼 보이기 때문입니다. @State도 후자와 같은 경우로 값의 변경을 지원하는 것 같습니다. 예시로 설명하겠습니다.

``` swift
import SwiftUI

struct ContentView: View {
    
    @State private var text: String = ""

    var body: some View {
        VStack(alignment: .leading) {
            Text("Name")
            TextField("input your name", text: $text)
        }
        .padding()
        .onChange(of: text) {
            print($0)
        }
    }
}
```
<center><img src="https://user-images.githubusercontent.com/46335714/141722293-e1ea2b19-3f49-4ac8-a74c-4847041abe75.png" width="300"></center>
TextField에서 User가 입력하는 값을 text라는 property에 저장하는 코드입니다. Hello World를 입력하면서 text의 값이 계속해서 바뀌는 것을 확인할 수 있습니다.
별건 아니지만 State가 어떻게 동작하는지 한번 알아볼까요 ?

``` swift
struct ContentView: View {
    
    @State private var text: String = "Hello"
    
    var body: some View {
        VStack {
            Text(text)
                .font(.largeTitle)
        }
    }
}
```
먼저 이 코드를 보시면 따로 동작 화면은 첨부하지 않겠습니다. 예상이 가실 거라 생각합니다. 화면 중앙에 Hello라는 큰 글씨가 있을 거라 예상이 갑니다.
그렇다면 여기서 init 에서 값을 변경해준다면 어떻게 될까요? 

``` swift
    init() {
        text = "World"
    }
```

정답은 바뀌지 않는다 입니다. text의 값을 바꿔주었는데 왜 안될까? 그 원인은 @State가 값을 저장하는 storage와 연결이 되지 않았기 떄문입니다.

``` swift
struct ContentView: View {
    
    @State private var text: String = "Hello"
    
    init() {
        print(text)
        text = "World"
        print(_text)
    }
    
    var body: some View {
        VStack {
            Text(text)
                .font(.largeTitle)
        }
    }
}
```
의 코드의 로그를 확인해보겠습니다.

<center><img src="https://user-images.githubusercontent.com/46335714/141723505-79a57fcc-01b0-4fb5-88d0-14886c10d00c.png" width="600"></center>

@State Property로 변수를 선언하면 컴파일러가 _text라는 변수를 또하나 생성합니다. 이 변수는 State<String> 타입의 변수입니다. 이 변수를 출력해보면 value 와 _location으로 구성되어 있는 것을 확인할 수 있습니다. 즉 SwiftUI가 변수를 저장하고 관리하는 location이 있는데 아직 이 location이 할당되지 않아 우리가 값을 수정해도 그 값이 적용되지 않는 것입니다.

``` swift
        .onAppear {
            print("onappear", _text)
        }
```
location은 onAppear에서 부터 접근이 가능하기 떄문에 이 life cycle에서 값을 수정하면 됩니다. onAppear에서는 location의 값이 잡히는것을 볼 수 있습니다.
<center><img src="https://user-images.githubusercontent.com/46335714/141723800-07151480-0613-46a2-98fc-76a91ac3dc21.png" width="600"></center>

그렇지만 init에서 값을 원하는 값으로 설정하는 방법도 있습니다.

``` swift
    init() {
        print(text)
        _text = .init(initialValue: "World")
        print(_text)
    }
```
 
컴파일러가 자동으로 생성한 _text변수에 다시 init해서 값을 설정하는 방법으로 원하는 값으로 변경할 수는 있습니다. 유용할지는 모르겠습니다. 
