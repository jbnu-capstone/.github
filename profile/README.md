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

**사용 이유**: 여러 악기가 섞여 있는 공연 상황에서 악기 별로 연주를 추적해서 페이지를 넘길 필요가 있었기 때문이다. 

<img width="2464" height="610" alt="image" src="https://github.com/user-attachments/assets/9ccd6ad3-3213-4dca-87ec-566cd969bfea" />

**모델 파이프라인**:

  - 입력 쪼개기: 
  혼합 파형을 짧은 구간 L로 나눈다. (슬라이딩 윈도우)

  - 인코딩: 
  각 구간을 1D 컨볼루션 인코더로 변환해 특징 벡터로 만든다. STFT 없음. 시간 도메인 그대로 처리.

  - 분리(마스크 추정): 
  특징을 Separation 모듈에 넣어 소스별 마스크를 예측한다. 예: 피아노, 잡음.
  
  - 마스킹 적용: 
  인코더 출력 × 마스크 = 소스별 인코더 표현. (그림의 ⊗)
  
  - 디코딩: 
  소스별 인코더 표현을 디코더로 통과시켜 다시 파형으로 복원한다.
  
  - 구간 결합: 
  복원된 짧은 구간들을 겹침-추가(OLA)로 이어 붙여 연속 파형을 만든다.
  
  - 결과: 
  각 소스에 대한 분리 파형이 나온다. 필요하면 목표 소스만 사용한다.

---


- **한 트랙 안에 섞인 여러 소리를 시간 도메인에서 직접 분리하는 딥러닝 모델**
- **실제 연주 환경의 잡음·다른 악기 영향을 줄이려 전처리 단계에 Conv-TasNet을 둠**
- **혼합 연주에서 피아노 트랙만 분리하여 기존 Multi-Resolution Prediction 기반 악보 추적 모델의 입력으로 사용**

  **성과: 분리한 피아노 신호를 쓰면 실시간 추적이 더 안정적으로 변함**

  ---

### 2. WebSocket을 이용한 실시간 좌표 예측 서버 구축

<img width="493" height="400" alt="image" src="https://github.com/user-attachments/assets/7affbba4-3efb-4cd6-bc6c-33e9067bf50e" />

- **클라이언트가 연주음(‘Performance’)을 0.5초 간격으로 전송**
- **서버가 예측된 좌표(‘Coords’)를 클라이언트에 전송**
- **클라이언트가 악보(‘Score’)를 전송**

---

### 3. Yolo를 이용한 Multi-resolution Prediction 모델

[출처](https://github.com/CPJKU/cyolo_score_following)

![yolo](https://github.com/user-attachments/assets/0a61098f-d261-4617-9eba-c8b7fb0aab48)

- **오디오**: waveform → STFT/로그멜 → CNN → LSTM → 조건 벡터 z

- **이미지**: 악보 이미지 → Downscale CNN → FiLM으로 z 주입 → Upscale/합성

- **출력**: YOLO식 Detection Heads가 노트·마디·시스템 박스를 예측

---

#### 3.1 해당 모델을 사용한 이유

1. **사용자 편의성**: 기존 앱은 사전 작업이 완료된 악보만 악보 추적 기능을 제공하였다. 이를 해결하기 위해 사용자가 악보를 업로드하고 해당하는 악보를 인식하는 기술이 필요하였다.

2. **실시간 추적에 맞는 속도·간단한 파이프라인·박스 기반 출력**:

   - 작은 노트, 큰 마디·시스템을 동시에 봐야 한다. YOLO 계열의 멀티스케일 헤드로 한 번에 예측 가능하다.
   
   - 오디오 임베딩(z)을 FiLM 등으로 중간 피처에 주입하기 쉽다. 구조가 모듈식이라 결합·실험이 빠르다.
   
3. **데이터 효율**: 적은 악보 데이터로도 수렴이 빠르고 학습 및 튜닝 비용이 낮다.

---

### 4. 화면 흐름도

**악보 추적 흐름도**
<img width="1091" height="1359" alt="image" src="https://github.com/user-attachments/assets/e8a77a4a-315d-4314-add7-4ba9025c9c11" />

**악보 필기 흐름도**
<img width="1026" height="1380" alt="image" src="https://github.com/user-attachments/assets/787c5de6-ad22-4dae-a5c5-5d0192165b98" />



### 5. 화면 설계

**시작 화면**
<img width="1215" height="612" alt="image" src="https://github.com/user-attachments/assets/95a9d514-2e8d-4c9e-8b0d-a0ae73bd2344" />

**로그인 화면**
<img width="1215" height="612" alt="image" src="https://github.com/user-attachments/assets/edd51d55-4adc-4a97-8d10-c203b8d707f0" />

**악보 선택 화면**
<img width="1327" height="612" alt="image" src="https://github.com/user-attachments/assets/ed7ddeaa-41e3-4d4c-b5ac-8d0ea3d20a34" />

**악보 편집 화면**
<img width="1328" height="612" alt="image" src="https://github.com/user-attachments/assets/41857a9c-1c4e-4d69-8f8f-410da0a61927" />

**악보 필기 화면**
<img width="1327" height="612" alt="image" src="https://github.com/user-attachments/assets/dfe3c34f-df1c-4d91-9c89-e83f0fb6460e" />


## 출판한 논문
[오디오 소스 분리 기술을 통한 악보 추적 성능 개선 방안 (1).pdf](https://github.com/user-attachments/files/23414077/1.pdf)



