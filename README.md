# jetson_dli_1234
![image](https://github.com/user-attachments/assets/f2953de3-1d15-478f-9387-acbe3d2c9aef)

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

### 3. Camera (차례로 입력)
![image](https://github.com/user-attachments/assets/aee59202-22bb-4f04-8729-7405ab7892f8)

![image](https://github.com/user-attachments/assets/431e3730-287c-4606-9568-5fc1947c7f7b)


dli@dli-desktop:~$  ls /dev/vi*

dli@dli-desktop:~$ git clone https://github.com/jetsonhacks/USB-Camera.git

dli@dli-desktop:~$ cd USB-Camera

dli@dli-desktop:~/USB-Camera$ python3 usb-camera-gst.py

dli@dli-desktop:~/USB-Camera$  python3 face-detect-usb.py

### 4. 헤드리스 모드 (차례로 입력)

![image](https://github.com/user-attachments/assets/33acc73d-5adc-402e-b7fd-bdeb429bd5cd)


dli@dli-desktop:~$ ls

dli@dli-desktop:~$ mkdir -p ~/nvdli-data

dli@dli-desktop:~$#!/bin/bash
 sudo docker run --runtime nvidia -it --rm --network host \
    --memory=500M --memory-swap=4G \
    --volume ~/nvdli-data:/nvdli-nano/data \
    --volume /tmp/argus_socket:/tmp/argus_socket \
    --device /dev/video0 \
    nvcr.io/nvidia/dli/dli-nano-ai:v2.0.2-r32.7.1k

dli@dli-desktop:~$ chmod +x docker_dli_run.sh

dli@dli-desktop:~$ ./docker_dli_run.sh

sudo systemctl disable nvzramconfig

sudo systemctl set-default multi-user.target

sudo fallocate -l 18G /mnt/18GB.swap

sudo chmod 600 /mnt/18GB.swap

sudo mkswap /mnt/18GB.swap

sudo su

echo "/mnt/18GB.swap swap swap defaults 0 0" >> /etc/fstab

exit

sudo reboot

reboot 한 후, MICRO 5 PIN 연결 -> L4T 연결 된 표시가 오른쪽 하단에 나온다.

sudo systemctl set-default graphical.target (시스템을 GUI 모드로 설정)

### 5. Image Classification Project

![image](https://github.com/user-attachments/assets/4b94568c-c9d1-4893-a2f5-5262daadf44a)


#### **1단계: 노트북 열기**  
- JupyterLab에서 `classification` 폴더로 이동한 후, `classification_interactive.ipynb` 파일 열기.  

#### **2단계: 코드 실행**  
- 노트북의 모든 셀을 순차적으로 실행하며 환경 초기화 진행.  
- 카메라 활성화 시 USB 카메라가 올바르게 선택되었는지 확인.

#### **3단계: 데이터 수집**  
1. `data_collection_widget`를 사용해 데이터 수집 진행.  
   - "thumbs-up" 이미지를 30장 수집.  
   - "thumbs-down" 이미지를 30장 수집.  
   - 각도를 조정하며 다양한 데이터를 확보하도록 설정.  

#### **4단계: 모델 훈련**  
- 에포크 수를 10으로 설정하고 "Train" 버튼 클릭하여 훈련 시작.  
- 진행 상황에서 손실(Loss)과 정확도를 확인하며 모델 개선 확인.

#### **5단계: 실시간 테스트**  
- 라이브 카메라 피드를 통해 테스트 실행.  
- 예측 결과와 확률을 슬라이더로 확인하여 성능 평가 진행.

#### **6단계: 모델 개선**  
1. 다른 배경과 각도로 "thumbs-up" 및 "thumbs-down" 데이터를 각각 30장 추가 수집.  
2. 5 에포크만큼 추가 훈련 진행.  
3. 다양한 거리와 위치에서 모델 성능 확인 및 테스트 반복.  

#### **7단계: 모델 저장**  
- 모델 이름 입력 후 "save model" 클릭하여 모델 저장 완료.

---

위 단계를 반복하며 모델 성능 개선 작업 지속.

### 6. 젯슨나노-아두이노, 미세먼지 측정 코드 (오류 날 경우, 한글을 영어로 바꿔서 진행)

---

int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000;  // 30초 동안 샘플링
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float concentration = 0;

void setup()
{
    Serial.begin(9600);
    pinMode(pin, INPUT);
    starttime = millis();
    Serial.println("미세먼지 측정을 시작합니다...");
    Serial.println("==============================");
}

void loop()
{
    duration = pulseIn(pin, LOW);
    lowpulseoccupancy = lowpulseoccupancy + duration;

    if ((millis()-starttime) > sampletime_ms)  // 30초마다 측정
    {
        ratio = lowpulseoccupancy/(sampletime_ms*10.0);
        concentration = 1.1*pow(ratio,3)-3.8*pow(ratio,2)+520*ratio+0.62; // ug/m3 단위

        Serial.println("==============================");
        Serial.print("미세먼지 농도: ");
        Serial.print(concentration);
        Serial.println(" ug/m3");

        // 대기질 상태 표시
        Serial.print("대기질 상태: ");
        if(concentration <= 30) {
            Serial.println("좋음");
        }
        else if(concentration <= 80) {
            Serial.println("보통");
        }
        else if(concentration <= 150) {
            Serial.println("나쁨");
        }
        else {
            Serial.println("매우 나쁨");
        }

        Serial.println("------------------------------");
        Serial.print("측정 시간: ");
        Serial.print(millis()/1000);
        Serial.println("초");
        Serial.println("==============================\n");

        // 다음 측정을 위한 초기화
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
---
















# NVIDIA Jetson Nano 환경 구축 및 프로젝트 안내

![NVIDIA Jetson Nano](https://github.com/user-attachments/assets/f2953de3-1d15-478f-9387-acbe3d2c9aef)

## 목차

1. [NVIDIA Jetson Nano 환경 구축](#1-nvidia-jetson-nano-환경-구축)
2. [초기 설정 및 첫 부팅](#2-초기-설정-및-첫-부팅)
3. [카메라 설정](#3-카메라-설정)
4. [헤드리스 모드 설정](#4-헤드리스-모드-설정)
5. [이미지 분류 프로젝트](#5-이미지-분류-프로젝트)
6. [Jetson Nano와 Arduino를 이용한 미세먼지 측정](#6-jetson-nano와-arduino를-이용한-미세먼지-측정)

---

## 1. NVIDIA Jetson Nano 환경 구축

### 1-1. 소개

NVIDIA Jetson Nano는 AI 및 딥러닝 애플리케이션을 위한 강력한 개발 보드입니다. 이 가이드에서는 Jetson Nano를 설정하고 다양한 프로젝트를 수행하는 방법을 안내합니다.

### 1-2. JetPack 4.6 다운로드

[NVIDIA 공식 사이트](https://developer.nvidia.com/embedded/jetpack)를 방문하여 **JetPack 4.6** 이미지를 다운로드합니다. 이 이미지는 Jetson Nano에서 필요한 모든 드라이버와 라이브러리를 포함하고 있습니다.

### 1-3. SD 메모리 카드 포맷터 다운로드

[SD Association의 SD 메모리 카드 포맷터](https://www.sdcard.org/downloads/formatter/)를 다운로드하여 SD 카드를 포맷합니다.

### 1-4. Balena Etcher 다운로드

[Balena Etcher](https://www.balena.io/etcher/)를 다운로드하여 JetPack 이미지를 SD 카드에 굽습니다.

### 1-5. SD 카드에 JetPack 4.6 이미지 굽기

Balena Etcher를 사용하여 다운로드한 JetPack 4.6 이미지를 SD 카드에 굽습니다. 완료되면 SD 카드를 Jetson Nano에 삽입합니다.

---

## 2. 초기 설정 및 첫 부팅

[NVIDIA Jetson Nano 시작하기 가이드](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#setup)를 참고하여 Jetson Nano를 초기 설정합니다.

### 2-1. 한글 입력기 설치

터미널을 열고 다음 명령을 실행하여 `fcitx-hangul`을 설치합니다:

```bash
sudo apt-get update
sudo apt-get install fcitx-hangul
```

### 2-2. 언어 및 입력기 설정

1. **Ubuntu 설정**에서 **Region & Language**를 선택합니다.
2. **Manage Installed Languages** 버튼을 클릭하여 필요한 언어 패키지를 설치합니다.
3. 설치가 완료되면 **Keyboard input method system**을 `fcitx`로 변경합니다.
4. 시스템을 재부팅합니다.
5. 재부팅 후, **Input Method Configuration**에서 **Korean - Hangul**을 추가합니다.

---

## 3. 카메라 설정

Jetson Nano에서 USB 카메라를 사용하기 위해 다음 단계를 수행합니다.

### 3-1. 카메라 장치 확인

터미널에서 다음 명령을 실행하여 비디오 장치를 확인합니다:

```bash
ls /dev/video*
```

### 3-2. USB 카메라 예제 코드 클론

```bash
git clone https://github.com/jetsonhacks/USB-Camera.git
cd USB-Camera
```

### 3-3. 카메라 실행 테스트

```bash
python3 usb-camera-gst.py
```

### 3-4. 얼굴 인식 테스트

```bash
python3 face-detect-usb.py
```

---

## 4. 헤드리스 모드 설정

헤드리스 모드로 Jetson Nano를 설정하여 모니터 없이 원격으로 접근할 수 있습니다.

### 4-1. 작업 디렉토리 생성

```bash
mkdir -p ~/nvdli-data
```

### 4-2. Docker 실행 스크립트 작성

`docker_dli_run.sh` 파일을 생성하고 다음 내용을 추가합니다:

```bash
#!/bin/bash
sudo docker run --runtime nvidia -it --rm --network host \
    --memory=500M --memory-swap=4G \
    --volume ~/nvdli-data:/nvdli-nano/data \
    --volume /tmp/argus_socket:/tmp/argus_socket \
    --device /dev/video0 \
    nvcr.io/nvidia/dli/dli-nano-ai:v2.0.2-r32.7.1
```

### 4-3. 스크립트에 실행 권한 부여 및 실행

```bash
chmod +x docker_dli_run.sh
./docker_dli_run.sh
```

### 4-4. ZRAM 비활성화 및 스왑 메모리 설정

```bash
sudo systemctl disable nvzramconfig
sudo systemctl set-default multi-user.target
sudo fallocate -l 18G /mnt/18GB.swap
sudo chmod 600 /mnt/18GB.swap
sudo mkswap /mnt/18GB.swap
```

`/etc/fstab` 파일에 다음 내용을 추가합니다:

```bash
/mnt/18GB.swap swap swap defaults 0 0
```

### 4-5. 시스템 재부팅 및 GUI 모드 설정

```bash
sudo reboot
```

재부팅 후, 다음 명령으로 GUI 모드를 활성화합니다:

```bash
sudo systemctl set-default graphical.target
```

---

## 5. 이미지 분류 프로젝트

이미지 분류 프로젝트를 수행하여 딥러닝 모델을 학습하고 테스트합니다.

### 5-1. 노트북 열기

- JupyterLab에서 `classification` 폴더로 이동하여 `classification_interactive.ipynb` 파일을 엽니다.

### 5-2. 코드 실행

- 노트북의 모든 셀을 순차적으로 실행하여 환경을 초기화합니다.
- 카메라 활성화 시 USB 카메라가 올바르게 선택되었는지 확인합니다.

### 5-3. 데이터 수집

1. `data_collection_widget`를 사용하여 데이터 수집을 시작합니다.
   - "thumbs-up" 이미지를 30장 수집합니다.
   - "thumbs-down" 이미지를 30장 수집합니다.
   - 다양한 각도와 배경에서 이미지를 촬영합니다.

### 5-4. 모델 훈련

- 에포크 수를 10으로 설정하고 "Train" 버튼을 클릭하여 모델 훈련을 시작합니다.
- 훈련 과정에서 손실(Loss)과 정확도(Accuracy)를 확인합니다.

### 5-5. 실시간 테스트

- 라이브 카메라 피드를 통해 모델을 테스트합니다.
- 예측 결과와 확률을 확인하여 모델의 성능을 평가합니다.

### 5-6. 모델 개선

1. 추가 데이터 수집:
   - 다른 배경과 각도로 "thumbs-up" 및 "thumbs-down" 이미지를 각각 30장씩 추가로 수집합니다.
2. 추가 훈련:
   - 에포크 수를 5로 설정하여 추가 훈련을 진행합니다.
3. 성능 확인:
   - 다양한 거리와 위치에서 모델의 성능을 확인합니다.

### 5-7. 모델 저장

- 모델 이름을 입력한 후 "Save Model" 버튼을 클릭하여 모델을 저장합니다.

---

## 6. Jetson Nano와 Arduino를 이용한 미세먼지 측정

Jetson Nano와 Arduino를 사용하여 미세먼지 센서를 제어하고 데이터를 수집합니다.

### 6-1. Arduino 코드 작성

아래 코드를 Arduino IDE에 복사하여 사용합니다. (오류 발생 시 한글 주석을 영어로 변경하세요.)

```cpp
int pin = 8;
unsigned long duration;
unsigned long starttime;
unsigned long sampletime_ms = 30000;  // 30초 동안 샘플링
unsigned long lowpulseoccupancy = 0;
float ratio = 0;
float concentration = 0;

void setup()
{
    Serial.begin(9600);
    pinMode(pin, INPUT);
    starttime = millis();
    Serial.println("미세먼지 측정을 시작합니다...");
    Serial.println("==============================");
}

void loop()
{
    duration = pulseIn(pin, LOW);
    lowpulseoccupancy += duration;

    if ((millis() - starttime) > sampletime_ms)  // 30초마다 측정
    {
        ratio = lowpulseoccupancy / (sampletime_ms * 10.0);
        concentration = 1.1 * pow(ratio, 3) - 3.8 * pow(ratio, 2) + 520 * ratio + 0.62; // 단위: ug/m3

        Serial.println("==============================");
        Serial.print("미세먼지 농도: ");
        Serial.print(concentration);
        Serial.println(" ug/m3");

        // 대기질 상태 표시
        Serial.print("대기질 상태: ");
        if (concentration <= 30) {
            Serial.println("좋음");
        }
        else if (concentration <= 80) {
            Serial.println("보통");
        }
        else if (concentration <= 150) {
            Serial.println("나쁨");
        }
        else {
            Serial.println("매우 나쁨");
        }

        Serial.println("------------------------------");
        Serial.print("측정 시간: ");
        Serial.print(millis() / 1000);
        Serial.println("초");
        Serial.println("==============================\n");

        // 다음 측정을 위한 초기화
        lowpulseoccupancy = 0;
        starttime = millis();
    }
}
```

### 6-2. 설명

- **핀 설정**: 미세먼지 센서의 신호 핀을 Arduino의 8번 핀에 연결합니다.
- **샘플링 시간**: 30초 동안 데이터를 수집하여 평균 농도를 계산합니다.
- **농도 계산**: 수집된 데이터를 바탕으로 미세먼지 농도를 계산합니다.
- **대기질 상태**: 계산된 농도에 따라 대기질 상태를 출력합니다.

### 6-3. 주의 사항

- 코드에서 한글 주석이나 문자열로 인해 오류가 발생할 경우, 한글을 영어로 변경하여 재시도합니다.
- Arduino와 Jetson Nano 간의 시리얼 통신 설정에 유의합니다.

---

## 추가 참고 자료

- [NVIDIA Jetson Nano Developer Kit 사용자 매뉴얼](https://developer.nvidia.com/embedded/downloads)
- [JetsonHacks GitHub 리포지토리](https://github.com/jetsonhacks)
- [Balena Etcher 공식 웹사이트](https://www.balena.io/etcher/)
- [SD 카드 포맷터 다운로드](https://www.sdcard.org/downloads/formatter/)

---

이 가이드를 따라 NVIDIA Jetson Nano의 환경을 구축하고 다양한 프로젝트를 수행해보세요. 궁금한 사항이나 문제가 발생하면 위의 참고 자료를 확인하거나 관련 커뮤니티에 질문하시기 바랍니다.
