# ARP_Spoofing

## 전제 :

해당 실습 문서는 host pc가 windows 10 운영체제로 구동된다는 전제로 작성되었음

윈도우 11에서는 아래 문서 내용을 따르면 실습이 가능하며,

Mac OS 를 사용하는 경우 타 hypervisor 를 사용해서 진행하면 된다.

## 준비 :

1. 가상 OS들을 구동시킬 VMware Workstation [0.5GB]

[https://www.vmware.com/content/vmware/vmware-published-sites/us/products/workstation-player/workstation-player-evaluation.html.html](https://www.vmware.com/content/vmware/vmware-published-sites/us/products/workstation-player/workstation-player-evaluation.html.html)

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled.png)

버전 무관, 무료버전 다운받아도 됩니다

1. Kali Linux ( 버전 상관 없음 ) [2.9GB]

[https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%201.png)

1. 우분투 16 [1.6GB]

[https://releases.ubuntu.com/16.04/](https://releases.ubuntu.com/16.04/)

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%202.png)

*꼭 16이 아니더라도 구버전이라면 실습 가능, 권장 사항일 뿐

**이미 최신 버전의 우분투가 설치되어 있는 경우 그 우분투로 실습해도 진행은 가능함 - 개념 이해에는 문제가 없지만 보호기법이 있어서 시범 화면과 차이가 있을 수 있음

---

VMware에서 제공하는 기본값으로 Kali Linux와 Ubuntu가 설치되었다는 전제로 작성되었다.

Kali linux의 설치 방법은 지난주에 ubuntu 설치한 방법과 동일하며, 다수의 블로그에서 설명하므로 생략하도록 한다.

---

이번 실습에서는 Kali Linux가 공격자, Ubuntu가 피해자 역할을 하게 된다.

Kali Linux를 부팅하고 접속해본다.

Kali Linux를 공식 이미지로 설치하면 ID와 PS가 걸려있다.

```c
대다수의 경우

ID : kali
PS : kali

이며 구버전일 경우

ID : kali
PS : toor
```

Kali Linux에서 `ifconfig` 를 쳐서 칼리리눅스의 ip를 확인해본다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%203.png)

위 이미지에서는 `192.168.194.128` 가 할당되었다.

Kali Linux에서 `ip route` 를 쳐서 해당 네트워크의 gateway 를 확인해본다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%204.png)

위 이미지에서는 `192.168.194.2` 가 gateway로 할당되었다.

다음으로 Ubuntu를 부팅하고 접속해본다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%205.png)

설치 시에 등록한 IP, PS로 인증하고 접속한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%206.png)

터미널을 실행시킨 후 `ifconfig` 를 실행시켜보겠다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%207.png)

시범 실습에서는 우분투의 IP가 192.168.194.130 으로 할당된 것을 확인할 수 있다.

*최신 버전의 우분투를 이용하는 경우 `sudo apt install net-tools` 를 먼저 실행시킨 후 `ifconfig` 를 실행한다.

`ip route` 를 통해 gateway를 확인해보겠다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%208.png)

당연하게도 kali linux에서 확인한 `192.168.194.2` 와 일치하는 것을 확인할 수 있다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%209.png)

ubuntu 내부에서 firefox 브라우저를 실행시켜, [naver.com](http://naver.com) 으로 접속해본다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2010.png)

현재 네트워크 구성을 그려보면 다음과 같다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2011.png)

네트워크 패킷의 흐름을 gateway 관점에서 보면, 내부 망의 ubuntu 가 naver와 같은 서버에 도달하기 위해서는 gateway를 반드시 통과해야 한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2012.png)

gateway 개념은 osi 7계층 중 3계층에 해당하는 개념이며, 따라서 3계층 이상의 장비가 gateway를 구성한다.

*gateway는 장비가 아닌 개념이다

목적지 ip를 보고 외부망으로 전달하는 역할을 gateway가 한다면, 내부망은?

일반적으로 가정의 경우, 3계층 이상 장비인 라우터가 사용된다.

라우터는 routing table을 기반으로, 도착지 ip주소를 보고 패킷을 전달한다.

오늘 arp spoofing 실습의 핵심을 구성하는 계층은 라우터의 하위 계층이다.

주로 2가지 형태로 구성되어 있다.

1. 허브 ( 더미 허브, 1계층 )
패킷을 모든 장비에 전달하는 방식

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2013.png)

1. 스위치 ( 스위칭 허브, 2계층 )
mac address table 을 가지고 있으며, 도착지 mac주소를 보고 전달
table에 등록되지 않은 mac주소 일 경우, 허브와 동일하게 작동함

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2014.png)

위 2가지 구성을 통해 아래와 같이 라우터 하위 네트워크를 구성한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2015.png)

이번 실습 환경을 적용해서 생각해보면

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2016.png)

으로 볼 수 있을 것이다.

---

### 불편한 진실

사실 VMware로 arp spoofing 실습을 100% 진행할 수 없다.

구글에 검색해보면 자료가 많이 나오는데, 대부분의 자료는 실제로 arp spoofing 공격이 성공하지 않은 실습이다.

network switch는 하나의 os를 구동하는 장치로, vmware의 가상 네트워크 연결은 switch역할을 하지 않고, hub 역할을 한다.

switch 역할을 한다면 정상적인 상황에서 ubuntu가 gateway 외부로 통신하는 내용이 kali linux에서 보이면 안된다. 하지만 이번 구성에서는 switch 역할을 못하고 hub 역할을 하기에, ubuntu의 통신 내용이 kali linux에서 확인된다.

그럼에도 vmware에서 실습을 진행하는 이유는, 해당 사실을 숙지하고 실습을 진행 할 경우, arp spoofing 이 발생하는 상황과 100% 일치하는 상황이 재현되며 개념 학습에는 부족함이 없기 때문이다.

만약에 100% 완벽한 arp spoofing 공격을 실습하고 싶다면 물리적인 switch 장비가 1대 이상 존재해야한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2017.png)

이 구성이 정상적인 switch로 구성했을 때의 네트워크 흐름이다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2018.png)

vmware의 가상네트워크(기본값)을 사용했을 때 실습 네트워크 흐름이다

지금부터 실습을 위해, kali linux ↔ ubuntu 는 switch 장비에 연결되어 있다고 가정하고 설명하겠다.

---

[기본 개념] ARP 패킷을 통해, IP 주소에 대응하는 MAC 주소를 수집한다

이전에 `ifconfig` 명령어를 통해 알아낸 정보를 바탕으로 유추해보면 switch의 arp table에는

| IP | MAC |
| --- | --- |
| 192.168.194.2 | 00:50:56:e7:1f:6a |
| 192.168.194.128 | 00:0c:29:0d:3d:76 |
| 192.168.194.130 | 00:0c:29:e0:b7:ce  |

이러한 값이 쓰여있을 것이다. ( gateway 의 mac 주소는 `arp` 명령을 통해 알 수 있다 )

Ubuntu와 Kali linux 에서 `arp` 명령을 실행시켜 보겠다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2019.png)

gateway의 mac 주소가 확인된다. 2개의 vm을 모두 부팅하고 충분한 시간이 지났다면 서로의 mac 주소도 확인될 수 있다.

ubuntu가 가지는 arp table은 아래와 같은 것이다.

| IP | MAC |
| --- | --- |
| 192.168.194.2 | 00:50:56:e7:1f:6a |
| 192.168.194.128 | 00:0c:29:0d:3d:76 |

ubuntu에서 [naver.com](http://naver.com) 등의 외부 네트워크로 접속을 시도하면 해당 패킷은 gateway로 전달되게 되며, 이때 gateway를 찾아가는 과정은 mac 주소를 기준으로 진행된다.

( gateway에서 외부로 나갈 때는 ip 주소를 기준으로 진행된다. )

이 때, gateway의 mac 주소를 kali linux의 mac 주소라고 거짓 정보를 흘리면, kali linux가 gateway 줄 알고 ubuntu의 패킷은 kali linux로 넘어갈 것이다.

여기서 다수의 블로그 글이 틀린 포인트가 있다. 오염시키는 arp table은 switch 장비가 아닌, 피해자 컴퓨터 1대 ( 특정 컴퓨터 )의 arp table을 오염시키는 것이다.

여기서부터는 모두 ubuntu 의 관점에서 확인되는 arp table이라는 점을 주의해야한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2020.png)

이런 상황에서

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2021.png)

gateway의 mac 주소가 kali linux의 mac 주소라는 arp 패킷(거짓 패킷)을 ubuntu가 받게되면

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2022.png)

ubuntu 관점에서 gateway의 mac 주소는 kali linux 의 mac주소가 된다.

따라서 ubuntu가 외부로 통신하는 모든 내용은 kali linux 로 전달되게 된다.

이 과정을 kali linux에서 진행하려면 아래와 같은 명령어를 사용하면 된다.

```c
sudo arpspoof -t <피해자 IP> <게이트웨이 IP>
```

위 케이스에서는 `sudo arpspoof -t 192.168.194.130 192.168.194.2` 이렇게 치면 된다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2023.png)

위 명령을 시작한 상태로 ubuntu 에서 `arp` 명령을 치면

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2024.png)

멈춘 상태에서는 

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2025.png)

*여기서 최신 버전의 우분투 일 경우 차이가 발생한다

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2026.png)

실습에 지장이 가진 않으나, 정상적인 mac address로 돌아오려는 노력을 한다.( kali linux가 지속적으로 공격하면 돌아오진 못한다 )

이 상황에서 ubuntu에서 firefox를 실행하고 naver.com을 접속하면 접속이 실패한다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2027.png)

패킷이 gateway로 안가고 kali linux로 빠지기 때문이다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2028.png)

ubuntu가 정상적으로 인터넷이 가능해야 kali linux에서 sniffing 할 내용이 있을 것이다. 따라서 kali linux 에 위와 같은 네트워크 구성을 해주겠다.

기존 명령어가 실행되고 있는 상태에서, 새로운 터미널을 하나 더 열어서 아래 명령어를 실행시킨다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2029.png)

```c
sudo fragrouter -B1
```

가끔 위 명령어를 사용하지 않은 상태로 ubuntu가 인터넷이 되는 경우가 있다. kali linux가 대신 해주었기 때문에 된 경우다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2030.png)

이제 ubuntu로 돌아가서 naver.com을 접속해보면 정상적으로 접속되는 것을 확인할 수 있다.

### Sniffing

지금까지 arp spoofing을 진행하였다.

arp spoofing의 목적이었던 sniffing을 실습해보겠다.

kali linux에서 wireshark를 실행하고 시작(파란 지느러미)해준다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2031.png)

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2032.png)

arp spoofing 이 진행중인 상태에서 ubuntu 의 firefox브라우저로

[http://server.zeropointer.co.kr](http://server.zeropointer.co.kr) ( sniffing 실습은 법적인 이슈가 있기에, 실습 전용 페이지를 제작하였음 )

을 접속한다.

[https://www.boannews.com/media/view.asp?idx=33794&kind=3](https://www.boannews.com/media/view.asp?idx=33794&kind=3)

임의 id, ps를 입력하고 submit을 누른다.

kali linux로 돌아와서 wireshark의 중지 버튼을 누른다. ( 빨간 정사각형 )

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2033.png)

연두색 배경의 `POST /login_dummy HTTP/1.1` 이라고 적힌 패킷을 찾고 더블클릭한다.

패킷을 가장 아래까지 스크롤 해본다. 패킷 body에 입력한 id와 ps가 확인된다.

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2034.png)

### 추가 실습

공격 탐지

![Untitled](ARP_Spoofing%205cd7cc28e310452a876638e00682563a/Untitled%2035.png)

kali linux 가 fake ARP packet 을 보내는 모습이 확인된다.

---

공격 방어

ARP Caching Table을 Static하게 MAC 주소를 등록하는 방법이 일반적이다.

Mac 주소를 정적으로 등록하여 가짜 ARP 패킷으로 인해 변경되지 않도록 하는 방법이다.
