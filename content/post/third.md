---
title: "Third"
date: 2018-10-27T00:14:31+03:00
draft: false
topics: ["to_swift", "to_xcode"]
tags: ["tag_swift", "tag_xcode"]
description: spf13-vim is a cross platform distribution of vim plugins and resources for Vim.
---

Perhaps even more exciting is news of a MacBook Air successor with a high-res screen. It’s still not clear exactly what we’re in for — whether it’ll be updated parts in the Air’s body, a brand new take on the Air, or just an alternate version of the MacBook. But it’s clear that some kind of much-needed update is coming. Bloomberg says it’ll have a 13-inch screen, which means that at the very least, it’ll be different than today’s MacBook.


```swift

let myNames: [String] = ["John", "Joe", "Jack"]

// No need to flatMap (or compactMap)
let myCounts = myNames.flatMap { $0.count }
// [4, 3, 4]

// map is enough
let myCounts = myNames.map { $0.count }
// [4, 3, 4]

import Foundation
	
func count(word: String, in filePath: String) throws -> Int {
    let fileContent = try String(contentsOf: URL(fileURLWithPath: filePath))
    let words = fileContent
        .replacingOccurrences(of: "[^\\w']+", with: ",", options: .regularExpression)
        .components(separatedBy: ",")
    return words.filter { $0 == word }.count
}
	
guard CommandLine.argc == 3 else {
    print("Usage: command word file")
    exit(1)
}
	
do {
    try print(count(word: CommandLine.arguments[1], in: CommandLine.arguments[2]))
} catch {
    print(error.localizedDescription)
    exit(1)
}
```


Mac Mini fans will also find their immense (and honestly questionable) patience rewarded, as the much-neglected PC will get a spec update for the first time since 2014. Bloomberg says it’ll get new processors and some other features meant for pro users. Updated iMacs and iMac Pros are also in the works, as is a processor bump for the 12-inch MacBook, but it’s not clear if those will all be ready for the event.