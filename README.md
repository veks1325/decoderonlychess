# ♟️ Transformer Chess Engine 

이 프로젝트는 GPT와 같은 **Decoder-only Transformer** 아키텍처를 기반으로 작동하는 딥러닝 체스 엔진입니다. 
체스 기보를 하나의 언어 시퀀스로 취급하여, 주어진 국면(지금까지의 수)에서 가장 최적의 '다음 수(Next Move)'를 예측하도록 학습됩니다.

##  구현 순서 및 파이프라인

### 1. 데이터 수집 및 전처리 (Data Preprocessing)
- **PGN 파싱:** Lichess 등의 오픈 데이터베이스에서 레이팅2000+ 체스 기보(PGN)를 가져와 `python-chess` 라이브러리로 읽습니다.
- **UCI 포맷 변환:** 기물의 움직임을 모델이 이해하기 쉬운 텍스트 형태의 UCI 문자열(예: `e2e4`, `g1f3`)로 추출합니다.
- **어휘 사전(Vocabulary):** 가능한 모든 체스 수(약 2,000개)와 특수 토큰(`<BOS>`, `<EOS>`, `<PAD>`)을 정수 ID에 매핑하는 단어장을 만듭니다.

### 2. 데이터셋 및 로더 구축 (Dataset & DataLoader)
- **전체 시퀀스 학습:** 일반 텍스트 데이터처럼 기보를 중간에서 무작위로 자르지 않습니다. 체스의 물리적 룰에 어긋나는 불법적인 수(Illegal state)가 발생하는 것을 막기 위해, 무조건 게임의 첫 수(`<BOS>`)부터 끝까지를 하나의 온전한 시퀀스로 묶어 학습합니다.
- **패딩(Padding):** 게임마다 종료되는 턴 수가 다르므로 모델의 `max_seq_len` (예: 256)에 맞춰 `<PAD>` 토큰을 채워 텐서 길이를 맞춥니다.
- **Shifted 시퀀스:** 모델 입력용(`x`)과 정답용(`y`) 시퀀스를 한 칸씩 어긋나게 구성하여 다음 수 예측(Next-token prediction)을 위한 미니배치를 생성합니다.

### 3. 트랜스포머 모델 아키텍처 구현 (Model Architecture)
- **디코더 온리(Decoder-only) 구조:** 과거의 수만 보고 미래의 수를 예측하기 위해 **마스크드 셀프 어텐션(Masked Self-Attention)**을 적용합니다.
- **임베딩(Embedding):** 토큰화된 수의 ID를 고차원 임베딩 벡터로 변환하고, 위치 인코딩(Positional Encoding)을 더해 수의 순서 및 턴(Turn) 정보를 모델에 전달합니다.
- **포지셔널 임베딩 rope 나 절대위치로 해서 성능비교로 선택하기**

### 4. 모델 훈련 (Training)
- **프리트레이닝 (Autoregressive Training):** 교차 엔트로피 손실(Cross-Entropy Loss) 함수를 사용하여, 모델이 매 턴마다 정답의 확률을 최대화하도록 가중치를 업데이트합니다.
- 이 과정은 대규모 기보 패턴을 모방하는 행동 복제(Behavioral Cloning) 형태의 프리트레이닝으로 진행됩니다.

### 5. 추론 및 UCI 엔진 연동 (Inference & UCI Integration)
- **불법적인 수 필터링(Legal Move Filtering):** 추론 시 모델이 체스 규칙에 어긋나는 수를 두는 것(할루시네이션)을 막기 위해, `python-chess`를 활용해 현재 보드 상태의 '합법적인 수' 목록을 뽑아냅니다. 이후 모델이 예측한 확률 분포에서 불법적인 수의 확률을 0으로 마스킹합니다.
- **UCI 프로토콜 연동:** Arena, Lucas Chess 같은 체스 GUI 프로그램과 엔진이 통신할 수 있도록 UCI(Universal Chess Interface) 규격에 맞춰 파이썬 스크립트를 작성하고 연동합니다.

---

##  기술 스택
- **Language:** Python 3.x
- **Deep Learning Framework:** PyTorch
- **Chess Library:** `python-chess`

## 💡 향후 과제 (TODO)
- [ ] 데이터 전처리 스크립트 작성 (PGN to Tokens)
- [ ] PyTorch Dataset / DataLoader 모듈화 완료
- [ ] Transformer Decoder 모델 클래스 설계 및 구현
- [ ] 학습 루프(Training Loop) 작성 및 대규모 데이터로 모델 학습
- [ ] 불법 수 마스킹이 포함된 추론(Generation) 함수 구현
- [ ] UCI 연동 인터페이스 스크립트 작성 및 GUI 테스트
