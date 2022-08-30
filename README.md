# MusicVAE

## 목표
- Groove MIDI Dataset을 이용한 MusicVAE를 구현하여 4마디에 해당하는 드럼 샘플을 생성

### 참고자료
- 깃헙 코드: https://github.com/magenta/magenta/tree/master/magenta/models/music_vae
- 관련 논문: https://arxiv.org/pdf/1803.05428.pdf
- 관련 데이터: https://magenta.tensorflow.org/datasets/groove

## 프로젝트 진행

1. 환경 설정
- 실행 환경 : 구글 코랩 + 구글 드라이브 마운트
  - 로컬 환경에서는 llvmlite 설치 이슈로 인해 magenta 설치 실패
  - 빠른 프로젝트 진행을 위해 구글 코랩에서 진행
- 깃헙 코드 참고하여 환경 설정하되 전처리 및 학습 코드에서 모듈 실행하는 부분이 있어 magenta 라이브러리만 git clone하여 설치
  
2. 데이터 수집(Groove MIDI Dataset)
- 참고자료 관련 데이터에서 Groove MIDI Dataset 다운로드 url을 확인하고 다운로드

3. 데이터 준비
- 다운로드된 파일이 압축되어 있어서 압축해제

4. 전처리
- 참고코드의 전처리 부분(https://github.com/magenta/magenta/tree/main/magenta/scripts) 참고하여 midi파일을 tfrecord로 변환

5. 모델 학습
- 깃헙 코드 참고하여 훈련 진행
- 훈련 파라미터
  - config : groovae_4bar(프로젝트 목표는 4마디 드럼 샘플 생성이기 때문에 4마디 모델 config를 사용)
  - num_steps : 20000(default는 200000이지만 gpu환경에서도 colab 연결 시간 동안 학습이 불가할 것으로 판단되어 10분의 1로 줄여서 학습 진행)

6. 샘플 추출
- 현재 목표에서 모델의 성능은 고려하지 않기 때문에 마지막 체크포인트를 사용
- 깃헙 코드(https://colab.research.google.com/github/magenta/magenta-demos/blob/master/colab-notebooks/MusicVAE.ipynb#scrollTo=zRUlAshMpDnR) 참고하여 샘플 추출

## 상세 내용
- VAE
  - 훈련 데이터가 가지는 데이터 분포와 같은 분포에서 샘플링된 값으로 새로운 데이터를 생성하는 Generative model로써 해당 데이터와 유사하지만 완전히 새로운 데이터를 생성하는 모델
  - 입력 데이터가 들어오면 해당 데이터에서의 다양한 특징들이 각각의 확률 변수가 되는 확률 분포를 만들어, 입력데이터의 분포를 잘 근사하는 데이터를 생성


![vae_구조](https://user-images.githubusercontent.com/58794670/187360174-d6cfa03a-6521-4848-8561-a0062f9a6d3d.png)

- VAE의 단점
  - 시퀀스가 길어지면 성능이 떨어지는 문제가 있음
  - 따라서 MusicVAE에서는 hierarchical decoder를 사용함
  - 이 구조는 모델이 잠재 코드를 활용하도록 장려하여 posterior collapse 문제를 방지

    - posterior collapse
      - VAE 모델은 latent z를 이용하여 시퀀스를 생성하는데, 이때 decoder가 encoder를 무시하고 sequence를 생성하는 현상
      - 원인
        1. Decoder가 latent z없이 과거 데이터만으로 충분히 generation이 가능한 경우
        2. 조건에 맞는 다양한 latent z가 존재할 수 있는 가능성이 있는 경우
        3. VAE의 local information을 선호하는 경향
        4. 학습 초기에 encoder가 meaningful z를 표현하지 못한 경우
        5. 가정한 Gaussian prior에 아무런 의미 있는 정보가 없는 경우
        6. ELBO와 evidence 사이의 gap, true posterior approximation의 실패

- 데이터 구조
  - 참고 논문에 따르면 각 음표를 16개의 간격으로 양자화하여 각 마디를 16개의 이벤트로 구성함
  - 따라서 1마디에 길이 16, 4마디는 데이터 길이가 64가 되고, 각 시퀀스는 하나의 마디에 해당함

- 모델
  - 논문에서 사용한 활성화 함수로는 Adam을 사용하며 batch_size 는 512
