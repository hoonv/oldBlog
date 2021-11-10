---
layout: post
title:  "CollectionView 관철하기 Layout"
date:   2021-07-30 08:45:14 +0900
categories: Swift
---
CollectionView 관철하기 2번째 포스팅입니다. CollectionView에 대해 공부하다 보니 글을 쓰면서도 아 이거는 따로 또 써야겠다 싶었던 토픽들을 하나씩 다룰 예정입니다 😃 이번에는 Layout에 대해 조금 더 자세히 알아보고 싶은데요, 이번에는 조금 실제로 구현을 해보며 Layout을 알아가려 합니다.
CollectionView는 Flow Layout을 사용하잖아요, 이 layout이 한줄을 쫙 채우죠?

<br/>
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883666-f44f40f8-eac0-45f9-8a81-8221b7e693dc.png" width="75%"></p>
이런식으로 한줄을 넘어갈때는 그 줄을 꽉 채우고 넘어가게 됩니다. 이게 의도할 수도 있지만 보통 Cell간에 간격이 일정하지 않아 보기에 이뻐 보이지 않습니다. 보통 Tag들을 표현할때는 아래 사진과 같이 왼쪽으로 정렬해서 표현합니다. 더 가지런해 보이고 정돈되어 보이죠 ㅎㅎ

<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883686-ef560e53-ea17-4d9f-8cf6-205421193a8e.png" width="75%"></p>
이것을 구현하기 위해서 기존에 존재하는 FlowLayout을 조금 수정해서 구현해보도록 하겠습니다. 그러기 위해서 Layout이 어떻게 동작하는지 조금 알아보면서 가보도록 할꼐요 ~
### ⚙️Layout Process
<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883706-593590f3-bdd9-47d1-987d-ce0af9ee7542.png" width="75%"></p>
apple에서 제공하는 그림 하나 보고 가시죠 CollectionView는 layout객체에게 prepareLayout, collectionViewContentSize, layoutAttributesForElementsInRect 세가지를 요구 합니다. 이 사진은 옛날 꺼라 메소드 이름이 조금 변경되었는데요 prepare, collectionViewContentSize, layoutAttributesForElements이렇게 이름이 변경되었으니 참고하시면됩니다. 즉 이 3가지의 메소드가 핵심입니다. 이 3가지만 부수고 갑시당~!

#### prepare
prepare단계는 layoutAttributes를 위한 준비 단계입니다. Cell마다 어떠한 속성값을 줄지 결정하는 단계는 세번째 layoutAttributesForElements 인데요, 이 단계를 위해 필요한 것들을 계산 해두기 위한 메소드입니다. 현재 우리가 하려는 tag는 FlowLayout을 상속해서 구현할 것이기 때문에 FlowLayout의 prepare만 호출하고 넘깁니다. 이부분이 필요한 것은 Circular layout이나 조금 더 복잡한 layout을 구현할때 사용할 것입니다.
``` swift
override func prepare() {
    print("Layout: prepare")
    super.prepare()
}
```
#### collectionViewContentSize

collectionViewContentSize는 collectionView의 ContentSize를 몇으로 할것인가 결정하는 변수입니다. ContentSize는 어떻게 보면 view의 bounds라고 할 수 있는데요, 실기기는 width가 약 400, height는 800정도입니다. 하지만 우리가 표현해야할 Size는 Cell이 20개 30개가 된다면 이 크기를 넘어가니 스크롤을 해야하잖아요? 따라서 그 안에 Content들의 Size를 결정하는 곳입니다. 이부분에서 FlowLayout은 안에 itemSize의 값을 통해 계산 하던가
``` swift
func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
}
```
Delegate의 이 method를 통해 하나하나 Cell을 호출한 후 Size를 결정하게 됩니다. 우리가하는 프로젝트에서도 FlowLayout의 도움을 얻어
``` swift
override var collectionViewContentSize: CGSize {
    print("Layout: collectionViewContentSize")
    return super.collectionViewContentSize
}
```
이렇게 진행하고 넘어가겠습니다.

#### layoutAttributesForElements

먼저 어떠한 속성들이 있는지 살펴보겠습니다
``` swift
@available(iOS 6.0, *)
open class UICollectionViewLayoutAttributes : NSObject, NSCopying, UIDynamicItem {


    open var frame: CGRect

    open var center: CGPoint

    open var size: CGSize

    open var transform3D: CATransform3D

    @available(iOS 7.0, *)
    open var bounds: CGRect

    @available(iOS 7.0, *)
    open var transform: CGAffineTransform

    open var alpha: CGFloat

    open var zIndex: Int // default is 0

    open var isHidden: Bool // As an optimization, UICollectionView might not create a view for items whose hidden attribute is YES

    open var indexPath: IndexPath
    ...
```
배치하기위한 frame과 zindex IndexPath를 포함하고 있습니다. 이것들로 Cell의 위치를 결정하는 것입니다. indexpath로 각각 지정된 Cell이있습니다. 배열의 원소 순서는 상관이 없습니다. 이 속성을 수정하여 Left align을 구현해보겠습니다.
``` swift
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        print("Layout: layoutAttributesForElements")
        guard let atts = super.layoutAttributesForElements(in: rect) else {
            return []
        }
        for i in atts {
            print(i.frame, i.indexPath.row)
        }
        return atts
    }
```

<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883729-e955f38f-4390-4c09-87ea-97524b575a43.png" width="50%">
<img src="https://user-images.githubusercontent.com/46335714/129883735-f611c198-e398-413a-8434-fa63b681a772.png" width="50%"></p>
기존 FlowLayout이 배치한 Attributes들을 print로 출력해보았습니다. indexPath 별로 하나씩 비교해보시면 매칭이 되실것입니다. FlowLayout이 이미 한줄에 어떠한 Cell이 배치될지 결정을 해놓았기 때문에 할일 이 대폭 줄었습니다. 우리는 Flow layout이 배치해둔 이 Cell들을 왼쪽으로 일정한 간격으로 밀어주면 됩니당.😇 layoutAttributesForElements의 frame을 직접 수정할 수 있습니다. 진행해보겠습니다~

``` swift
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        print("Layout: layoutAttributesForElements")
        guard let atts = super.layoutAttributesForElements(in: rect) else {
            return []
        }
        for i in atts {
            print(i.frame, i.indexPath.row)
        }
        /* 추가 */
        for (idx, val) in atts.enumerated() {
            if val.frame.origin.x == 10 {
                continue
            }
            val.frame.origin.x = atts[idx-1].frame.maxX + minimumLineSpacing
        }
        /*    */
        return atts
    }
```

origin.x == 10 을 비교하는 부분은 제가 CollectionView의 Inset을 10으로 주어서 10부터 시작했습니다. 즉 가장 왼쪽에 있는 Cell은 넘어가고, 그 다음 Cell부터 그 전 Cell의 가장 끝 부분에 + spacing으로 배치하는 모습입니다. 쉽죠 ~? 직접 이 속성들을 조작하면서 layout을 완성하면 됩니다.!


<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883745-3b93730f-7c89-4e59-b576-985cc5247114.png" width="50%"></p>

그럼 오른쪽이나 중앙으로도 가능할것 같지 않나요? 사실 오른쪽으로 정렬하는건 쓸만하지 않지만 한번 해볼까요 ~?!
``` swift
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        print("Layout: layoutAttributesForElements")
        guard let atts = super.layoutAttributesForElements(in: rect) else {
            return []
        }
        for i in atts {
            print(i.frame, i.indexPath.row)
        }
        for (idx, val) in atts.enumerated() {
            if val.frame.origin.x == 10 {
                continue
            }
            val.frame.origin.x = atts[idx-1].frame.maxX + minimumLineSpacing
        }
        let contentMaxX: CGFloat = collectionViewContentSize.width - 10
        let grouped = Dictionary(grouping: atts, by: { $0.frame.minY })
        grouped.values.forEach { atts in
            let maxXs = atts.map{ $0.frame.maxX }
            let diff = (contentMaxX - maxXs.max()!)
            atts.forEach { $0.frame.origin.x += diff }
        }
        return atts
    }
```

마지막에 group으로 한줄한줄 묶었구 Cell의 maxX로 가장 마지막Cell의 최대 X값을 구한다음 그 차이만큼 x값을 더해주면 모든 오른쪽 정렬을 할 수 있습니다.
가운데 정렬은 diif / 2로 반만큼만 이동하면 됩니다.

<p align="center"><img src="https://user-images.githubusercontent.com/46335714/129883753-8ee39ec2-a4b1-4ded-91c2-98212803df02.png" width="50%">
<img src="https://user-images.githubusercontent.com/46335714/129883761-d2a14911-9e54-4ce5-9441-7fecb75ee6dd.png" width="50%">
</p>
FlowLayout을 조금 조작해서 왼쪽 정렬 오른쪽 정렬 가운데 정렬 Layout을 구현해보았습니다. FlowLayout이 많은 것을 해주어서 생각보다 쉽게 구현할 수 있었습니다.