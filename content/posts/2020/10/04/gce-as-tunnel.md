---
title: "Home → Public #0"
date: 2020-10-04T22:24:39+09:00
tags:
- home-cloud
- vpn
- IPSec
---

## ✏️ 글을 쓰는 배경

OpenStack 을 이용한 프라이빗 클라우드 구축은 [2018 컨트리뷰톤][contributhon-2018] 이후로 계속 꾸준히 가지고 있던 작은 목표였다. 이번에 데스크탑을 구매하면서 사용하다보니 상당히 많은 리소스가 놀고 있는 것을 보았고 굳이 비싼 클라우드 인스턴스를 사용하지 않더라도 터널링만 잘 되면 로컬에서 서버를 운영해도 괜찮을 거라는 생각이 들었다. 그래서 여러가지 시도를 하고 있고 그 중 GCE free instance 위에서 IPSec VPN 구축해본 경험에 대한 이야기를 간략히 적어보려고 한다.

[contributhon-2018]: https://www.oss.kr/contributhon_history?category_item_id=374

## 💭 ngrok을 고려하지 않은 이유

굳이 GCE free instance가 아니어도 TCP 터널을 제공하는 서비스에는 [ngrok]이 있다. 개발 단계에서는 무료로 사용할 수 있으며 특정 고정 hostname 및 port를 선점하여 [CNAME]을 하기 위해서는 유료 결제를 해야하긴 하다. 간략히 ngrok를 사용하려고 하지 않았던 이유를 나열하면 아래와 같다.

1. **커넥션 제한** [^1]
1. 유료 구독 사용

가능하면 추가적인 비용없이 무료로 제공되는 GCE 인스턴스 내에서 처리하고 싶었다. 하지만 필히 트래픽 때문에 비용이 나갈 예정이었기 때문에 유료 구독 비용[^2]은 크게 문제되지 않았다. 가장 문제될 거라고 생각했던 부분은 커넥션 제한이었다. 분 당 20 커넥션이라는 말은 HTTP로 치면 분당 20개의 request 밖에 못 받는다는 것처럼 느껴졌고 그래서 굉장히 꺼려졌다. 지금 이 글을 작성하는 시점에는 테스트 해보니 이전 문장에서 설명한 *connections/minute*의 의미와는 상이해서 ngrok을 사용하는게 좋았을까 하는 생각을 조금 하고 있다. 😂

[ngrok]: https://ngrok.com/
[CNAME]: https://en.wikipedia.org/wiki/CNAME_record

[^1]: 40 conections per minute
[^2]: $5 per month

## 🔨 설치

그래서 방법을 찾던 중 간단히 고등학교 네트워크 시간에 배운 것이 기억나 VPN으로 연결하고 GCE 쪽에서 포트 포워딩만 잘 해주면 괜찮지 않을까 싶었다. 그래서 VPN으로 방향을 잡고 찾던 중 IPSec 가 있었고 쉬운 [설치 스크립트][setup-ipsec-vpn][^3]도 제공되고 있었다. 추가적으로 적을 것도 없이 VPN 설정은 설치 스크립트만으로 충분했다. 그저 작업하면서 cisco 쉘로  네트워크 작업하고 싶다는 생각이 잠깐 들었다.

[setup-ipsec-vpn]: https://github.com/hwdsl2/setup-ipsec-vpn

[^3]: 비공식

## 😔 문제점

IPSec로 VPN을 구성하여 데스크탑과 연결하는데는 성공했다. 그리고 외부에서 데스크탑의 로컬 서버로 리퀘스트를 보내어 받아갈 수도 있었다. 하지만 문제점이 하나 있었다. Windows에서 기본적으로 제공되는 VPN 기능을 사용하여 연결하였더니 서비스가 아닌 개인적인 웹 서핑 등으로 인한 트래픽도 모두 VPN을 통해 오갔다. 덕분에 트래픽 수치가 상승했고 결국엔 월 결제로 카드에서 나갔다. 웃음 밖에 나오지 않았다 😥.

## 👟 다음 행보

그래서 ngrok 과 비슷한 터널링을 제공하는 오픈소스 프로젝트가 있을까 싶어서 찾아보다가 프로젝트들이 라이센스 때문에 사용하기가 어려워 보여서 [Kotlin]을 공부할 겸 직접 만들어보고자 한다.[^4]

[Kotlin]: https://kotlinlang.org
[^4]: 물론 이 또한 MIT License로 제공되는 프로젝트가 있었다. https://github.com/jpillora/chisel
