

#6장. 웹어플리케이션 보안

> SQL Injection 공격?

> XSS 공격?

> CSRF 공격?

취약점 점검 : 설계상 취약점, 관리자 설정 부주의 등

CVE : 알려진 취약점

취약점 : 제로데이 원데이 올데이

취약점 점검 방법

자동진단(자동화 점검 도구), 수동진단(점검자가 직접 매뉴얼로 진단)
소스코드 점검 : secure coding - 취약점이 있는 library를 갖다 쓰는 경우

스캐닝 : 공격 대상에 대한 정보를 얻기

호스트 탐색 : 네트워크에 연결되어 있는 시스템을 파악하는 것 : **NAT로 보완** (내부 시스템을 스캐닝하지 못하도록)

포트 스캐닝 : **어떤 포트를 통해(어떤 어플리케이션을 통해) 접근할 수 있는지**

운영체제 탐색 : 일밙거으로 운영체제 탐색은 포트 스캐닝과 함께 이루어짐

보안 취약점 점검 : 다양한 점검도구가 있다

웹 프록시 툴 : 모두 지나가게 하여 스니핑하는 도구

버프슈트 : 웹 프록시 도구

쿡시 툴바 : 쿠키값을 제어할 수 있다, 다양한 기능이 존재

OWASP-ZAP : 산업 표준(de facto) OWASP의 공개용 웹 취약점 점검 진단 기준
- **Fuzzer** : 취약점을 찾는 툴 (다양한 입력, interrup를 걸어서(Fuzzing) crash를 유도하여 취약점 점검)
- **AI Fuzzer** : 단순한 입력이 아니라 다양한 시도를 통해 Fuzzing을 하고 이를 통해 깊이있는 취약점을 발견

모의 취약점 점검 : blue team, red team, ~~~ box

가상 머신 환경 구축 : 웹 모의 해킹에서 사용, **honeypot, honeynet**

웹 어플리케이션 보안 취약점 : SQL Injection XCC, 파일 업로드

운영체제 명령 실행 취약점

파일 업로드 취약점 : 웹쉘 업로드 시도

SQL Injection 취약점 : 문자열에 대해 유효성 검사를 실시

Xpath Injection 취약점 : 특정 디렉터리에 파일

=> **입력값 검증이 중요**

정보누출 취약점 : 주석구문을 포함시키 않도록

평문전송 취약점 : 시큐어 코딩, 암호화 필요

관리자 페이지 노출 취약점

위치 공개 취약점

경로 추적 취약점 :디렉터리 접근 통제

악성 콘텐츠 취약점

**XSS 취약점 : 공격자의 스크립트가 피공격자의 시스템에서 실행되고 피공격자의 쿠키 정보를 전송하여 추가적인 공격 실행**

취약점 점검 방법, 대응 방법

약한 문자열 강도 취약점

불충분한 인증 및 인가 취약점

**불충분한 세션 관리 취약점**

**CSRF 취약점 (다른 시스템에서 위조된 요청을 보낸다)**
1. 공격자는 CSRF 스크립트를 업로드
2. 피해자는 해당 게시글을 통해 스크립트 다운로드
3. 피해자의 권한으로 CSRF 실행 (조작된 요청이 작성된 스크립트가 실행)

: 임의의 토큰을 추가하여 정상/비정상 요청을 판별한다

쿠키 변조 취약점

continuous Auth : 쓰다가 중간에 한번씩 체크

디렉터리 인덱싱 취약점

자동화 공격 취약점

구글 해킹 : 개인정보 등을 이용

**Mydata 사업 : 개인정보(금융정보, 의료정보)를 개인이 통제할 수 있게 한다**`
`
#7장. 악성코드분석 기초

네트워크를 타고 퍼져나가는것 : 웜

트로이목마 : 알려진 프로그램 안에 실행코드를 넣는다

바이러스 분류
- 기생형 바이러스
- 겹쳐 쓰기형 바이러스
- 산란형 바이러스
- 연결형 바이러스
- ...

매크로 바이러스

트로이 목마

이메일을 통한 유포, 드라이브 바이 다운로드 (Redirect)

긴급 패치, 백신 제작

악성코드 분석 방법 : 디버깅 (Reverse Engineering)

안티 리버싱, 역분석 크래커를 지치게 하는 것

동적 분석 : 실행시켜서 분석, code coverage 범위가 중요

행위 분석 방식, 리버싱

YARA : 시그니쳐

정적 분석(Graph, Flow 활용) : wrapping(packing), 바이너리 코드 꺼내기도 어렵다.
요즘은 외부 서버와 연결하여 라이브러리 링크하는 기능 사용

#7장-1. 안드로이드 악성코드분석 기초

안드로이드 악성코드 분석 방법

안드로이드 APK 추출 (1)

웹사이트, Google Play Store

ADB를 이용하여 추출

정적 분석 기술
- Manifest 분석 : AndroidManifest.xml 파일을 분석 : Permission 제공
- SDK 분석 : classes.dex 파일을 디컴파일하여 분석
- NDK 분석 ; .so 파일 분석
- 응용 분석 : Taint, Data flow Analysis

안드로이드 악성코드 정적 분석 : apktool을 이용해 리소스 형태로 압축 해제

AndroidManifest.xml 분석

Smali 분석 : Dex바이너리를 사람이 읽을 수 있도록 표현한 언어

dex2jar를 이용한 jar 파일 생성

jd-jui를 이용한 java 코드 확인

안드로이드 Repackaging을 통한 분석 : 소스코드에 후킹을 대신하는 바이너리 커널 코드를 붙여 repackaging한다

난독화 기법이 적용된 악성코드

**패킹**
Compressor, Protector

안드로이드 정적 분석의 한계

동적 분석 기술

안드로이드 악성코드 동적 분석

안드로이드 디버깅

동적 분석 툴 - Frida



# 8장. 침해사고분석 및 디지털포렌식

침해사고분석 절차

정보 수집 단계 :
  시스템 정보 수집 : 메모리 스냅샷
  네트워크 기반 정보 수집

윈도우 침해사고 정보 수집 및 분석 방법
  시스템 로그를 지우더라도 로그 서버에는 남아 있도록 한다

**휘발성 정보, 비휘발성 정보**
휘발성 정보 : 껐을 때 없어지는 데이터
압수수색시

**침해사고 대응 방법**
reactive : 터진 뒤 대응
preactive : 사전 대응

False-Positive를 줄이는 것이 중요

**AI 시스템 (SOAR) : 인공지능 이용**

디지털 포렌식

디지털 증거 : 법적 증거를 가지기 위해 논리적이고 표준화된 절차로 진행

디지털 포렌식의 5대 원칙
정당성의 원칙
재현의 원칙
연계성의 원칙
무결성의 원칙
신속성의 원칙

디지털 증거의 4대 원칙
진정성 범죄 현장에 그 데이터가 존재해야   한다
무결성 조작이 안되었느냐
원본성 원본이 맞냐
신뢰성 신뢰할만한 데이터냐


# 9장-1. 사이버보안에서의 적대적 공격 및 대응 방안

Introduction
  알고리즘 + 학습 데이터 -> 학습된 정보 (learned parameters)
  ML을 통해 사이버 보안문제를 해결

Taxonomy of Attacks
  공격자가 머신 러닝을 공격하는 방법
    poisoning attack : training 데이터를 오염
    evasion attack : 잘못된 결정을 하도록 만드는 샘플을 사용한다
    model extraction stealing : Model을 추출, 스틸

Blackbox vs Whitebox Attack
  공격자가 모를 때 : Blackbox attack, query
  알 때 : Whitebox, back propagation, query

Machine Learning : Security & privacy
  GAN (Generative Adversarial Network)

GAN
  Generator : Discriminator를 속이기 위한 가짜 데이터를 생성
  Discriminator : 진짜와 가짜를 판별

Adversarial Attack
  Evasion attack : 테스트시 교란을 통해 샘플을 잘못 분류시킴
    non-targeted : 오분류만 하게끔 한다 (올바른 예측)
    targeted : 공격자가 원하는 특정한 클래스로 유도하여 틀린 예측을 하게 한다
    무작위 섭동(random perturbation) : random noize를 준다
    FGSM(Fast Gradien Sign Method) : 손실의 기울기를 최대화하는 이미지 생성

Poisoning Attack : 의도적으로 유해한 학습 데이터를 주입
Backdoor(Trojan) Attack : 잘못된 네트워크를 병합하여 잘못 동작하도록 한다
Transferable Attack : 타겟 모델을 학습한 대체 모델을 사용해 적대적 예제를 생성
Model Stealing/Extraction : 쿼리를 계속 던지면서, 결과값을 분석해 유사한 모델을 생성
Further attack : 도용한 모델에 대한 적대적 샘플을 생성하면 실제 모델을 생성할 수 있다
Attribute Inference & Model Inversion : 숨겨야할 데이터를 찾을 수 있다

**공격 정리**
- Poisoning Attack : 데이터 자체를 오염
- Evasion Attack : 입력 데이터에 Noise를 줘서 잘못 오분류하게 만드는 것
- Model Extraction Attack : 쿼리를 던져 얻은 output을 종합해 대체 모델을 생성
- Model Inversion : Query로 얻은 값을 가지고 반대로 Training Data를 찾아낸다

**방어 정리**
- Adversarial Retraining : Training stage에서 적대적 예제를 포함시킨다, DNN의 견고함을 향상
- Adversarial Detection : 적대적 예제로 감지되는 경우 차단한다, 탐지기로 이상 ReLU 활성화를 확인하여 탐지한다
- Network Distillation : 큰 모델에서 정확도를 유지하며 작은 모델을 만들며 DNN의 민감도를 낮출 수 있다면
                        Perturbation Noise에 의한 Cross boundary를 줄일 수 있다
- Input Reconstruction : 적대적 예제를 reconstruction을 통해 clean data로 변환한다 (AutoEncoder)

이미지에서는 제약이 없고
악성코드는 constraint domain이다

**Trust AI**
- robust한 알고리즘 필요 (공격에 강인한)
- explainability (AI의 결정은 설명이 가능해야 한다)
- AI ethics 결정 알고리즘이 공익/사익을 추구하도록 하는 것

# 10장. 클라우드 보안 이슈와 인증제도
