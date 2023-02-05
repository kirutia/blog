---
author: "kirutia"
title: "about noonoo.tv"
date: "2023-02-01"
description: "analysis about noonoo.tv"
tags: ["analysis"]
ShowToc: false
ShowBreadCrumbs: false
---

- [noonoo.tv](https://ndonoo.tv)는 파라과이에서(명목상) 운영되고 있는
불법 스트리밍 사이트이다.

- [censys.io](https://censys.io)를 사용하여 서버를 스캔한 결과,
미국에 위치한 Hivelocity사의 베어메탈 클라우드 서비스를 사용했던 것으로 보인다. (ip: [104.254.128.158](https://search.censys.io/hosts/104.254.128.158))

- dns history를 [조회](https://securitytrails.com/domain/noonoo.tv)해본 결과, NuCDN LLC 사를 이용하고 있다.

- 대한민국 ISP에서 도메인을 차단할 시, 빠른 주기로 도메인을 변경하고 있다.
	- noonoo[n].tv 형태로 도메인이 변경된다.
	- 오래된 도메인을 통하여 접근시, NuCDN 사의 ip로 전달, 새로운 도메인으로 리다이렉션 된다.

- noonoo.tv를 소유하고 있는 것으로 보이는 studiouniversal.net 의 서브도메인을 통하여 cdn과 플레이어 등을 처리하고 있다.
	-  [서브도메인 리스트](https://subdomainfinder.c99.nl/scans/2023-02-01/studiouniversal.net)

- 동영상 cdn으로는 BunnyCDN을 사용한다. (ip: 143.244.49.183)

- DMCA Takedown을 위한 [저작권 센터](https://noonoo26.tv/copyright-report/)를 운영하고 있지만, 실제로 내용을 기입하고 제출을 하더라도 서버에 업로딩되는 데이터는 없다.

- 동영상 플레이어로는 jwplayer를 사용한다.
