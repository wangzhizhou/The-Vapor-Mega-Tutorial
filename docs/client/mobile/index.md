允许https请求，不验证证书

info.plist 设置

App Transport Security Settings
|- Allow Arbitrary Loads = YES

```swift
  func session() -> URLSession {
    let session = URLSession(configuration: .default, delegate: self, delegateQueue: OperationQueue())
    return session
  }
 
  //https所有证书都验证通过
  func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge, completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
    let credential = URLCredential(trust: challenge.protectionSpace.serverTrust!)
    completionHandler(.useCredential, credential)
  }
```

iOS 项目比较烦琐，这里给一个链接地址吧: <https://github.com/wangzhizhou/TILiOS.git>