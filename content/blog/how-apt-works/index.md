---
title: apt-get이 동작하는 원리
date: 2021-08-13
description: "install 전 update를 해야 하는 이유"
tags: [Ubuntu, Command]
---

# 개요
데비안 계열 리눅스는 deb 패키지 시스템을 사용한다. deb 패키지 시스템을 관리하는 툴 중 하나로 `dpkg`가 있다. 이 툴을 사용하여 패키지를 설치하거나, 삭제하거나, 정보를 확인하는 등의 일을 할 수 있다. 하지만 `dpkg`는 의존성 문제가 존재하여 이를 해결하기 위해 `APT(Advanced Package Tool)`라는 툴이 등장했다<sup id="rfn-1">[\[1\]](#fn-1)</sup>.

> ## APT? apt?
> 여기서 말하는 `APT`는 `apt`와 다르다는 점에 유의하기 바란다. `APT`는 `dpkg`의 프론트엔드로써 설계된 패키지 매니저이며, 커맨드 라인 프로그램 `apt`, `apt-get`, `apt-cache` 등을 포함한다.

> ## apt와 apt-get의 차이
> `apt-get`은 패키지 설치를 담당하는 프로그램이고, 위에서 잠깐 언급한 `apt-cache`는 패키지 검색을 담당하는 프로그램이다. 오래 전부터 개발되었기 때문에 사용할 수 있는 옵션도 많다(대부분의 리눅스 사용자는 절대 사용할 일이 없는 옵션도 수두룩하다). `apt`는 이 둘을 대체하기 위해 만들어졌으며, 많이 사용되는 옵션만 포함시킨 **end-user용 프로그램**이라고 할 수 있다.

제목에는 `apt-get`이 동작하는 원리라고 썼지만, `apt`가 동작하는 원리라고 봐도 무방하다.

# 동작 원리
`apt-get update`와 `apt-get install` 명령어의 역할을 살펴봐야 한다. `apt-get`을 사용해본 사람이라면 어떤 패키지를 설치하기 위해 `apt-get install` 명령어를 사용했을 때 다음과 같은 메시지를 맞닥뜨린 적이 있을 것이다.
```bash
$ apt-get install nodejs
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package nodejs
```
`apt-get install` 명령어를 입력하고 나면 `apt-get`은 /var/lib/apt/lists 디렉터리를 참조한다. 해당 디렉터리에는 패키지 목록이 있고, 이를 이용하여 맵핑된 주소를 따라가 패키지를 다운받고 설치하는 방식이다. 하지만 우분투를 새로 막 설치한 경우에는 패키지 목록이 없을 것이고, 패키지 목록이 있다고 하더라도 시간이 지남에 따라 패키지들의 버전은 달라질 수 있다.

이를 해결해주는 것이 `apt-get update` 명령어다. 이 명령어를 입력하면 `apt-get`은 /etc/apt/sources.list 파일에 기록된 패키지 저장소의 주소를 사용한다. 이 파일의 첫 몇 줄은 다음과 같다.
```text
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://archive.ubuntu.com/ubuntu/ focal main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates main restricted
```
주소와 일부 내용은 버전 및 설정 등에 따라 상이할 수 있다. 위 내용에 따르면 http://archive.ubuntu.com/ubuntu/ 라는 저장소를 이용한다는 사실을 알 수 있다.

> ## 패키지 저장소 미러 사이트
> 국내에서 운영하는 패키지 저장소 미러 사이트로는 다음과 같다.
> - http://kr.archive.ubuntu.com/, https://ftp.kaist.ac.kr/ (카이스트)
> - http://mirror.kakao.com/ (카카오, 추천)
> - http://ftp.harukasan.org/ (부산 부경대)
> - http://mirror.navercorp.com/ (네이버, 우분투는 없음)
> - http://ftp.neowiz.com/ (네오위즈, 2020년 4월 운영 종료)

> ## 우분투 저장소와 4개의 카테고리
> 우분투 저장소는 4개의 카테고리로 나뉜다. `main`, `universe`, `restricted`, `multiverse`가 바로 그것이다.
> ### main
> 우분투에서 공식적으로 지원하며, 자유롭게 사용할 수 있는 소프트웨어.
> ### restricted
> 우분투에서 공식적으로 지원하지만, 무료가 아닐 수도 있고 오픈소스도 아닐 수 있는 소프트웨어.
> ### universe
> 우분투에서 공식적으로 지원하지 않아 보안 업데이트가 보장되지 않는 오픈소스 소프트웨어.
> ### multiverse
> 우분투에서 공식적으로 지원하지도 않고, 오픈소스도 아닐 수 있는 소프트웨어.

/etc/apt/sources.list 파일에 기록된 패키지 저장소로부터 패키지 목록을 받아오고 이를 /var/lib/apt/lists 디렉터리에 저장한다. 이 디렉터리를 일종의 캐시 개념으로 쓰는 것이다. `apt-get update`를 하고 난 후 /var/lib/apt/lists 디렉터리에는 다음과 같은 파일들이 생길 것이다.
```bash
$ ll /var/lib/apt/lists
total 29112
drwxr-xr-x 4 root root     4096 Aug 14 07:21 ./
drwxr-xr-x 1 root root     4096 Aug 14 06:28 ../
-rw-r--r-- 1 root root   100555 Aug 14 06:29 archive.ubuntu.com_ubuntu_dists_focal-backports_InRelease
-rw-r--r-- 1 root root     4086 Jul  2 15:52 archive.ubuntu.com_ubuntu_dists_focal-backports_main_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root     9901 Jul 27 08:49 archive.ubuntu.com_ubuntu_dists_focal-backports_universe_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root   113785 Aug 14 06:29 archive.ubuntu.com_ubuntu_dists_focal-updates_InRelease
-rw-r--r-- 1 root root  2357292 Aug 14 01:19 archive.ubuntu.com_ubuntu_dists_focal-updates_main_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root    58797 Aug  5 01:47 archive.ubuntu.com_ubuntu_dists_focal-updates_multiverse_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root   802055 Aug 12 20:58 archive.ubuntu.com_ubuntu_dists_focal-updates_restricted_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root  1741311 Aug 14 01:02 archive.ubuntu.com_ubuntu_dists_focal-updates_universe_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root   264892 Apr 23  2020 archive.ubuntu.com_ubuntu_dists_focal_InRelease
-rw-r--r-- 1 root root  2048820 Apr 23  2020 archive.ubuntu.com_ubuntu_dists_focal_main_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root   276851 Apr 22  2020 archive.ubuntu.com_ubuntu_dists_focal_multiverse_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root    47909 Apr 23  2020 archive.ubuntu.com_ubuntu_dists_focal_restricted_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root 18119541 Apr 23  2020 archive.ubuntu.com_ubuntu_dists_focal_universe_binary-amd64_Packages.lz4
drwxr-xr-x 2 _apt root     4096 Aug 14 06:28 auxfiles/
-rw-r----- 1 root root        0 Aug 14 06:28 lock
drwx------ 2 _apt root     4096 Aug 14 07:21 partial/
-rw-r--r-- 1 root root   113787 Aug 14 06:29 security.ubuntu.com_ubuntu_dists_focal-security_InRelease
-rw-r--r-- 1 root root  1633539 Aug 12 17:18 security.ubuntu.com_ubuntu_dists_focal-security_main_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root    45967 Jul 26 18:25 security.ubuntu.com_ubuntu_dists_focal-security_multiverse_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root   712028 Aug  4 13:02 security.ubuntu.com_ubuntu_dists_focal-security_restricted_binary-amd64_Packages.lz4
-rw-r--r-- 1 root root  1309567 Aug 12 21:52 security.ubuntu.com_ubuntu_dists_focal-security_universe_binary-amd64_Packages.lz4
```
여기서 중요한 건 `*.lz4` 파일들인데, 실제로 패키지의 맵핑 정보를 확인하는 곳이 lz4 압축 파일이기 때문이다(실제로 lz4 파일을 삭제하고 install 하려고 하면 오류가 난다). nodejs는 universe 패키지에서 관리되므로 **archive.ubuntu.com_ubuntu_dists_focal_universe_binary-amd64_Packages.lz4**를 압축 해제하여 확인해 보자.
```bash
$ lz4 -d archive.ubuntu.com_ubuntu_dists_focal_universe_binary-amd64_Packages.lz4
Decoding file archive.ubuntu.com_ubuntu_dists_focal_universe_binary-amd64_Packages 
archive.ubuntu.com_u : decoded 50750067 bytes

$ vim archive.ubuntu.com_ubuntu_dists_focal_main_binary-amd64_Packages.lz4
...
Package: nodejs
Architecture: amd64
Version: 10.19.0~dfsg-3ubuntu1
Multi-Arch: foreign
Priority: extra
Section: universe/web
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian Javascript Maintainers <pkg-javascript-devel@lists.alioth.debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 154
Depends: libc6 (>= 2.4), libnode64 (= 10.19.0~dfsg-3ubuntu1)
Recommends: ca-certificates, nodejs-doc
Suggests: npm
Conflicts: nodejs-legacy
Replaces: nodejs-legacy
Filename: pool/universe/n/nodejs/nodejs_10.19.0~dfsg-3ubuntu1_amd64.deb
Size: 61128
MD5sum: e99245e2f78b02e736f6585be7a200f2
SHA1: 8a7f18fc393650d3f0830c6cde0e0bfa7e4afa17
SHA256: 392d4ae36ca956260df71b43f7cd04c1f8928dac5e53f4132d237772b16adf4a
Homepage: http://nodejs.org/
Description: evented I/O for V8 javascript - runtime executable
Description-md5: 0d0bbaed314d7d26588d112ee4ede074
...
```
여러 패키지들의 정보를 포함하고 있는 것을 확인할 수 있다. 패키지 이름, 아키텍처, 버전 등의 정보 뿐만 아니라 의존성이 걸려있는 패키지 정보, 패키지에 해당하는 파일 이름, 홈페이지, 기타 해시값 등등. 여기서 실제로 패키지를 다운받는 주소는 **저장소의 주소**와 **패키지 파일 이름**을 조합해서 만들어진다.

위에서 확인한 저장소의 주소는 `http://archive.ubuntu.com/ubuntu/`였고, 방금 확인한 Filename은 `pool/universe/n/nodejs/nodejs_10.19.0~dfsg-3ubuntu1_amd64.deb`이므로 이를 조합한 http://archive.ubuntu.com/ubuntu/pool/universe/n/nodejs/nodejs_10.19.0~dfsg-3ubuntu1_amd64.deb 이 다운받는 실제 주소인 것이다. 만약 사용자가 다운받는 실제 주소를 확인하고 싶다면 다음과 같은 명령어로 확인할 수 있다.
```bash
$ apt-get download --print-uris nodejs
'http://archive.ubuntu.com/ubuntu/pool/universe/n/nodejs/nodejs_10.19.0~dfsg-3ubuntu1_amd64.deb' nodejs_10.19.0~dfsg-3ubuntu1_amd64.deb 61128 SHA256:392d4ae36ca956260df71b43f7cd04c1f8928dac5e53f4132d237772b16adf4a
```
저장소의 주소와 패키지 파일 이름을 조합해서 구한 실제 주소와 동일하게 잘 나온다.

이제 update를 완료하여 /var/lib/apt/lists 디렉터리에 패키지 목록도 정상적으로 받아온 상태이므로 아까 실패했던 `apt-get install` 명령어를 입력해 보자.
```bash
$ apt-get install nodejs
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  ca-certificates libc-ares2 libicu66 libnghttp2-14 libnode64 libuv1 nodejs-doc openssl tzdata
Suggested packages:
  npm
The following NEW packages will be installed:
  ca-certificates libc-ares2 libicu66 libnghttp2-14 libnode64 libuv1 nodejs nodejs-doc openssl tzdata
0 upgraded, 10 newly installed, 0 to remove and 3 not upgraded.
Need to get 16.5 MB of archives.
After this operation, 70.4 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```
패키지를 제대로 읽어들였고 설치할지 말지를 묻고 있다. 설치하고 싶은 경우 Y를, 그렇지 않으면 n을 입력하면 된다.

# 마치며...
지금까지 apt를 이용하여 패키지 설치를 많이 했는데, 정작 동작 원리에 대해서는 전혀 생각을 해본 적이 없었다. `apt-get install some-package`을 했다가 안 되면 기계적으로 `apt-get update`를 입력하고 그 후에 다시 install해왔다. 왜 update를 먼저 해야 하는지에 대해서는 전혀 궁금해하지 않았던 것이다.

그러다가 최근에 와서야 문득 원리가 궁금해졌고, 열심히 찾아본 결과 내부적으로 동작하는 방식을 알아내어 이를 블로그에 정리한다. 이미 알고 있는 사람에게는 이러한 흐름이 당연한 것일 수도 있지만 나처럼 몰랐던 사람들은 이번 기회에 원리를 한 번 짚고 넘어가는 것도 괜찮을 것이다.

# References
- https://en.wikipedia.org/wiki/APT_(software)
- https://itsfoss.com/apt-vs-apt-get-difference/
- https://titanwolf.org/Network/Articles/Article?AID=c87b0ea5-add1-42d5-9711-c1956b772206
- https://www.quora.com/How-exactly-does-the-sudo-apt-get-install-package-work-in-Linux
- https://help.ubuntu.com/community/Repositories

<br>
<br>

---

<a id="fn-1">[\[1\]](#rfn-1)</a>: `dpkg`는 1994년 1월, `APT`는 1998년 3월에 릴리즈됐다. [↩](#rfn-1)
