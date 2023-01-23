---
title: "SwiftUI Firebase Authentication Apple ID Signin"
# author: wrkholic84
date: 2023-01-23 00:00:00 +0900
categories: [Development, Swift]
tags: [swiftui, apple, firebase]
math: true
mermaid: true
---
## Third-party Login Service
요즘들어 모바일 앱에 애플 계정을 이용한 OAuth가 늘었나 했는데, 역시 이유가 있었다.

```
Guideline 4.8 - Design - Sign in with Apple

Your app uses a third-party login service, but does not offer Sign in with Apple. Apps that use a third-party login service for account authentication need to offer Sign in with Apple to users as an equivalent option.

Next Steps
Please revise your app to offer Sign in with Apple as an equivalent option for account authentication.

Resources
- Review Sign in with Apple sample code.
- For an overview of design and formatting recommendations for Sign in with Apple, see the Human Interface Guidelines .
- Learn about the benefits of Sign in with Apple.
```
다른 서드 파티 로그인 서비스를 사용할거면, 애플 로그인 기능도 추가해야 한단다.  
앱 심사에서 떨어지는 이유가 되었다. 내가 만든 앱은 구글 계정을 통한 로그인만 지원하고 있었는데, 결국 애플 계정을 이용한 로그인 기능도 추가하게 되었다.

## Firebase Authentication for Apple ID
Firebase 기반 서버리스 앱이라 인증, 데이터베이스, 메시징등의 기능을 Firebase에 의존하고 있다.  
Firebase는 구글의 서비스다보니 구글 계정을 인증하기 매우 편하게 만들어져있다. 또한, 다른 Third-party를 통한 인증도 제공하는데, 약간 까다로운 것 같다.  
다음은 SwiftUI 기반의 Firbase Authentication 클래스 코드다.
```swift
//
//  FirebaseAppleIDSignin.swift
//  SecretApp
//
//  Created by ME on 2023/01/22.
//

import Foundation

import SwiftUI
import CryptoKit
import FirebaseAuth
import FirebaseDatabase
import FirebaseMessaging
import AuthenticationServices

final class AppleIDViewModel: NSObject, ObservableObject, ASAuthorizationControllerDelegate, ASAuthorizationControllerPresentationContextProviding {
    
    var window: UIWindow?
    fileprivate var currentNonce: String?
    
    private lazy var databasePath: DatabaseReference? = {
        let ref = Database.database().reference().child("#####")
        return ref
    }()
    
    func startSignInWithAppleFlow() {
        let nonce = randomNonceString()
        currentNonce = nonce
        let appleIDProvider = ASAuthorizationAppleIDProvider()
        let request = appleIDProvider.createRequest()
        request.requestedScopes = [.fullName, .email]
        request.nonce = sha256(nonce)
        
        let authorizationController = ASAuthorizationController(authorizationRequests: [request])
        authorizationController.delegate = self
        authorizationController.presentationContextProvider = self
        authorizationController.performRequests()
    }
    
    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        if let appleIDCredential = authorization.credential as? ASAuthorizationAppleIDCredential {
            guard let nonce = currentNonce else {
                fatalError("Invalid state: A login callback was received, but no login request was sent.")
            }
            guard let appleIDToken = appleIDCredential.identityToken else {
                print("Unable to fetch identity token")
                return
            }
            guard let idTokenString = String(data: appleIDToken, encoding: .utf8) else {
                print("Unable to serialize token string from data: \(appleIDToken.debugDescription)")
                return
            }
            
            // Initialize a Firebase credential.
            let credential = OAuthProvider.credential(withProviderID: "apple.com",
                                                      idToken: idTokenString,
                                                      rawNonce: nonce)
            
            // Sign in with Firebase.
            Auth.auth().signIn(with: credential) { (authResult, error) in
                if (error != nil) {
                    print(error!.localizedDescription)
                    return
                }
                
                // 사용자의 이름은 애플 계정으로 처음 로그인 했을 때만 받아옴.
                if appleIDCredential.fullName?.description.isEmpty != true {
                    let userName = "\(appleIDCredential.fullName?.givenName ?? "Unknown") \(appleIDCredential.fullName?.familyName ?? "")"
                    Auth.auth().currentUser?.setValue(userName, forKey: "displayName")
                } else {    // 두번째로그인부터는 데이터베이스에서 이름 불러옴.
                    let user = Auth.auth().currentUser
                    var email:String = user?.email?
                    
                    guard let databasePath = self.databasePath else {
                        return
                    }
                    
                    // TODO retrieve username from DB and set it to Auth
                }
            }
        }
    }
    
    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        // Handle error.
        print("Sign in with Apple errored: \(error)")
    }
    
    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        return window ?? UIWindow()
    }
    
    private func sha256(_ input: String) -> String {
        let inputData = Data(input.utf8)
        let hashedData = SHA256.hash(data: inputData)
        let hashString = hashedData.compactMap {
            String(format: "%02x", $0)
        }.joined()
        
        return hashString
    }
    
    private func randomNonceString(length: Int = 32) -> String {
        precondition(length > 0)
        let charset: [Character] =
        Array("0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._")
        var result = ""
        var remainingLength = length
        
        while remainingLength > 0 {
            let randoms: [UInt8] = (0 ..< 16).map { _ in
                var random: UInt8 = 0
                let errorCode = SecRandomCopyBytes(kSecRandomDefault, 1, &random)
                if errorCode != errSecSuccess {
                    fatalError(
                        "Unable to generate nonce. SecRandomCopyBytes failed with OSStatus \(errorCode)"
                    )
                }
                return random
            }
            
            randoms.forEach { random in
                if remainingLength == 0 {
                    return
                }
                
                if random < charset.count {
                    result.append(charset[Int(random)])
                    remainingLength -= 1
                }
            }
        }
        
        return result
    }
}

```
## Apple ID를 이용한 인증의 보안
매우 놀랐다.  
나도 보안을 업으로 하고 있는 사람인데, 이정도일줄이야.. 다른 Third-party에서 많이 참고해야 할 것 같다.  
애플의 보안은 개발자를 많이 힘들게 하지만, 의미 있어 보인다.
### 1. Apple 익명 데이터 요구 사항 준수
Firebase 도큐먼트에 나와 있는 내용을 기준으로 설명한다.
Sign In with Apple은 사용자에게 로그인 시 이메일 주소를 포함한 데이터를 익명화하는 옵션을 제공한다.   
무슨 말인지는, 아래 그림을 보면 쉽게 알 수 있다. 
![00](/assets/images/posts/20230123AppleIDSignin/00.png)

사용자가 애플 계정을 이용하여 로그인할 때 Hide My Email 옵션을 선택하면, privaterelay.appleid.com 의 이메일 주소를 앱에 전달한다. 예를 들면, 4kfjeo31s@privaterelay.appleid.com 과 같은 식이다.

### 2. 사용자 정보 전달 제한
Apple은 사용자가 '처음' 로그인할 때 표시 이름과 같은 사용자 정보만 앱과 공유한다. 그런데 이후 앱에 로그인한 정보를 바탕으로 다시 로그인하려고 하면 Apple은 사용자의 표시 이름을 Firebase에 제공하지 않는다.  
이거때문에 시간을 얼마나 낭비했는지 모른다. 위 코드의 100번째 라인은 이런 이유로 작성되었다.

** 그림에서 이메일 정보를 공유하는 옵션(Share My Email)으로 로그인하면 앱에 정확한 이메일 정보를 전달해 주지만, 여전히 두번째 로그인 부터는 이름을 제공해주지 않는다.

