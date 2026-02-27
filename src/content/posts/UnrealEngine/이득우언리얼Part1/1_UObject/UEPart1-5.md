---
title: '[UE5/Part1] 5. UObject Reflection 1'
published: 2026-02-04
description: 'UObject<->UClass객체 매칭, 메타데이터, CDO, Assertion함수의 개념을 알아보자'
image: '' 
tags: [Unreal Engine, C++]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   
> 
> 문서 내용 정리 -> 강의 요약 -> 실습 & 분석 -> 번외 순으로 정리되어 있습니다.   
> 복습할때는 실습&분석 파트를 읽는걸 권장

# 강의 목표
- 언리얼 오브젝트의 특징과 리플렉션 시스템의 설명
- 언리얼 오브젝트의 처리 방식의 이해
---
# (문서) Unreal Property System
[링크](https://www.unrealengine.com/ko/blog/unreal-property-system-reflection)
Reflection은 반사를 의미하는 그래픽 용어와 혼동되어 Unreal Property System이라고도 부른다.
- `Reflection`은 프로그램이 실행시간에 자기 자신을 조사하는 기능
	- but) C++은 지원하지 않아, 언리얼 자체적으로 구축되어 있음

```cpp
include "FileName.generated.h"
```

- UENUM(), UCLASS(), USTRUCT(), UFUNCTION(), UPROPERTY() 등의 매크로를 추가하면 언리얼 헤더 툴이 적절한 코드들을 자동으로 Intermediate 폴더 내부에 생성

- 모든 멤버 변수가 반드시 `UPROPERTY()` 로 설정될 필요는 없음
- 리플렉션된 프로퍼티가 아닌 것들은 UE시스템에서 관리를 받지 않음
	- GC가 레퍼런스를 확인 못함 (위험)
	- `UPROPERTY()` 붙이면 아넌
- UHT는 실제 C++ 파서가 아님 (한계)
	- 너무 족잡한 유형은 UHT이 읽어드리지 못함
---
- 프로퍼티 시스템의 계층 구조
	- 추후 강의에서 자세히 다룸
```
UField
	UStruct - 기본적인 종합 구조체
		UClass (C++ class) - 자손으로 함수나 프로퍼티 포함 가능
		UScriptStruct (C++ struct) - 자손은 프로퍼티로만 제한
		UFunction (C++ function)   - 자손은 프로퍼티로만 제한
	UEnum (C++ enumeration)
	UProperty (C++ member variable or function parameter)
		(Many subclasses for different types)
```

이렇게 구축된 UnrealObject 클래스는 Static 클래스 함수나 Get 클래스 함수로 접근 가능
- 이 함수를 호출하면 이러한 Reflection, UHT이 분석해서 만들어 놓은 Reflection 정보들을 보관한 특정 객체에 접근 가능

```cpp
for (TFieldIterator<UProperty> PropIt(GetClass()); PropIt; ++PropIt)
{
	UProperty* Property = *PropIt;
	// Do something with the property
}
```
`UProperty`라는 UnrealObject를 사용해서 순회하면, UnrealObject의 속성들을 우리가 조회하면서 거기에 대한 값이나 정보를 빼올 수 있다.
- 다음 강의 실습 예제로 확인
---
# (이론 강의) 
## 언리얼 오브젝트의 구성
- 언리얼 오브젝트에는 특별한 프로퍼티와 함수 지정 가능
	- UPROPERTY : 클래스 멤버 변수
	- UFUNCTION : 클래스 멤버 함수
- 모든 언리얼 오브젝트는 클래스 정보와 함께 함
  - 모든 프로퍼티 & 함수 정보를 언제든 조회 가능
    - 컴파일 타임 : `UMyGameInstance::StaticClass();`
    - 런타임 : `GetClass();`
- 이런 기능을 제공하는 언리얼 오브젝트는 NewObject API를 사용해 생성해야 함.
![alt text](image.png)

[언리얼 오브젝트 처리](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/unreal-object-handling-in-unreal-engine?application_version=5.7)
## Class Default Object; CDO
- 언리얼 클래스 정보에는 클래스 기본 오브젝트 `CDO`가 포함됨
- `CDO`는 언리얼 객체가 가진 기본 값을 보관하는 템플릿 객체
- 가정\) 한 클래스로부터 다수의 물체를 생성해 게임 콘텐츠에 배치하는 상황
  - 일관성 있게 기본 값을 조정하는데 유영하게 사용
- 클래스 정보`UClass` -> `GetDefaultObject()` = `CDO` 추출 가능
- `UClass`, `CDO`는 엔진 초기화 과정에서 생성
  - 직접 디버그 걸어서 확인 가능 (엔진 로드 75%쯤에서 브레이크 걸림)
![alt text](image-1.png)

UClass()와 CDO가 어떻게 동작하는지 실습을 통해 알아보자


# 0. 실습용 코드 구조
- 헤더 구조는 그대로 가져가고, cpp는 실습에 맞춰 수정
MyGameInstance.h
```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"

/**
 * 
 */
UCLASS()
class OBJECTREFLECTION_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
public:
	UMyGameInstance(); // 생성자

	virtual void Init() override;

private:
	UPROPERTY() // 이래야 UE가 관리해줌(GC, 직렬화, 리플렉션 등)
	FString SchoolName;
};
```

MyGameInstance.cpp
```cpp
#include "MyGameInstance.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("기본 학교"); // 이 기본값은 CDO 템플릿 객체에 저장
}

void UMyGameInstance::Init()
{
	Super::Init();

	UE_LOG(LogTemp, Log, TEXT("===================="));
	UClass* ClassRuntime = GetClass();
	UClass* ClassCompile = UMyGameInstance::StaticClass();
	UE_LOG(LogTemp, Log, TEXT("학교를 담당하는 클래스 이름 : %s"), *ClassRuntime->GetName());

	SchoolName = TEXT("임의의 학교");
	UE_LOG(LogTemp, Log, TEXT("학교 이름 : %s"), *SchoolName);
	UE_LOG(LogTemp, Log, TEXT("학교 이름 기본값 : %s"), *GetClass()->GetDefaultObject<UMyGameInstance>()->SchoolName);

	UE_LOG(LogTemp, Log, TEXT("===================="));
}
```

---


# 1. 검증 코드 (Assertion 함수)

MyGameInstance.cpp
```cpp
#include "MyGameInstance.h"
UMyGameInstance::UMyGameInstance()
{
}

void UMyGameInstance::Init()
{
	Super::Init();

	UE_LOG(LogTemp, Log, TEXT("===================="));
	UClass* ClassRuntime = GetClass();
	UClass* ClassCompile = UMyGameInstance::StaticClass();
	check(ClassRuntime == ClassCompile);

	UE_LOG(LogTemp, Log, TEXT("학교를 담당하는 클래스 이름 : %s"), *ClassRuntime->GetName());

	UE_LOG(LogTemp, Log, TEXT("===================="));
}
```

로그
```bash
LogTemp: ====================
LogTemp: 학교를 담당하는 클래스 이름 : MyGameInstance
LogTemp: ====================
```
- 정상적으로 `check()` Assertion 함수를 통과하고, 로그에 MyGameInstance가 찍히는걸 볼 수 있다.

## 1-1. check()
MyGameInstance.cpp
```cpp
check(ClassRuntime != ClassCompile);
```
- 이번에는, check를 부정이 뜨도록 바꿔보자.

Unreal Crash Reporter
```bash
LoginId:ec4d00644b8bfc8993dcb989a922f693
EpicAccountId:4d1d3208fcc147eba2964ad944fd2b63

Assertion failed: ClassRuntime != ClassCompile [File:F:\UE5\UE5Part1\ObjectReflection\Source\ObjectReflection\MyGameInstance.cpp] [Line: 17] 
```
- 컴파일 후, 언리얼을 실행해보면, 위와 같이 Crash Reporter가 뜨며 에디터가 다운된다.
	- `Assertion failed: ClassRuntime != ClassCompile`에서 Assertion 실패
## 1-2. ensure()
MyGameInstance.cpp
```cpp
ensure(ClassRuntime != ClassCompile);
```

Output Log
![[Pasted image 20260204093829.png]]
- 실행하면, 에디터는 돌아가나 로그에 빨간 에러가 잡힌다.

## 1-3. ensureMsgf()
MyGameInstance.cpp
```cpp
ensureMsgf(ClassRuntime != ClassCompile, TEXT("에러 발생 !!!"));
```
Output Log
![[Pasted image 20260204094409.png]]
- 로그에, 우리가 입력한 TEXT도 함께 잡힌다.

## 정리
- `check()`와 같은 함수를 계속 넣어주며 검증하기
	- 실제 게임으로 빌드할때는 모두 사라지니 안심하고 사용하자

---

# 2. Class Default Object; CDO
## 정의
- UE엔진은 모든 `UClass`타입에 대해 기본 객체 (Default Object)를 하나 자동으로 생성. 
	- 이 기본 객체를 `Class Default Object; CDO`라고 부른다.
## 역할
- 해당 클래스의 기본값(Default Propery Values)을 저장하는 템플릿 역할
- 새 인스터스를 만들 때, 이 CDO에 저장된 값들을 복사해서 초기화
- 블루프린트 에디터에서 Default Values 창에 보이는 값들이 사실상 CDO에 들어있는 값

## 특징
- 엔진이 자동으로 생성
	- 개발자가 `new`로 만드는 게 아님.
- `StaticClass()->GetDefaultObject()`로 접근 가능

## 실습
### 2-1. CDO를 덮어주는 값 출력
MyGameInstance.cpp
```cpp
#include "MyGameInstance.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("기본 학교"); // 이 기본값은 CDO 템플릿 객체에 저장
}

void UMyGameInstance::Init()
{
	Super::Init();

	UE_LOG(LogTemp, Log, TEXT("===================="));
	UClass* ClassRuntime = GetClass();
	UClass* ClassCompile = UMyGameInstance::StaticClass();
	UE_LOG(LogTemp, Log, TEXT("학교를 담당하는 클래스 이름 : %s"), *ClassRuntime->GetName());

	SchoolName = TEXT("임의의 학교"); // 현재 인스턴스의 멤버 변수를 덮는다. (CDO로 설정된 초기값이 덮어짐)
	UE_LOG(LogTemp, Log, TEXT("학교 이름 : %s"), *SchoolName);

	UE_LOG(LogTemp, Log, TEXT("===================="));
}
```
- 위와 같이 SchoolName = TEXT()로 CDO에 저장된 값을 덮어씌우면, 정상적으로 변경된다.
```bash
LogTemp: ====================
LogTemp: 학교를 담당하는 클래스 이름 : MyGameInstance
LogTemp: 학교 이름 : 임의의 학교
LogTemp: ====================
```


### 2-2. CDO 기본값 출력
MyGameInstance.cpp
```cpp
SchoolName = TEXT("임의의 학교");
UE_LOG(LogTemp, Log, TEXT("학교 이름 : %s"), *SchoolName);
UE_LOG(LogTemp, Log, TEXT("학교 이름 기본값 : %s"), *GetClass()->GetDefaultObject<UMyGameInstance>()->SchoolName);
```
- 이번에는, `*GetClass()->GetDefaultObject<UMyGameInstance>()->SchoolName`로 CDO 기본값을 출력해보자

```bash
LogTemp: ====================
LogTemp: 학교를 담당하는 클래스 이름 : MyGameInstance
LogTemp: 학교 이름 : 임의의 학교
LogTemp: 학교 이름 기본값 : 
LogTemp: ====================
```
- 위와 같이 기본값이 안나오는데...
- Class Default Object를 고쳐주는 생성자 코드를 변경하는 경우에도 헤더 파일을 고치는 것과 똑같이 에디터를 꺼줘야한다.
	- 헤더 파일의 리플렉션 정보에 구조를 변경
	- 생성자 코드에서 CDO의 기본값을 변경
```bash
LogTemp: ====================
LogTemp: 학교를 담당하는 클래스 이름 : MyGameInstance
LogTemp: 학교 이름 : 임의의 학교
LogTemp: 학교 이름 기본값 : 기본 학교
LogTemp: ====================
```
- 에디터를 다시 실행하면 정상적으로 로그에 찍히는걸 볼 수 있다.
---


# 3. 정리
## UObject <-> UClass 매칭
- 모든 `UObject`기반 클래스는 자신을 설명하는 `UClass`객체를 하나 가진다.
- `StaticClass()`로 해당 클래스의 `UClass`객체에 접근이 가능하다.
- `UClass()`aozmfhsms UHT가 해당 클래스를 리플렉션 시스테멩 등록하도록 표시한다.

## UClass의 역할
- `UClass`는 다음 2가지의 요소를 관리한다.
### 1. 클래스 메타데이터
- 클래스 이름, 함수 목록, 프로퍼티 목록 등
	- 클래스 자체의 정보
### 2. CDO (Class Default Object)
- 기본값을 담은 템플릿 객체
	- 인스턴스 생성 시 초기화 기준으로 사용
	- `UClass`는 CDO의 포인터를 가지고 있어 런타임에 접근 가능

## 생성 및 반영
- 클래스 정보와 CDO는 엔진 초기화 과정에서 자동으로 생성되므로 안전하게 사용 가능
- 헤더 변경 or 생정자에서 CDO 기본값 수정시 -> 에디터 종료 후 다시 컴파일해야 정상 반영

---
---
---

# 4. 번외
## UPROPERTY()
### 매크로 분석
```cpp
// These macros wrap metadata parsed by the Unreal Header Tool, and are otherwise
// ignored when code containing them is compiled by the C++ compiler
#define UPROPERTY(...)
#define UFUNCTION(...)
#define USTRUCT(...)
#define UMETA(...)
#define UPARAM(...)
#define UENUM(...)
#define UDELEGATE(...)
#define RIGVM_METHOD(...)
#define VMODULE(...)
```
- 주석 부분
	- 이 매크로들은 Unreal Header Tool (UHT)가 파싱하는 메타데이터를 감싸는 역할을 한다.
	- C++ 컴파일러가 코드를 컴파일할 때는 무시된다.

즉, `UPROPERTY()`는 C++ 컴파일러 입장에서는 아무 의미 없는 빈 매크로이고, UHT가 소스 코드를 읽을 때만 의미를 가진다.
- 매크로 정의 부분
	- \#define UPROPERTY(...)라는 이름의 매크로를 정의했는데, 인자를 받아도 아무 코드로 치환되지 않는다.
	- C++ 컴파일러가 볼 때는 그냥 '없음!' 으로 처리된다.

### 정리
- `UPROPERTY()`는 C++ 컴파일러에는 무시되는 깡통 매크로
- 하지만, UHT (Unreal Header Tool)이 소스 코드를 파싱할 때는 중요한 메타데이터로 인식
	- GC
	- 직렬화 
	- 리플렉션
	- 블루프린트 등에 필요한 정보를 생성
- C++ 컴파일러는 위에서 생성된 `.generated.h`에 메타데이터 코드를 포함해서 컴파일
---

## Assertion (어설션) 함수
### 정의
- 프로그램이 실행될 때 특정 조건이 참인지 검사하는 함수
### 목적
- 잘못된 상태가 발생했을 때 즉시 알려주어 디버깅을 쉽게하고, 치명적인 오류를 방지
### 종류
- `assert()` 표준 C/C++
	- `<cassert`헤더에 정의됨
	- 조건이 거짓이면 프로그램을 **강제 종료**하고 오류 메시지를 출력
- Unreal Engine의 `check()` / `ensure()`
	- `check()` : 조건이 거짓이면 **즉시 크래시** 발생 (개발 중 치명적 오류 확인용)
	- `ensure()` : 조건이 거짓이면 경고 메시지 출력 후 계속 실행 (비치명적 오류 확인용)
### 예시
```cpp
void UMyGameInstance::Init()
{
    Super::Init();

    // 반드시 SchoolName이 비어있지 않아야 한다고 가정
    check(!SchoolName.IsEmpty()); // 조건이 거짓이면 크래시 발생

    // 혹은 덜 치명적인 경우
    ensure(!SchoolName.IsEmpty()); // 조건이 거짓이면 경고만 띄우고 실행은 계속
}
```


---







