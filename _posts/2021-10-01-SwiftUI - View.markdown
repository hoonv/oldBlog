---
layout: post
title:  "SwiftUI - View"
date:   2021-08-10 10:45:14 +0900
categories: SwiftUI
---
SwiftUI를 시작하는데 있어서 새로 알아야 할 것이 많죠~ 가장 기본이 되는 View부터 살펴보려 합니다. Apple의 정의부터 한번 보고 가실까요? 

### View
A type that represents part of your app’s user interface and provides modifiers that you use to configure views.

앱의 UI의 부분을 나타내고, 뷰를 구성하는데 사용하는 변경자를 제공하는 타입이라고 되어 있습니다. 한마디로 앱의 UI입니다. 이전에 UIKit에서 사용하던 UIView와 같은 역할을 하고 있는 녀석 같습니다.
``` swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
public protocol View {

    associatedtype Body : View

    @ViewBuilder var body: Self.Body { get }

}

struct MyView: View {
    var body: some View {
        Text("Hello, World!")
    }
}
```
정의와 사용법을 한번 보고 가실까요? View의 정의는 associatedtype인 Body 즉 View 타입입니다. 그리고 body라는 property로 되어 있는 간단한? 정의를 갖고 있습니다. 사용예시인 MyView도 보면 body라는 computed property 하나만 선언 되어 있는 것을 볼 수 있습니다. 여기서 새로운 개념이라고 한다면 @ViewBuilder와 some이 있을 것 같습니다. 

### ViewBuilder
A custom parameter attribute that constructs views from closures.

클로져로 뷰를 생성하는 attribute라고 합니다. 즉 클로져 안에 뷰가 있으면 그것들을 하나의 뷰로 또 만들어주는 녀석입니다. ViewBuilder는 대부분 자식 뷰를 만들때 함수의 파라미터에 사용한다고 합니다. 
``` swift
extension View {
    public func bottomCard<Content: View>(
        @ViewBuilder content: @escaping () -> Content) -> some View {
        ZStack {
            self
            BottomCardView()
        }
    }
}

```
이런 식으로 말이죠 ~ ViewBuilder는 여러개의 함수가 될 수 있는데 이 함수는 0~10개의 View를 받을 수 있습니다.
``` swift
// build 파라미터 최대 10개
static func buildBlock<C0, C1, C2, C3, C4, C5, C6, C7, C8, C9>
(C0, C1, C2, C3, C4, C5, C6, C7, C8, C9) 
-> TupleView<(C0, C1, C2, C3, C4, C5, C6, C7, C8, C9)>
```
정의를 보면 파라미터로 0~10개의 View만 받고 있기 떄문에 하나의 클로져 안에 10개 이상의 View를 넣지 못하는 것은 이 이유 떄문으로 보입니다.
한가지 더 살펴볼 것은 buildEither인데 이녀석 덕분에 클로져 안에서 if 문을 사용 할 수 있는 것입니다. 
``` swift
    var body: some View {
        Text("xx")
        if true {
            Text("ssss")
        }
    }
```
