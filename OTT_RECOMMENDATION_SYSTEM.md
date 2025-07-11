# OTT 추천 시스템

> **콘텐츠 1~8개와 해당 콘텐츠들을 방영하는 OTT들을 추천하는 시스템**
> - 링크: [2025-MJU-Social/RecommendModel_withFastAPI](https://github.com/2025-MJU-Social/RecommendModel_withFastAPI/tree/test)

## 1. 프로젝트 개요

- **참가 인원**: 3인 프로젝트
- **기간**: 2025/05 ~ 2025/06 (약 6주)
- **기술 스택**: React, FastAPI, Pytorch
- **담당 파트**: 시스템 개발, 풀스택

오늘날 국내 OTT 시장은 수평, 수직적으로 확대되고 있으며, 소비자 입장에서는 이를 모두 구독하기 어렵다는 단점이 존재합니다. 이에 따라 사용자 취향에 맞는 콘텐츠를 추천하면서, 최대한 많은 추천 콘텐츠들을 사용자의 OTT 사용 시간 내에 시청할 수 있도록 효율적으로 OTT 구독을 관리할 수 있는 시스템을 설계하였습니다.

## 2. 프로젝트 설계

### 2.1 사용된 기술

시스템에 적용된 핵심 기술과 이론은 세 가지입니다.

첫 번째는 **자연어 처리 기술**입니다. Sentence Transformer라는 최신 NLP 모델을 사용하여 '스릴러', '로맨스', '액션'과 같은 장르명을 컴퓨터가 이해할 수 있는 384차원의 숫자 벡터로 변환하였습니다. 이 과정을 통해 ‘스릴러’와 ‘액션’의 유사도가 0.73으로 계산되는 등, 인간이 직관적으로 느끼는 장르 간 관계를 수치로 정확히 표현할 수 있게 하였습니다.

두 번째는 **유사도 계산 기술**입니다. 코사인 유사도를 기반으로 고차원 벡터 공간에서 두 벡터 간의 각도를 측정하여 유사성을 판단합니다. 이를 통해 장르뿐만 아니라 총 6가지 차원에서 각각 다른 가중치를 부여해 최종 추천 점수를 산출합니다.

세 번째는 **동적 프로그래밍 알고리즘 활용**입니다. 추천 콘텐츠를 특정 조건에 따라 필터링하는 과정에서 최대한 많은 콘텐츠를 추천할 수 있도록 이 알고리즘을 적용하였습니다.

### 2.2 데이터 수집

키노라이츠 웹사이트에서 Selenium 라이브러리를 활용하여 콘텐츠 데이터를 수집하였습니다. 연령대와 성별에 따라 다양한 콘텐츠를 확보하였으며, 해당 랭킹은 내부 인기도와 국내 미디어 트렌드 데이터를 종합하여 산정된 수치입니다.

세부 장르 데이터는 별도로 제공되지 않았기 때문에, 각 콘텐츠 타이틀을 위키백과 URL에 포함시켜 스크래핑을 통해 세부 장르, 출연 배우, 러닝타임 등의 정보를 수집하였습니다.

### 2.3 추천 시스템 종류

추천 시스템의 전체 흐름은 다음과 같습니다. 사용자가 연령대, 성별, 선호 장르, 주 시청 시간, 월 구독 예산, 좋아하는 콘텐츠 제목을 입력하면, 서버는 두 개의 추천 시스템에 해당 정보를 전달합니다.

첫 번째 시스템은 OTT만 추천하는 서비스입니다. 연령대, 성별, 좋아하는 콘텐츠 제목을 기반으로, 해당 콘텐츠와 유사한 콘텐츠가 존재하는 OTT를 추천합니다. 이 시스템은 참고용 지표로 활용됩니다.

두 번째 시스템은 콘텐츠와 OTT를 동시에 추천하는 서비스입니다. 연령대, 성별, 선호 장르, 주 시청 시간, 월 구독 예산 정보를 입력받아, 코사인 유사도를 기반으로 콘텐츠를 추천하고, 이를 시청할 수 있는 OTT를 함께 추천합니다.

### 2.4 콘텐츠 추천 프로세스

1. **SentenceTransformer를 활용하여 장르 유사도를 파악합니다.**
    
2. **성별 및 연령대별 콘텐츠 순위와 매치하여 점수를 부여합니다.**
    

![콘텐츠와 점수 예시](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%BD%98%ED%85%90%EC%B8%A0%EC%99%80%20%EC%A0%90%EC%88%98%20%EC%98%88%EC%8B%9C.png?raw=true)

장르 정보만으로도 추천이 가능하지만, 추천 정확도를 높이기 위해 연령대 및 성별에 따른 타겟 매칭도를 추가로 고려하였습니다. 예를 들어, 20대 남성 사용자가 있다고 가정하면 다음과 같은 결과가 도출됩니다.

- 콘텐츠1: 30대 남성 대상 3위, 연령 페널티 0.8 → 최종 점수 5.72
    
- 콘텐츠2: 20대 남성 대상 21위, 연령 일치 1.0 → 최종 점수 3.12
    
- 콘텐츠3: 20대 여성 대상 1위, 성별 페널티 0.3 → 최종 점수 3.60
    

결국 추천 순위는 콘텐츠1 > 콘텐츠3 > 콘텐츠2로 결정됩니다.

![통합 점수 결과](https://github.com/1Dohyeon/Projects/blob/main/imgs/%ED%86%B5%ED%95%A9%20%EC%A0%90%EC%88%98%20%EA%B2%B0%EA%B3%BC.png?raw=true)

3. **러닝타임 점수를 계산합니다.**
    
러닝타임이 길수록 점수를 낮추는 방식으로 설계하였습니다. 기준은 10시간이며, 시리즈 콘텐츠가 이 기준을 초과하면 점수가 감소합니다. 영화는 평균 2시간으로 대부분 1점을 부여합니다.

![이용 시간 자료](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%9D%B4%EC%9A%A9%20%EC%8B%9C%EA%B0%84%20%EC%9E%90%EB%A3%8C.png?raw=true)

국내 평균 시청 패턴(주 5일 × 일 2시간 = 주 10시간)을 고려하여 기준을 설정하였습니다.

4. **총 6개 점수에 가중치를 부여해 통합 점수를 계산하고, 상위 콘텐츠를 추천합니다.**
    
![통합 점수 결과 with 가중치](https://github.com/1Dohyeon/Projects/blob/main/imgs/%ED%86%B5%ED%95%A9%20%EC%A0%90%EC%88%98%20%EA%B2%B0%EA%B3%BC%20with%20%EA%B0%80%EC%A4%91%EC%B9%98.png?raw=true)

가중치는 주관적으로 설정하였지만, 변수별 상대적 중요도에 따라 다르게 적용하였습니다. 예를 들어 연령 유사도는 0.15, 성별 유사도는 0.1로 설정하였습니다.

![연령이 성별보다 중요한 이유](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%97%B0%EB%A0%B9%EC%9D%B4%20%EC%84%B1%EB%B3%84%EB%B3%B4%EB%8B%A4%20%EC%A4%91%EC%9A%94%ED%95%9C%20%EC%9D%B4%EC%9C%A0.png?raw=true)

이는 “KNN-based Recommender System” 논문 분석 결과를 기반으로 하였습니다. 성별보다 연령대가 더 유의미한 변수로 반복적으로 등장하였기 때문입니다.

최대 8개의 콘텐츠를 추천하되, 사용자의 1주 OTT 시청 시간과 월 구독 플랫폼 수를 고려하여 추천 수는 유동적으로 결정됩니다.

5. **동적 프로그래밍을 기반으로 콘텐츠 필터링을 수행합니다.**
    
![콘텐츠 추천 동적 알고리즘 예시1](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%BD%98%ED%85%90%EC%B8%A0%20%EC%B6%94%EC%B2%9C%20%EB%8F%99%EC%A0%81%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%20%EC%98%88%EC%8B%9C1.png?raw=true)

예를 들어 주간 시청 시간이 5시간이라면, 한 달 기준 총 20시간으로 계산하여 각 플랫폼별 시간 제한 내에서 가장 많은 콘텐츠를 추천할 수 있도록 동적 알고리즘을 적용합니다.

![콘텐츠 추천 그리디 알고리즘 예시](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%BD%98%ED%85%90%EC%B8%A0%20%EC%B6%94%EC%B2%9C%20%EA%B7%B8%EB%A6%AC%EB%94%94%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%20%EC%98%88%EC%8B%9C.png?raw=true)

그리디 방식은 비효율적인 결과를 낳을 수 있으나,

![콘텐츠 추천 동적 알고리즘 예시2](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%BD%98%ED%85%90%EC%B8%A0%20%EC%B6%94%EC%B2%9C%20%EB%8F%99%EC%A0%81%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%20%EC%98%88%EC%8B%9C2.png?raw=true)

동적 프로그래밍은 가능한 모든 조합을 고려하여 최적의 콘텐츠 집합을 도출합니다.

### 2.5 사용 예제와 OTT 추천

![사용자 입력 예시 ui](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EC%82%AC%EC%9A%A9%EC%9E%90%20%EC%9E%85%EB%A0%A5%20%EC%98%88%EC%8B%9C%20ui.png?raw=true)

위와 같이 사용자로부터 정보를 입력받으면, 아래와 같이 추천 콘텐츠와 해당 콘텐츠를 가장 저렴하게 시청할 수 있는 OTT를 함께 추천합니다.

![결과 예시 ui](https://github.com/1Dohyeon/Projects/blob/main/imgs/%EA%B2%B0%EA%B3%BC%20%EC%98%88%EC%8B%9C%20ui.png?raw=true)

사용자에게 콘텐츠를 반환하기 전, 추천된 OTT 간의 콘텐츠 집합을 비교하여 다른 플랫폼에서 대체 가능한 경우 중복 구독을 방지하는 **OTT 필터링 알고리즘**도 적용됩니다.

예를 들어 Netflix에서 추천된 콘텐츠들이 Wavve에서도 모두 시청 가능하다면, Netflix는 구독 추천 대상에서 제외됩니다.

## 3. 활용 한계점 및 확장성

현재 시스템은 사용자의 입력 정보에 기반하여 콘텐츠를 추천하고 있으나, 실제 시청 이력이나 평가 로그 등은 반영되지 않습니다. 이에 따라 개인화 수준에는 한계가 존재하며, 추천은 일반적인 선호 경향에 기반합니다.

향후 시청 로그와 평가 데이터를 축적하여 반영하게 된다면, 가중치를 모델이 직접 학습하게 하여 추천 정밀도를 향상시킬 수 있을 것으로 기대됩니다.

또한 추천 결과를 LLM과 연계하여 “이번 달엔 넷플릭스, 다음 달엔 티빙”처럼 자연어로 설명하는 컨설팅 형태로 확장할 수 있습니다. 이는 사용자가 점수나 리스트 대신 **직관적인 전략 제안**을 받을 수 있도록 도와줍니다.

나아가 생활 패턴이나 시청 환경까지 반영한 추천, 또는 구독 대비 만족도 분석을 통해 **자동 제안**하는 기능으로도 확장 가능하다고 판단하고 있습니다.

## 4. 성과 및 배운 점

- NLP 기반 콘텐츠 임베딩 및 벡터 연산을 활용한 추천 시스템 설계
- 다중 조건 최적화(다이나믹 알고리즘)를 실제 서비스 로직에 적용
- ML 모델 서빙과 프론트 통합을 통한 엔드 투 엔드 시스템 개발 경험