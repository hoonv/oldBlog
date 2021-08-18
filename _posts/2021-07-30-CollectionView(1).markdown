---
layout: post
title:  "CollectionView 관철하기"
date:   2021-07-30 08:45:14 +0900
categories: Swift
---
애플 공식문서를 한번 확인해보겠습니다. 사실 CollectionView를 많이 사용해왔지만 재대로 공식문서를 읽어본적이 없는 것 같아 스스로도 조금 충격을 받았습니다.. ㅎㅎ 여러분들도 안읽어 보셨다면 읽어보시는것을 추천드립니당..!
아참 관철하다라는 의미는 어려움을 뚫고 나아가 목적을 기어이 이루다 라는 뜻과 사물을 속속들이 꽤뚫어보다 라는 의미를 갖고 있어요 😃 개발자로 살아가는데 꼭 필요한 덕목이라는 생각이 들어서 저의 좌우명으로 삼았답니당😅
📘Apple Document

CollectionView는 여러 객체들이 협력해서 화면을 그려냅니다. 대표적으로 DataSource Delegate Layout 이 세가지의 객체들과 협력하는데요, 각각 담당하는 일이 명확히 구별 되어 있습니다. 사실 iOS를 공부하면서 아 왜 이렇게 어렵게 만들어 냈어 이렇게 불평한 적이 많습니다. 근데 천천히 생각해보면 이렇게 만드는게 좋구나 라고 느낄때가 많습니다. 객체의 역할을 잘 구별해서 협력하고 확정성있게 만드는 Apple의 노력.. 또 세계최고의 개발자들이 모여서 만든것이니 오히려 더 배워야할게 많은 것 같습니다. 그래서 CollectionView가 화면을 그리는데 필요한 각각의 객체를 이해하고 어떻게 협력 하는지 한번 차근 차근 하나하나 print를 하면서 알아 봅시당.. ㅎㅎ😭
재사용 Reuse

먼저 CollectionView의 특성을 한번 알아볼께요! 기본적으로 CollectionView는 ScrollView를 상속 받고있습니다. CollectionView의 역할이 많은 데이터들을 표현해야 하기때문에 당연히 Scroll이 필요합니다. CollectionView는 Cell을 통해 데이터를 표현하잖아요?! 그렇다면 데이터가 10000개가 있을때 Cell을 10000개 생성해야할까요? 아래 그림에서 처럼 데이터가 10000개가 있다 해도 화면에 표시하는 Cell은 몇개 되지 않습니다. 그래서 Scroll을 함에 따라 이전에 사용했던 Cell을 재사용 하는 것입니다. dequeueReusableCell이라는 메소드 많이 사용하셨죠? 재사용 셀은 dequeue에서 가져오는 메소드랍니다.
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129881900-d56af982-b959-4dad-8ced-1279397a4b92.png"></p>
왜 재사용을 해야 하는지는 조금 와닿으시나요~?🧐 CollectionView는 Cell을 재사용하기 때문에 우리가 해줘야 하는 일이 하나 있습니다. 그것은 Cell을 초기 상태로 만들어 줘야 하는것입니다. 밑에 사진은 제가 사진을 선택하면 분홍색으로 테두리를 주는 모습인데요. 셀을 사용할때 초기화를 하지 않으면 이렇게 테두리가 여러개 그려진 모습을 볼 수 있습니다..
![image](https://user-images.githubusercontent.com/46335714/129881925-827f130e-4e9e-4321-892b-3ef5c77f7073.png){: width="80%"){: .center}

(로직에는 문제가없어요 ㅠ) 따라서 재사용 하기전에 셀을 초기화 하는 과정이 필요합니다. 따라서 그부분은 Cell내부의 prepareForReuse method를 override 해서 초기화하는 작업을 진행 해주면 됩니다.!
``` swift
// class PhotoCell: UICollectionViewCel
override func prepareForReuse() {
    super.prepareForReuse()
    self.layer.borderWidth = 0
    self.layer.borderColor = nil
}
```
prefetch

두번째 CollectionView의 특징입니다. prefetch에 관한 부분입니디. 미리 가져온다 라는 뜻입니다. CollectionView가 Cell을 데이터에 맞게 다 만들지 않고 화면에 보이는 것만 만들기 때문에 prefetch가 필요합니다. 이것을 하지 않는다면 스크롤 하면서 cell을 준비하게 될텐데 무거운 로직이 들어있다면 cell을 만드는데 오래 걸리기 때문에 스크롤이 버벅거리는 현상이 발생합니다. 이를 막기 위해 앞으로 필요할 Cell과 Data를 미리 준비하는 과정이 바로 prefetch입니다. 잠깐 위의 사진 보고 오시면 하늘색 Cell이 화면에 보이는 Cell이라면 연보라색 Cell은 prefetch된 Cell이라고 할 수 있습니다.
CollectionView는 두가지 prefetch기술을 제공합니다. 첫번째로는 Cell을 Prefetch하는 것입니다. Cell이 화면에 표시되야할때 만드는 것이 아니라 미리 보이지않는 곳이지만 곧 보일것 같은 Cell을 미리 준비하고 있는것입니다. 이것은 Default로 기능이 제공됩니다. 저희가 딱히 신경쓸것이 없습니다. 두번째로는 Data preFetch입니다. Cell을 표현하기위한 Data를 미리 가져오는 것입니다. 원격서버에 저장 되어있는 Image를 가져오는 경우가 시간이 많이 걸리는데 이것을 Cell이 Display되고 나서 가져오기에는 너무 무거운 작업입니다. 이렇게 무거운 작업들을 미리 가져올 수 있도록 prefetch기능을 제공합니다. 이 기능은 default로 제공되지 않습니다. `collectionView.prefetchDataSource`의 객체를 할당 해줘야합니다.
prefetch는 이번 포스팅에서 다루기엔 내용이 많아서 다음 포스팅에서 한번 다뤄 보겠습니다..!
DataSource

CollectionView가 협력하는 객체 DataSource입니다. 여러번 사용해보신분들은 익숙하시죠? collectionView.dataSource = self라는 코드로 자기자신을 DatsSource로 지정할 수 있습니다. CollectionView는 DataSource에게 Data가 몇개인지. 몇번째 데이터는 무엇인지 요구합니다. DataSource는 CollectionView를 구현하는데 필수적인 객체 입니다. 반면에 Delegate는 필수로 구현하지 않아도 문제는 없습니다.
``` swift 
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    ...
}

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    ...
}
```
이 두가지의 메소드는 필수입니다. default는 section은 한개 있고, 그 section의 item은 몇개인지 반환하는 함수와 Cell을 반환하는 함수만 구현하면 됩니다.
Layout

CollectionView에서 꼭 필요한 객체인 layout입니다. layout을 설정을 안해주면 런타임에러로 앱이 죽습니다.. 자 한마디로 Cell을 어떻게 배치할 것인지 결정하는 객체입니다. 저희가 자주 사용하는 layout은 Flow Layout입니다. Grid나 Row형태로 Cell을 구현할때 딱 알맞은 layout입니다. 딱히 더 설명 할 게 없네요.. 한마디로 Cell을 배치하는 객체인데 왜 Cell이 여기 그려지지?? 하는 것들은 이 Layout이 그렇게 다 배치한 것입니다. ꉂꉂ(ᵔᗜᵔ*)
⚙️Present Process

자 제가 사실 포스팅하고 싶었던 부분입니다. CollectionView가 Cell을 그릴때 DataSource와 Layout Delegate와 협력하여 화면을 그리는 데 어떤 식으로 협력하고 있는지 한땀한땀 print를 통해 알아보겠습니다..
![image](https://user-images.githubusercontent.com/46335714/129881932-d1188b5b-9f69-4efe-8c54-6601980ba047.png){: width="80%"){: .center}

먼저 Apple 문서에 있는 그림입니다. 어떻게 CollectionView가 Cell을 그리는지 간단히 나타낸 사진입니다. DataSource에서 Cell을 생성하고 layout에서 Attributes를 생성하면 CollectionView가 이를 배치하는 구조입니다. 어떠한 순서로 메소드들이 불리는 알아봅시당.! 핵심적인 메소드들을 print로 log를 찍어 봤습니다.

먼저 모든 View는 ViewController의 viewWillAppear viewDidAppear 사이에서 이루어 집니다. viewWillAppear가 불리고 나면 먼저 Layout의 Prepare 메소드가 불립니다. 이 메소드 안에서는 Layout을 위한 Attributes를 생성하는 일을 하거나 계산하는 일을 진행합니다. 지금은 FlowLayout을 상속받았기 때문에 특별히 추가적인 동작은 하지 않은 모습입니다만, CustomLayout을 작성할떄는 이부분에서 속성들을 미리 준비 해두고 여러 계산들을 진행합니다.

그 후 DataSource의 numberOfItemsInSection 메소드가 불립니다. 아이템이 몇개인지 확인 하기 위함인데 이는 Layout의 ContentSize를 결정하기 위함입니다. Layout: collectionViewContentSize 이 변수를 결정 하기 위함입니다. 아이템의 갯수가 몇개인지 확인 한 후 이 CollectionView의 Size가 어떻게 될지 계산하는 것입니다. 아이템의 개수만 알면 ContentSize를 알 수 있는것은 아니나. 현재 제가 CollectionView를 생성할때 itemSize를 결정해주어서 이게 가능한것입니다. 아래 사진을 보시면 Layout을 생성한 후 itemSize를 통해 Cell의 size를 미리 고정해둔 모습입니다.
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129881952-d7545273-7bbc-4727-b0a4-d6cdc8259d33.png" width="75%"></p>
ItemSize를 결정하는 방법으로는 FlowLayoutDelegate를 통해 CellSize를 결정 할 수 있습니다만, 이 방법은 Cell마다 크기가 다를때 이 방법으로 CellSize를 결정 해주시면 됩니다.
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129881958-29f5f4bc-848a-4a26-8069-116890dcc109.png" width="75%"></p>
만약 CellSize가 동일하다면 이 방법은 약간의 비효율을 초래합니다. ContentSize를 구하기 위해서 Cell 마다 이 함수를 호출하기 때문입니다.
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129881966-b263acb1-a2db-4f30-8fd6-f36d73721c66.png" width="75%"></p>
그 후 layout에서 layoutAttributesForElements 메소드들을 통해 각각의 원소들마다 layout을 잡아 줍니다. 그 후 Cell을 하나씩 생성해서 표시하는 순서입니다. 최종적으로 Cell을 표시하기 위해서 Context가 layout datasource layout datasource순으로 번갈아가면서 접근하고 Cell을 그려내는 모습을 확인 할 수 있었습니다.
layout을 하는데 가장 핵심적인 객체가 UICollectionViewLayoutAttributes 입니다. cell마다 하나의 attributes를 요구하며 이 요구 사항 대로 Collectionview가 Cell을 배치하게 됩니다. 다음 포스팅에서는 layout에 대해 조금 더 관철해보도록 하겠습니당