---
layout: post
current: post
cover:  assets/images/useful_log_cover.png
navigation: True
title: Logger for Useful Logs
date: 2020-08-18 10:47:00
tags: devlog
class: post-template
subclass: 'post'
author: seyoon
---
#### print("error") 보다 조금 더 쓸 모 있는 로그를 찍기 위한 함수
메시지에 자동으로 함수와 라인이 포함되어 표시되므로 디버깅에 유용하다.

```swift
func DEBUG_LOG(_ msg: Any, file: String = #file, function: String = #function, line: Int = #line) {
    let filename = file.split(separator: "/").last ?? ""
    let funcName = function.split(separator: "(").first ?? ""
    print("👻 [\(filename)] \(funcName)(\(line)): \(msg)")
}

func ERROR_LOG(_ msg: Any, file: String = #file, function: String = #function, line: Int = #line) {
    let filename = file.split(separator: "/").last ?? ""
    let funcName = function.split(separator: "(").first ?? ""
    print("👹 [\(filename)] \(funcName)(\(line)): \(msg)")
}
```
