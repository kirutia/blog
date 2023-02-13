---
author: "naijun0403 on kirutia"
title: "Analyzing Reaction on Kakaotalk Profile"
date: "2023-02-02"
description: "Analyzing Reaction on Kakaotalk Profile with hooking"
tags: ["kakaotalk"]
ShowToc: false
ShowBreadCrumbs: false
---
## Analyzing Kakaotalk profile reaction
최근 카카오톡 프로필에 공감 기능이 추가되었습니다.
이 포스트를 통하여 공감 기능의 구조에 대하여 톺아보겠습니다.

## Viewing http packet
물론 http 패킷을 보기위해 `SSL Pinning bypass`를 하여 버프슈트와 같은 툴을 사용할 수 있습니다.
필자는 이를 매우 귀찮아 하니 `frida hook`을 하기로 하였습니다.

다행히 카카오톡은 `okhttp3` 라이브러리를 사용하여, 통신부 부분만 후킹하면 쉽게 `http packet`을 볼 수 있습니다
관련 내용은 나중에 따로 글을 적을 계획이니, 우선은 후킹 스크립트 작성은 넘어갑시다.

## Full-fledged hooking start
### Adding Reaction Item on profile
먼저 공감 아이템을 어떻게 프로필에 넣는지 분석을 하기위해서 후킹을 진행하였습니다.

먼저 프로필을 누르면
```
[+] Http Request!
[+] - Method: GET
[+] - Url: https://katalk.kakao.com/android/profile3/me?lastReactedAt=0
[+] - Headers: Authorization: accessToken-deviceUUID
A: android/10.0.6/ko
ADID: fda34d89-aba9-4524-bbd1-148f37de98a4
C: f12de75b-a58a-4120-a61e-3c72120c2979
Accept-Language: ko
User-Agent: KT/10.0.6 An/11 ko
Host: katalk.kakao.com
Connection: Keep-Alive
Accept-Encoding: gzip
```
이런식으로 요청이 가고, 서버 측에서는 이에대한 응답으로 프로필에 어떠한 아이템이 있는지 응답해줍니다.
뭐 보시는데로 헤더중에는 `Authorization`로 인증 헤더가 있습니다.
이는 간단한 분석을 통하여 발견할 수 있었습니다.

```java
String.format(Locale.US, "%s-%s", str, Hardware.a.t());
```

(`ADID`와 `C`는 그냥 `randomUUID` 값을 넣으면 되는거 같습니다.)

그리고

```
[+] Http Request!
[+] - Method: GET
[+] - Url: https://katalk.kakao.com/android/profile/item-catalog.json
[+] - Headers: Authorization: accessToken-deviceUUID
A: android/10.0.6/ko
ADID: fda34d89-aba9-4524-bbd1-148f37de98a4
C: 0da3fdb4-4b8d-44d5-8433-aa6fca342254
Accept-Language: ko
User-Agent: KT/10.0.6 An/11 ko
Host: katalk.kakao.com
Connection: Keep-Alive
Accept-Encoding: gzip
```
해당 요청으로 모든 프로필 아이템들은 불러오게 됩니다. (~~이유없이 기네요~~)

이제 정말 공감 아이템 하나를 추가해봅시다.

```
[+] Http Request!
[+] - Method: POST
[+] - Url: https://katalk.kakao.com/android/profile/update.json
[+] - Headers: Authorization: accessToken-deviceUUID
A: android/10.0.6/ko
ADID: fda34d89-aba9-4524-bbd1-148f37de98a4
C: f05160e2-5600-4888-9d1d-bc504e70c940
Accept-Language: ko
User-Agent: KT/10.0.6 An/11 ko
Content-Type: application/json; charset=UTF-8
Content-Length: 313
Host: katalk.kakao.com
Connection: Keep-Alive
Accept-Encoding: gzip
[+] - Body: {"decoration":[{"itemKind":"Inticker","itemId":"1","x":0.4424716,"y":0.3565341,"cx":0.49982244,"cy":0.38352272,"width":0.28813362,"height":0.1360631,"rotation":0.0,"parameters":{"name":"multiClick-b","intickerType":"multiClick","backgroundColor":"FFFFFFFF","textColor":"000000","emojiId":74}}],"music":{"ids":[]}}
```

공감 아이템은 카카오 내부에서 `Inticker`라고 불리는것으로 보입니다.

`itemId`, `emojiId`등 아이템에 대한 다양한 정보가 같이 나가는걸 볼 수 있습니다.

이후 이전에 본 `me` 요청이 다시 나가게 됩니다.

### Click inticker event
이제 공감 아이템을 클릭 하는걸 분석해봅시다.

다른 사람의 공감아이템을 클릭하면
```
[+] Http Request!
[+] - Method: POST
[+] - Url: https://talk-pilsner.kakao.com/talk/profile/inticker/reaction/multi_click
[+] - Headers: Authorization: accessToken-deviceUUID
talk-agent: android/10.0.6
talk-language: ko
Content-Type: application/json; charset=UTF-8
Content-Length: 67
Host: talk-pilsner.kakao.com
Connection: Keep-Alive
Accept-Encoding: gzip
User-Agent: okhttp/4.9.3
[+] - Body: {"intickerId":inticker_id,"userId":userId,"clickCount":count}
```
이런식으로 요청이 나가는걸 확인할수 있습니다.

생각보다 요청이 간단합니다, ~~아이디랑, 유저아이디, 그리고 클릭수는 1개의 요청시 최대 100개까지 가능한거 같습니다.~~
(2023/2/14 수정 - 관련 요청과 제한이 조금 바뀌었습니다. 후에 관련 부분을 작성하겠습니다.)

---------------------------------------------------------------------------------

지금까지는 간단한 맛보기였습니다.

다음 시간에는 이를 이용한 간단한 스크립트와, 이외의 방문수 아이템, 바 아이템 등을 알아보겠습니다
