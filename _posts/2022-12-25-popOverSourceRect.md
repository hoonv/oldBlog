---
layout: post
title: popOver sourceRect center
description: popOver
summary: popOver SourceRect
tags: swift
minute: 1
---

UIPopoverPresentationController의 instance Property인 sourceRect를 알아보려고 합니다. 
sourceRect란 소스뷰 안에서 popOver가 가르킬 영역 (닻을 내릴) 입니다. 이 사각형은 반드시 sourceView의 좌표계 안에 있어야 합니다.


기본적으로는 arrow가 가르킬 영역을 선택해주면 됩니다. 하지만 arrow 없이 중앙에 띄우고 싶은 경우도 있습니다. 우리앱의 스펙이 그런데, 이 경우엔 어떻게 하는게 좋을까요?  
먼저는 arrow가 없이 띄우고 싶다면 
``` swift
permittedArrowDirections = []
``` 
을 통해 arrow의 위치를 정해주지 않으면 됩니다.. UIPopoverArrowDirection은 optionSet을 채택하고 있어서 permittedArrowDirections = [.up, .down] 같이 여러개의 옵션을 할당 할 수 있는데, 빈 배열은 어떠한 값도 주지 않겠다라는 의미가 됩니다.  

그다음 sourceRect를 설정해주면 됩니다. 기본적으로는 popOver의 arrow가 가르킬 영역을 의미하지만 permittedArrowDirections = [] 인 경우엔 popOver의 center와 sourceRect의 center를 맞추도록 띄워지는걸 확인했습니다. 따라서 
``` swift
sourceRect = targetView.bounds // or
sourceRect = CGRect(x: targetView.bounds.midX, y: ~.~.midY, width: 0, height: 0)
```
으로 할당 해주어도 됩니다. 하지만 이러한 값이 명확하지 않다고 생각 할 수 있습니다. 그럴땐 sourceRect의 어떠한 값도 할당하지 않는것도 방법이 될 수 있습니다. 기본값은 CGRectZero라고 설명하고 있는데
실제 값을 확인해보니 CGRect(inf, inf, 0, 0)으로 확인됩니다. 이 기본값 또한 popOver를 center에 띄우고 있습니다.
