---
layout: git
title: iOS应用内多平台登录功能开发指南
date: 2023-08-21 20:00:00
tags: OC, iOS
---

# iOS应用内多平台登录功能开发指南

在现代的移动应用中，提供多种第三方登录方式是非常重要的。本文将介绍如何在iOS应用中实现多平台的登录功能，包括Line、Apple ID、Google、Facebook、Twitter和TikTok等。本文将详细展示代码实现，并逐步解释每个实现步骤。

## 登录工具类实现

下面的代码展示了一个登录工具类`BLJSAPILogin`的实现，这个工具类支持多种登录方式。

### 登录工具类的声明

声明一个类`BLJSAPILogin`，用于处理各种第三方登录。

```objective-c
#import <Foundation/Foundation.h>
#import <AuthenticationServices/AuthenticationServices.h>
#import <FBSDKLoginKit/FBSDKLoginKit.h>
#import <GoogleSignIn/GoogleSignIn.h>
#import <FirebaseAuth/FirebaseAuth.h>
#import <TikTokOpenAuthSDK/TikTokOpenAuthSDK.h>
#import <LineSDK/LineSDK.h>
#import <TwitterKit/TwitterKit.h>

typedef void(^JSCallback)(NSDictionary *response, BOOL success);

@interface BLJSAPILogin : NSObject <ASAuthorizationControllerDelegate, ASAuthorizationControllerPresentationContextProviding, NSURLSessionDelegate>

@property(strong) JSCallback completionHandler;
@property(strong) JSCallback facebookCompletionHandler;

+ (instancetype)sharedInstance;

- (void)loginWithLine:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;
- (void)loginWithAppleId:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;
- (void)loginWithGoogle:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;
- (void)loginWithFacebook:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;
- (void)loginWithTwitter:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;
- (void)loginWithTiktok:(NSDictionary *)args completionHandler:(JSCallback)completionHandler;

@end
```

### Line登录

Line的登录主要是通过`LineSDK`来实现。

```objective-c
- (void)loginWithLine:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    NSSet *permissions = [NSSet setWithObjects:
                          [LineSDKLoginPermission profile],
                          [LineSDKLoginPermission email],
                          [LineSDKLoginPermission openID],
                          nil];
    
    LineSDKLoginManagerParameters *parameters = [[LineSDKLoginManagerParameters alloc] init];
    parameters.onlyWebLogin = NO;
    
    [[LineSDKLoginManager sharedManager] loginWithPermissions:permissions
                                             inViewController:self.jsAPIContext.container.viewController
                                                   parameters:parameters
                                            completionHandler:^(LineSDKLoginResult *result, NSError *error) {
        if (result) {
            NSDictionary *loginInfo = @{
                @"accessToken": result.accessToken.value,
                @"idToken": result.accessToken.IDToken.payload.email,
                @"email": result.accessToken.IDToken.payload.email,
                @"nick": result.userProfile.displayName,
                @"avatar": result.accessToken.IDToken.payload.picture.absoluteString
            };
            completionHandler([self successMutableDictionaryWithData:loginInfo], YES);
        } else {
            NSDictionary *errorInfo = @{
                @"code": @(error.code),
                @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown"
            };
            completionHandler(errorInfo, YES);
        }
    }];
}
```

### Apple ID登录

Apple ID登录主要是通过`AuthenticationServices`来实现。

```objective-c
- (void)loginWithAppleId:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    self.completionHandler = completionHandler;
    if (@available(iOS 13.0, *)) {
        ASAuthorizationAppleIDProvider *appleIDProvider = [[ASAuthorizationAppleIDProvider alloc] init];
        ASAuthorizationAppleIDRequest *authAppleIDRequest = [appleIDProvider createRequest];
        authAppleIDRequest.requestedScopes = @[ASAuthorizationScopeFullName, ASAuthorizationScopeEmail];
        
        ASAuthorizationController *authorizationController = [[ASAuthorizationController alloc] initWithAuthorizationRequests:@[authAppleIDRequest]];
        authorizationController.delegate = self;
        authorizationController.presentationContextProvider = self;
        [authorizationController performRequests];
    }
}

- (void)authorizationController:(ASAuthorizationController *)controller didCompleteWithAuthorization:(ASAuthorization *)authorization {
    if ([authorization.credential isKindOfClass:[ASAuthorizationAppleIDCredential class]]) {
        ASAuthorizationAppleIDCredential *credential = (ASAuthorizationAppleIDCredential *)authorization.credential;
        NSDictionary *loginRes = @{
            @"userIdentifier": credential.user,
            @"email": credential.email ?: @"",
            @"nick": credential.fullName.nickname ?: @"",
            @"authorizationCode": [[NSString alloc] initWithData:credential.authorizationCode encoding:NSUTF8StringEncoding] ?: @"",
            @"idToken": [[NSString alloc] initWithData:credential.identityToken encoding:NSUTF8StringEncoding] ?: @""
        };
        if (self.completionHandler) {
            self.completionHandler([self successMutableDictionaryWithData:loginRes], YES);
        }
    }
}

- (void)authorizationController:(ASAuthorizationController *)controller didCompleteWithError:(NSError *)error {
    NSDictionary *errorInfo = @{
        @"code": @(error.code),
        @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown"
    };
    if (self.completionHandler) {
        self.completionHandler(errorInfo, YES);
    }
}

- (nonnull ASPresentationAnchor)presentationAnchorForAuthorizationController:(nonnull ASAuthorizationController *)controller  API_AVAILABLE(ios(13.0)){
    return self.jsAPIContext.container.viewController.view.window;
}
```

### Google登录

Google登录主要是通过`GoogleSignIn`来实现。

```objective-c
- (void)loginWithGoogle:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    GIDConfiguration *signInConfig = [[GIDConfiguration alloc] initWithClientID:@"YOUR_CLIENT_ID"];
    [GIDSignIn.sharedInstance signInWithConfiguration:signInConfig
                             presentingViewController:[UINavigationController currentViewController]
                                             callback```objective-c
                                             callback:^(GIDGoogleUser * _Nullable user, NSError * _Nullable error) {
        if (error) {
            NSDictionary *errorInfo = @{
                @"code": @(error.code),
                @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown"
            };
            completionHandler(errorInfo, YES);
            return;
        }
        
        if (user) {
            [user.authentication doWithFreshTokens:^(GIDAuthentication * _Nullable authentication, NSError * _Nullable error) {
                if (error) {
                    NSDictionary *errorInfo = @{
                        @"code": @(error.code),
                        @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown"
                    };
                    completionHandler(errorInfo, YES);
                    return;
                }
                
                if (authentication) {
                    NSDictionary *loginInfo = @{
                        @"idToken": authentication.idToken,
                        @"email": user.profile.email,
                        @"nick": user.profile.name,
                        @"avatar": [user.profile imageURLWithDimension:88].absoluteString
                    };
                    completionHandler([self successMutableDictionaryWithData:loginInfo], YES);
                }
            }];
        }
    }];
}
```

### Facebook登录

Facebook登录主要是通过`FBSDKLoginKit`来实现。

```objective-c
- (void)loginWithFacebook:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    FBSDKLoginManager *loginManager = [[FBSDKLoginManager alloc] init];
    [self facebookLogOut];
    self.facebookCompletionHandler = completionHandler;
    
    [loginManager logInWithPermissions:@[@"public_profile", @"email"]
                    fromViewController:[UINavigationController currentViewController]
                               handler:^(FBSDKLoginManagerLoginResult *result, NSError *error) {
        if (error) {
            NSDictionary *errorInfo = @{
                @"code": @(error.code),
                @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown"
            };
            completionHandler(errorInfo, YES);
            self.facebookCompletionHandler = nil;
        } else if (result.isCancelled) {
            NSDictionary *errorInfo = @{
                @"code": @(100),
                @"message": @"cancel",
                @"isCancel": @(result.isCancelled)
            };
            completionHandler(errorInfo, YES);
            self.facebookCompletionHandler = nil;
        } else {
            [self handleFacebookLoginResult:result];
        }
    }];
}

- (void)handleFacebookLoginResult:(FBSDKLoginManagerLoginResult *)result {
    FBSDKProfile *profile = [FBSDKProfile currentProfile];
    if (profile && result.token) {
        NSDictionary *loginInfo = @{
            @"accessToken": result.token.tokenString,
            @"idToken": result.token.tokenString,
            @"email": profile.email,
            @"nick": profile.name,
            @"avatar": profile.imageURL.absoluteString
        };
        self.facebookCompletionHandler([self successMutableDictionaryWithData:loginInfo], YES);
    } else {
        NSDictionary *errorInfo = @{
            @"code": @(100),
            @"message": @"No token or profile"
        };
        self.facebookCompletionHandler(errorInfo, YES);
    }
    self.facebookCompletionHandler = nil;
}

- (void)facebookLogOut {
    FBSDKLoginManager *loginManager = [[FBSDKLoginManager alloc] init];
    [loginManager logOut];
}
```

### Twitter登录

Twitter登录主要是通过`FirebaseAuth`和`TwitterKit`来实现。

```objective-c
- (void)loginWithTwitter:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    self.twitterProvider = [FIROAuthProvider providerWithProviderID:@"twitter.com"];
    
    [self.twitterProvider getCredentialWithUIDelegate:nil completion:^(FIRAuthCredential *credential, NSError *error) {
        if (credential) {
            [[FIRAuth auth] signInWithCredential:credential completion:^(FIRAuthDataResult *authResult, NSError *error) {
                if (error) {
                    [self handleTwitterLoginError:error source:@"sign_in" completionHandler:completionHandler];
                } else {
                    if ([credential isKindOfClass:[FIROAuthCredential class]]) {
                        FIROAuthCredential *OAuthCredential = (FIROAuthCredential *)authResult.credential;
                        NSDictionary *loginInfo = @{
                            @"authToken": OAuthCredential.accessToken ?: @"",
                            @"authTokenSecret": OAuthCredential.secret ?: @"",
                            @"name": authResult.user.displayName ?: @"",
                            @"sub": authResult.user.uid ?: @"",
                            @"deviceId": [self deviceID] ?: @"",
                            @"email": authResult.user.email ?: @"",
                            @"picture": authResult.user.photoURL.absoluteString ?: @""
                        };
                        completionHandler([self successMutableDictionaryWithData:loginInfo], YES);
                        self.twitterProvider = nil;
                    } else {
                        [self handleTwitterLoginError:error source:@"credential_is_not_oauth" completionHandler:completionHandler];
                    }
                }
            }];
        } else {
            [self handleTwitterLoginError:error source:@"no_credential" completionHandler:completionHandler];
        }
    }];
}

- (void)handleTwitterLoginError:(NSError *)error source:(NSString *)source completionHandler:(JSCallback)completionHandler {
    NSDictionary *errorInfo = @{
        @"code": @(error.code),
        @"message": error.userInfo[NSLocalizedDescriptionKey] ?: @"unknown",
        @"errorSource": source,
        @"data": @{}
    };
    completionHandler(errorInfo, YES);
    self.twitterProvider = nil;
}
```

### TikTok登录

TikTok登录主要是通过`TikTokOpenAuthSDK`来实现。

```objective-c
- (void)loginWithTiktok:(NSDictionary *)args completionHandler:(JSCallback)completionHandler {
    self.completionHandler = completionHandler;
    NSString *redirectUri = @"YOUR_REDIRECT_URI";
    TTKSDKAuthRequest *authRequest = [[TTKSDKAuthRequest alloc] initWithScopes:[NSSet setWithArray:@[@"user.info.basic"]] redirectURI:redirectUri];
    
    [authRequest send:^(id<TTKSDKBaseResponse> response) {
        if (response == nil) {
            NSDictionary *errorInfo = @{
                @"code": @(-100),
                @"message": @"unknown",
                @"data": @{}
            };
            completionHandler(errorInfo, YES);
        } else {
            TTKSDKAuthResponse *authResponse = (TTKSDKAuthResponse *)response;
            if (authResponse.errorCode == 0) {
                [self fetchTiktokAccessToken:authResponse codeVerifier:authRequest.pkce.codeVerifier];
            } else {
                NSDictionary *errorInfo = @{
                    @"code": @(authResponse.errorCode),
                    @"message": @"unknown",
                    @"data": @{}
                };
                completionHandler(errorInfo, YES);
            }
        }
    }];
}

- (void)fetchTiktokAccessToken:(TTKSDKAuthResponse *)response codeVerifier:(NSString *)codeVerifier {
    NSDictionary *params = @{
        @"code": response.authCode ?: @"",
        @"client_key": @"YOUR_CLIENT_KEY",
        @"grant_type": @"authorization_code",
        @"code_verifier": codeVerifier,
        @"redirect_uri": @"YOUR_REDIRECT_URI",
        @"client_secret": @"YOUR_CLIENT_SECRET"
    };
    
    NSMutableArray *parameterArray = [NSMutableArray array];
    for (NSString *key in params.allKeys) {
        NSString *encodedKey = [key stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
        NSString *value = params[key];
        NSString *encodedValue = [value stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
        NSString *parameterString = [NSString stringWithFormat:@"%@=%@", encodedKey, encodedValue];
        [parameterArray addObject:parameterString];
    }
    
    NSString *queryString = [parameterArray componentsJoinedByString:@"&"];
    NSData *httpBodyData = [queryString dataUsingEncoding:NSUTF8StringEncoding];
    
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];
    NSURL *url = [NSURL URLWithString:@"https://open.tiktokapis.com/v2/oauth/token/"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:30.0];
    [request addValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    [request setHTTPMethod:@"POST"];
    request.HTTPBody = httpBodyData;
    
    NSURLSessionDataTask *postDataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (data) {
            NSDictionary *resDic = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil];
            NSString *accessToken = resDic[@"access_token"];
            NSString *openID = resDic[@"open_id"];
            if (accessToken && openID) {
                [self fetchTiktokUserInfoWithAccessToken:accessToken openID:openID];
            } else {
                NSDictionary *errorInfo = @{
                    @"code": @(error.code),
                    @"message": @"unknown",
                    @"data": @{}
                };
                self.completionHandler(errorInfo, YES);
            }
        } else {
            NSDictionary *errorInfo = @{
                @"code": @(error.code),
                @"message": @"unknown",
                @"data": @{}
            };
            self.completionHandler(errorInfo, YES);
        }
    }];
    [postDataTask resume];
}

- (void)fetchTiktokUserInfoWithAccessToken:(NSString *)accessToken openID:(NSString *)openID {
    NSString *domain = @"https://open.tiktokapis.com/v2/user/info/";
    NSURLComponents *urlComp = [NSURLComponents componentsWithString:domain];
    urlComp.queryItems = @[
        [NSURLQueryItem queryItemWithName:@"fields" value:@"open_id,union_id,avatar_url,display_name"]
    ];
    NSURL *url = urlComp.URL;
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request addValue:[NSString stringWithFormat:@"Bearer %@", accessToken] forHTTPHeaderField:@"Authorization"];
    
    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (data) {
            NSDictionary *deserializedObject = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil];
            NSDictionary *userInfo = deserializedObject[@"data"][@"user"];
            if (userInfo) {
                NSDictionary *loginInfo = @{
                    @"authToken": accessToken,
                    @"name": userInfo[@"display_name"],
                    @"sub": userInfo[@"union_id"],
                    @"authTokenSecret": @"YOUR_CLIENT_SECRET",
                    @"picture": userInfo[@"avatar_url"],
                    @"deviceId": [self deviceID]
                };
                self.completionHandler([self successMutableDictionaryWithData:loginInfo], YES);
            }
        } else {
            NSDictionary *errorInfo = @{
                @"code": @(error.code),
                @"message": @"unknown",
                @"data": @{}
            };
            self.completionHandler(errorInfo, YES);
        }
    }];
    [task resume];
}
```

## 总结

本文详细介绍了如何在iOS应用中实现多种第三方登录功能，包括Line、Apple ID、Google、Facebook、Twitter和TikTok。通过上述示例代码，你可以在自己的项目中轻松添加这些登录功能。希望这篇文章对你有所帮助。