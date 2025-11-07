# 연주자를 위한 자동 페이지 터너 A.P.T(Auto Page Turner)

## 시스템 아키텍쳐

<img width="2026" height="847" alt="image" src="https://github.com/user-attachments/assets/aca7141f-1a62-4d8c-bd09-797367d5f13f" />

## 프로젝트 기획

이 프로젝트는 연주 중 손을 쓰지 않아도 악보가 자동으로 넘어가게 해, 페이지 넘김 실수와 몰입 방해를 없애는 자동 페이지 터너 시스템입니다.

기존 블루투스 페이지터너의 추가 장비·학습 부담을 줄이고, 연주 몰입도와 안정성을 높이는 것이 프로젝트의 핵심 가치입니다.

## Business Process Model

<img width="1868" height="1425" alt="image" src="https://github.com/user-attachments/assets/c65c2eb9-b916-45c4-a718-5a508a6dfe6f" />

- **로그인**: 사용자가 로그인 요청 → APT가 로그인 처리.

- **악보 준비**: 업로드 요청 → APT가 악보 전처리. 불러오기 요청 → APT가 데이터 처리.

- **연주 중 동작**: APT가 악보를 출력하고, 추적 여부에 따라 분기. 추적 O면 추적 모듈로 보내고 결과를 다시 악보 출력으로 피드백해 페이지를 자동 전환. 추적 X면 그대로 표시.

- **관리 기능**: 사용자의 악보 관리 요청 → APT가 관리 처리.

- **추적만 별도 요청 가능**: “악보 추적 요청” 시 APT가 연주음 전처리를 수행해 추적 품질을 높임.

- **수정·삭제**: 요청 → APT가 반영하고 종료.


## System Architecture

<img width="2291" height="1275" alt="image" src="https://github.com/user-attachments/assets/9641c601-991d-4627-8d3d-5c72b34175dd" />

**구성요소**

- **APP**

  - Uploader: 사용자 악보(PDF/PNG) 업로드. 서버로 전송.

  - Microphone: 연주 오디오 캡처. 스트림 또는 청크로 서버 전송.

  - ScoreViewer: 악보 표시. 서버와 DB에서 악보 불러오고, 예측 좌표를 받아 뷰를 이동·넘김. 주석(드로잉)도 저장 요청.

  - LoginManager: 로그인 요청 처리. 사용자 정보 조회.

- **Server**

  - Prediction Manager: 오디오 전처리 → Prediction Model 호출 → 예측 좌표 반환. 결과를 웹소켓을 통해 좌표를 클라이언트 측에 전달.

  - Prediction Model: 전처리된 악보·오디오를 입력으로 현재 위치 좌표를 예측.

- **저장소**

  - External File System(DB): 사용자·악보 메타데이터 저장.

  - Local File System(DB): 악보, 전처리 결과, 필기 데이터 저장.
 

## 시연 연상

![apt_test (1)](https://github.com/user-attachments/assets/154ab0d3-3749-45ed-a8ff-86b1a15bb8de)

## 사용된 기술 스택
<p>
<img src="https://img.shields.io/badge/python-3776AB?style=for-the-badge&logo=python&logoColor=white"> 

<img src="https://img.shields.io/badge/firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=white">

<img src="https://img.shields.io/badge/flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white">

</p>

## 모델 구성 요소

### 1. 오디오 전처리: Conv-TasNet 기반 소스 분리

<img width="1435" height="1023" alt="image" src="https://github.com/user-attachments/assets/340c2e96-bfb7-42db-a623-353806fa4f31" />

- **한 트랙 안에 섞인 여러 소리를 시간 도메인에서 직접 분리하는 딥러닝 모델**
- **실제 연주 환경의 잡음·다른 악기 영향을 줄이려 전처리 단계에 Conv-TasNet을 둠**
- **혼합 연주에서 피아노 트랙만 분리하여 기존 Multi-Resolution Prediction 기반 악보 추적 모델의 입력으로 사용**

  **성과: 분리한 피아노 신호를 쓰면 실시간 추적이 더 안정적으로 변함**



## 출판한 논문
[오디오 소스 분리 기술을 통한 악보 추적 성능 개선 방안 (1).pdf](https://github.com/user-attachments/files/23414077/1.pdf)



