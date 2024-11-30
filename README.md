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
