# jetson_dli_1234

### 1. NVIDIA Jetson Nano 환경 구축

1-1. Introduction

1-2. jetack 4.6 다운로드

1-3. SD Memory Card Formatter 다운로드

1-4. balena Etcher 다운로드

1-5. uSD에 Jetpack 4.6 burning (GUI이미지 굽기)

### 2. Setup and First Boot

https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#setup

일단 아래와 같이 수행해서 fcitx-hangul을 설치한다. 

$ sudo apt-get update

$ sudo apt-get install fcitx-hangul
 
우분투 Setting에서 Region & Language 수행 - Manage installed Language 버튼 실행한다.

실행되면 Language 관련 여러가지를 설치한다고 나오는데 설치한다.

설치가 완료되면 하단 keyboard input method system 항목을 fcitx로 변경한다.

시스템 재부팅

Input Method Configuration에서 한글 추가

