---
title: '[UE5 / Part1] 9. Delegate & Pub/Sub'
published: 2026-02-08
description: '[OOP 3/3] - 발행-구독(Pub/Sub) 패턴의 이해, 델리게이트 매크로(DECLARE) 활용, 그리고 브로드캐스팅을 통한 이벤트 기반 설계를 알아보자.'
image: '' 
tags: [Unreal Engine, C++, OOP]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.   
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   
>

# 강의 목표

- 느슨한 결합의 장점과 이를 편리하게 구현하도록 도와주는 델리게이트의 이해
- 발행 구독 디자인 패턴의 이해
- 언리얼 델리게이트를 활용한 느슨한 결합의 설계와 구현의 학습
# Delegate

이전에 배운 컴포지션(Composition)은 강한 결합 관계를 가진다.   
델리게이트(Delegate)는 약한 결합 관계를 가지며, 주로 구독자 패턴(Observer Pattern)을 구현할 때 사용한다.
- 언리얼 Delegate [공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/delegates-and-lamba-functions-in-unreal-engine?application_version=5.1)

# 발행 구독 디자인 패턴 (Pub/Sub)

![alt text](image-4.png)

- 푸시(Push) 형태의 알림(Notification)을 구현하는데 적합한 디자인 패턴
- `발행자(Publisher)`와 `구독자(Subscriber)`로 구분
    - 콘텐츠 제작자는 콘텐츠를 생산한다.
    - 발행자는 콘텐츠를 배포한다.
    - 구독자는 배포된 콘텐츠를 받아 소비한다.
    - 제작자와 구독자가 서로를 몰라도, 발행자를 통해 콘텐츠를 생산하고 전달할 수 있다.
- 장점
    - 제작자, 구독자는 서로를 모르기 때문에 `느슨한 결합(Loose Coupling)`으로 구성된다.
    - 유지 보수가 쉽고, 유연하게 활용될 수 있으며, 테스트가 쉬워진다.
    - 시스템 스케일을 유연하게 조절할 수 있으며(Scalability), 기능 확장(Extensibility)이 용이하다.

# 실습 시나리오

![alt text](image-5.png)
- `학사정보 (CourseInfo)` `학생(Student)`
    - 학교는 학사 정보를 관리한다.
    - 학사 정보가 변경되면 자동으로 학생에게 공지한다.
    - 학생은 학사 정보의 알림 구독을 해지할 수 있다.
- 시나리오 흐름
    - 학사 정보 시스템과 3명의 학생이 존재한다.
    - 시스템에서 학사 정보를 변경한다.
    - 학사 정보가 변경되면 알림을 구독한 학생들에게 변경 내용을 자동으로 전달한다.

# 언리얼 델리게이트 선언 시 고려사항

- 인자의 수와 각각의 타입을 설계
    - 일대일?
    - 일대다?
        
- 프로그래밍 환경 설정
    - Only C++?
    - UFUNCTION으로 지정된 블루프린트(BP) 함수와 함께 사용?
        
- 어떤 함수와 연결할 것인가?
    - 클래스 외부에 설계된 C++ 함수?
    - 전역에 설계된 정적 함수?
    - 언리얼 오브젝트의 멤버 함수? (보통 이 방식을 사용)
        

### 매크로 구조

`DECLARE_{델리게이트유형}DELEGATE{함수정보}`

- 델리게이트 유형
    - `DECLARE_DELEGATE` (일대일)
    - `DECLARE_MULTICAST` (일대다)
    - `DECLARE_DYNAMIC` (블루프린트 연동, 일대일)
    - `DECLARE_DYNAMIC_MULTICAST` (블루프린트 연동, 일대다)
        
- 함수 정보
    - `DECLARE_DELEGATE`
    - `DECLARE_DELEGATE_OneParam`
    - `DECLARE_DELEGATE_RetVal_ThreeParams`
    - 최대 9개 파라미터까지 지원
        

# 실습

## 설계 내용

- 학사정보 변경 시 `학교명`, `내용`을 학생에게 전달해야 함 -> 2개의 인자
- 변경된 학사 정보는 다수 인원을 대상으로 발송해야 함 -> MULTICAST
- 오직 C++ 프로그래밍에서만 사용 -> DYNAMIC 사용 X
    

결론: `DECLARE_MULTICAST_DELEGATE_TwoParams` 매크로 사용

## 1. CourseInfo

- **발행자 (Publisher)** 역할
- 델리게이트를 정의하고, 학사 정보가 변경될 때 `Broadcast`로 알림을 발행한다.

**CourseInfo.h**
```cpp
// delegate 매크로 선언
DECLARE_MULTICAST_DELEGATE_TwoParams(FCourseInfoOnChangedSignature, const FString&, const FString&);

UCLASS()
class UNREALDELEGATE_API UCourseInfo : public UObject
{
    GENERATED_BODY()
public:
    UCourseInfo();

    // 델리게이트 멤버 변수 선언
    FCourseInfoOnChangedSignature OnChanged;
    
    // 외부에서 학사정보 변경 요청 시 호출되는 함수
    void ChangeCourseInfo(const FString& InSchoolName, const FString& InNewContents);

private:
    FString Contents;
};
```

**CourseInfo.cpp**
```cpp
UCourseInfo::UCourseInfo()
{
    Contents = TEXT("기존 학사 정보");
}

void UCourseInfo::ChangeCourseInfo(const FString& InSchoolName, const FString& InNewContents)
{
    // 5. 학사 정보 내용 업데이트
    Contents = InNewContents;
    
    UE_LOG(LogTemp, Log, TEXT("[CourseInfo] 학사 정보가 변경되어 알림을 발송합니다."));
    
    // 6. 델리게이트를 통해 알림 발행 (구독자들에게 방송)
    OnChanged.Broadcast(InSchoolName, Contents);
}
```

## 2. Student
- **구독자 (Subscriber)** 역할
- 발행자(CourseInfo)가 알림을 보내면 자신이 등록한 함수(`GetNotification`)를 자동으로 호출받는다.   

**Student.h**
```cpp
UCLASS()
class UNREALDELEGATE_API UStudent : public UObject
{
    GENERATED_BODY()

public:
    // 알림을 받을 함수
    void GetNotification(const FString& School, const FString& NewCourseInfo);
private:
    FString Name;
};
```

**Student.cpp**
```cpp
// 7. 모든 구독자 학생들이 알림을 받음
void UStudent::GetNotification(const FString& School, const FString& NewCourseInfo)
{
    UE_LOG(LogTemp, Log, TEXT("[Student] %s 님이 %s로부터 받은 메시지 : %s"), *Name, *School, *NewCourseInfo);
}
```

## 3. MyGameInstance

- **조율자 (Coordinator)** 역할
- 발행자와 구독자를 연결해주는 역할을 한다.
    - `CourseInfo`를 생성
    - `Student`들을 생성한다. (여기서는 3개)
    - Student들을 CourseInfo의 델리게이트에 바인딩(구독)시킨다.
- 유일하게 두 클래스(`CourseInfo`, `Student`)를 모두 `include`한다.

**MyGameInstance.h**
```cpp
private:
    UPROPERTY()
    TObjectPtr<class UCourseInfo> CourseInfo;
```

**MyGameInstance.cpp**
```cpp
void UMyGameInstance::Init()
{
    Super::Init();

    // CourseInfo는 MyGameInstance의 SubObject가 되고
    // MyGameInstance는 CourseInfo의 Outer가 됨
    // 1. Composition 관계로 CourseInfo 생성 (발행자)
    CourseInfo = NewObject<UCourseInfo>(this); 

    UE_LOG(LogTemp, Log, TEXT("==============================="));

    // 2. Student 객체 생성 (구독자)
    UStudent* Student1 = NewObject<UStudent>(this);
    Student1->SetName(TEXT("학생1"));
    UStudent* Student2 = NewObject<UStudent>(this);
    Student2->SetName(TEXT("학생2"));   
    UStudent* Student3 = NewObject<UStudent>(this);
    Student3->SetName(TEXT("학생3"));   
    
    // 3. 모든 학생을 구독자로 등록 (Binding)
    CourseInfo->OnChanged.AddUObject(Student1, &UStudent::GetNotification);
    CourseInfo->OnChanged.AddUObject(Student2, &UStudent::GetNotification);
    CourseInfo->OnChanged.AddUObject(Student3, &UStudent::GetNotification);

    FString SchoolName = TEXT("언리얼대학교");
    
    // 4. 발행자에게 정보 변경 요청 (이벤트 발생 트리거)
    CourseInfo->ChangeCourseInfo(SchoolName, TEXT("변경된 학사 정보"));
}
```

## 출력 결과
```
LogTemp: ===============================
LogTemp: [CourseInfo] 학사 정보가 변경되어 알림을 발송합니다.
LogTemp: [Student] 학생3 님이 언리얼대학교로부터 받은 메시지 : 변경된 학사 정보
LogTemp: [Student] 학생2 님이 언리얼대학교로부터 받은 메시지 : 변경된 학사 정보
LogTemp: [Student] 학생1 님이 언리얼대학교로부터 받은 메시지 : 변경된 학사 정보
```

## 전체 흐름 요약
1. `MyGameInstance`: `CourseInfo` 생성 (발행자 준비)
2. `MyGameInstance`: `Student` 객체 생성 (구독자 준비)
3. `MyGameInstance`: 학생들을 `CourseInfo.OnChanged` 델리게이트에 구독자로 등록 (`AddUObject`)
4. `CourseInfo`: 학사 정보 변경 시 델리게이트 `Broadcast` 실행 (모든 학생에게 발행)
5. `Student`: 등록된 함수 `GetNotification` 자동 호출 (구독자 알림 수신)

# 정리

### 델리게이트 vs 컴포지션

- **델리게이트**: **느슨한 결합** (기능적 의존성 제거, 구독자 패턴 활용)
- **컴포지션**: **강한 결합** (객체의 생성과 소멸 주기를 완전히 소유)
    

### 느슨한 결합 (Loose Coupling)의 장점

- 향후 시스템 변경 사항에 대해 손쉽게 대처할 수 있다.
- 발행-구독 모델의 이점
    - 클래스는 자신이 해야 하는 작업에만 집중 가능
    - 외부 변경 사항에 영향을 받지 않음
    - 자신의 기능을 확장해도 다른 모듈에 영향을 주지 않음

### 언리얼 C++ 델리게이트 선언시 생각할 점

적합한 매크로를 선택하기 위해 다음을 고려한다
1. **인자의 개수**: 파라미터가 몇 개 필요한가?
2. **동작 방식**: 1:1 통신인가, 1:N(Multicast) 통신인가?
3. **블루프린트 연동**: 에디터/BP에서 접근이 필요한가? (Dynamic 필요 여부)

### 기타: UML에서의 표기
- 따로 번외편에서 정리.


