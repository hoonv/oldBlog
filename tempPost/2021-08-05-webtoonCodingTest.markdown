---
layout: post
title:  "CodingTest 정리"
date:   2021-08-13 08:45:14 +0900
categories: Swift
---

### P1

```swift
import Foundation
func solution(_ win_lose:[Int]) -> Int{
    var answer = 0
    var ret: [Int] = []
    for num in win_lose {
        if num == 1 {
            answer += 1
        } else {
            ret.append(answer)
            answer = 0
        }
    }
    ret.append(answer)
    return ret.max()!
}

let a = [0, 1, 0, 1, 1, 1, 0, 1, 1]

print(solution(a))
```
문제는 잘 기억나지 않지만. 이기면 1 지면 0인 배열이 입력되는데 최대 몇연승했는지 물어보는 문제였다.
for문을 쭉 탐색하면서 1이면 answer을 하나씩 올리고 지면 0으로 초기화하고 그전까지 올렸던 승을 
정답 후보에 등록하는 식으로 해결했다.

### P2
``` swift
struct Cord {
    let minX: Int
    let minY: Int
    let maxX: Int
    let maxY: Int
}

func fillEdge(board: inout [[Int]], cord: Cord, startNum: Int, clock: Bool) {
    if cord.maxX == cord.minX && cord.maxY == cord.minY {
        board[cord.maxY][cord.maxY] = startNum
        return
    }
    var num = startNum
    for i in cord.minX..<cord.maxX {
        let y = clock ? cord.minY : cord.maxY
        board[y][i] = num
        num += 1
    }
    num = startNum
    for i in cord.minY..<cord.maxY {
        let x = clock ? cord.maxX : cord.minX
        board[i][x] = num
        num += 1
    }
    num = startNum
    for i in ((cord.minX+1)...cord.maxX).reversed() {
        let y = clock ? cord.maxY : cord.minY
        board[y][i] = num
        num += 1
    }
    num = startNum
    for i in (cord.minY+1...cord.maxY).reversed() {
        let x = clock ? cord.minX : cord.maxX
        board[i][x] = num
        num += 1
    }
}


func solution(_ n:Int, _ clockwise:Bool) -> [[Int]] {
    var board = Array(repeating: Array(repeating: 0, count: n), count: n)
    let loop = Int(ceil((Double(n) / 2)))
    var startNum = 1
    for i in 0..<loop {
        let cord = Cord(minX: i, minY: i, maxX: n-i-1, maxY: n-i-1)
        fillEdge(board: &board, cord: cord, startNum: startNum, clock: clockwise)
        startNum += cord.maxX - cord.minX
    }
    return board
}
[01, 02, 03, 04, 05, 06, 07, 01]
[07, 08, 09, 10, 11, 12, 08, 02]
[06, 12, 13, 14, 15, 13, 09, 03]
[05, 11, 15, 16, 16, 14, 10, 04]
[04, 10, 14, 16, 16, 15, 11, 05]
[03, 09, 13, 15, 14, 13, 12, 06]
[02, 08, 12, 11, 10, 09, 08, 07]
[01, 07, 06, 05, 04, 03, 02, 01]
```

숫자 토네이도 만드는 문제가 나왔다. 시계방향과 반시계방향으로 나뉘는데 먼저 사이즈만큼 배열을 만든후 겉에 한 꺼플 한 꺼플로
채워 나가면서 배열을 완성했다. 4개의 사이즈라면 2번만하면되고 5번이라면 3번을 해야 해서 ceil로 맞춰주었고
매번 채워야하는 위치와 숫자가 동일했기때문에 for문으로 진행했고, 시계냐 반시계냐에 따라 채워야하는 위치가 대칭적으로
정해졌기떄문에 이것을 처리했따.

### P3
```swift
func solution(_ play_list:[Int], _ listen_time:Int) -> Int {
    let playList = play_list + play_list
    var heardMusic: [Int] = []

    var low = 0
    var high = 0
    var ret = 0

    while high < playList.count {
        var sum = heardMusic.reduce(0, +)
        if let first = heardMusic.first {
            sum = sum - first + 1
        }
        if sum < listen_time {
            heardMusic.append(playList[high])
            high += 1
        } else {
            heardMusic.removeFirst()
            low += 1
        }
        ret = max(ret, heardMusic.count)
    }

    return min(play_list.count, ret)
}

let play_list = [1, 2, 3, 4]
let listen_time = 5

print(solution(play_list, listen_time))
```
이부분은 투포인터, 슬라이딩 윈도우로 해결했다. 주어진 음악을 1분이라도 들으면 들은것으로 인정이 되는데 최대 몇개의 음악을
들을 수 있는지 판단하는것으로, CirCular로 진행되기 떄문에 배열 두개를 합쳐서 진행했다.
또한 가장 앞에 포함된 음악은 1분만 들어도 인정되기 떄문에 다 들을 필요없이 1만 추가해서 sum을 판단했고
Sum보다 작다면 음악을 하나 추가했따. 추가하고 나서 listen_time이 넘어도 가능한것이
마지막음악도 1분만 들으면 되기 떄문이다. sum이 넘는다면 가장 앞에 것을 빼고 low를 늘려주엇다.