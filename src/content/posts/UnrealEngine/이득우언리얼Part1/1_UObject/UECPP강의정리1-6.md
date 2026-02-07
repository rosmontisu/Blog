---
title: '[이득우의 UE5 C++] 6. UObject Reflection 2'
published: 2026-02-05
description: '언리얼 오브젝트의 리플렉션 시스템으로 프로퍼티와 함수를 접근하고 수정해보자.'
image: '' 
tags: [Unreal Engine, C++]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.   
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   


# 강의 목표
- 리플렉션 시스템을 이용해 언리얼 오브젝트를 다루는 방법
	- 속성/함수 검색 및 호출/변경
	- 접근 지시자 우회

---
# 1. FProperty로 속성 접근/변경


MyGameInstance.cpp
```cpp
Super::Init();

// 언리얼 오브젝트 생성 - NewObject<T>()
UStudent* Student = NewObject<UStudent>(this);
UTeacher* Teacher = NewObject<UTeacher>(this);

FString CurrentTeacherName;
FString NewTeacherName(TEXT("새이름"));

FProperty* NameProp = UTeacher::StaticClass()->FindPropertyByName(TEXT("Name"));
if (NameProp)
{
	// 접근
	NameProp->GetValue_InContainer(Teacher, &CurrentTeacherName);
	UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *CurrentTeacherName);
	
	// 변경
	NameProp->SetValue_InContainer(Teacher, &NewTeacherName);
	UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *Teacher->GetName());
}
```

Person.h
```cpp
protected:
	UPROPERTY()
	FString Name; // 선생(학생) 이름
```

Output Log
```bash
LogTemp: 현재 선생님 이름 이선생
LogTemp: 현재 선생님 이름 새이름
```


- `FProperty`의 API를 이용해 프로퍼티의 접근 및 변경이 가능하다.
	- `GetValue_InContainer()`
	- `SetValue_InContainer()`
- 이때, Protected로 보호받고 있는 프로퍼티 값인 선생님 이름을 Reflection을 이용해 그대로 출력 가능


---

# 2. UFunction으로 함수 사용

MyGameInstance.cpp
```cpp
Student->DoLesson();
UFunction* DoLessonFunc = Teacher->GetClass()->FindFunctionByName(TEXT("DoLesson"));
if (DoLessonFunc)
{
	Teacher->ProcessEvent(DoLessonFunc, nullptr); // 인자가 없는 함수라 null 전달
}
```
Output Log
```bash
LogTemp: 새이름님이 수업에 참여합니다. // Person.cpp
LogTemp: 3년차 선생님 새이름님이 수업을 강의합니다. // Teacher.cpp
```

- `UFunction`의 API를 이용해 함수 사용이 가능하다.










---
# 정리
- Reflection System 활용
	- 언리얼 오브젝트의 특정 속성과 함수를 이름으로 검색 가능
	- 접근 지시자와 무관하게 값을 설정
	- 언리얼 오브젝트의 함수 호출
- 언리얼 엔진의 프레임워크는 리플렉션을 활용해서 구축되어 있으므로, 이해가 필요
- (주의) 컴파일러의 타입 체크 지원을 받지 못하므로, 런타임때 검증 로직을 직접 작성해야 함

---

# 번외
## 타입 안정성 개선 : CastField\<T\>()
### 문제
- `FindPropertyByName`은 단순히 메모리 오프셋 정보만 가져온다.
- 타입 일치 여부를 보장하지 않아, 메모리 오염(Memory Corruption) 위험이 존재

### 개선
- `CastField<T>`를 사용해 런타임 타입 검사(RTTI)를 수행

```cpp
FProperty* NameProp = UTeacher::StaticClass()->FindPropertyByName(TEXT("Name"));
if (NameProp)
{
	// FStrProperty인지 확인 (int, float 등 다른 타입일 경우 크래시 방지를 위해서)
	FStrProperty* StrProp = CastField<FStrProperty>(NameProp);
	if (StrProp)
	{
		StrProp->SetValue_InContainer(Teacher, &NewTeacherName);
	}
}
```

---

## 상속시 주의점 : generated.h
```cpp
#include "Student.generated.h"
#include "Person.h" // generated.h 아래에 있으면 빌드 오류 발생
```
- 언리얼에서 다른 헤더를 상속 받을때 항상 `generated.h`파일이 가장 밑에 include 되도록 주의하자.
- UHT가 `Student.h`와 같은 C++ 헤더를 파싱하여, 리플렉션 정보가 담긴 `.generated.h`를 만들기 때문에, 당연히 이 파일이 마지막에 있어야 `GENERATED_BODY` 매크로가 정상 작동한다.

---

## GetValue_InContainer vs C++ 직접 접근
### 문제
- 리플렉션은 유연하지만 느리다. 매 프레임 호출되는 `Tick`같은 곳에서는 지양해야한다.
	- `FindPropertyByName`검색은 당연히 직접 접근하는  `Teacher->SetName()`보다 느리다.
### 써야 한다면 캐싱해서 쓰자
- `BeginPlay`에서 `FindPropertyByName`으로 `FProperty*`를 찾아 변수에 저장(캐싱)해두고
- `Tick`에서는 저장된 포인터로 `SetValue_InContainer`만 수행

---
---
---
# 실습 코드 구조
- Student와 Teacher는 Person을 상속받음

MyGameInstance.h
```cpp
#pragma once
#include "CoreMinimal.h"
#include "Engine/GameInstance.h"
#include "MyGameInstance.generated.h"
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
#include "MyGameInstance.h" // 현재 언리얼 오브젝트 헤더가 가장 위에 있어야 함 (UHT 규칙)
#include "Student.h"
#include "Teacher.h"

UMyGameInstance::UMyGameInstance()
{
	SchoolName = TEXT("기본 학교"); // 이 기본값은 CDO 템플릿 객체에 저장
}

void UMyGameInstance::Init()
{
	Super::Init();

	// 언리얼 오브젝트 생성 - NewObject<T>()
	UStudent* Student = NewObject<UStudent>();
	UTeacher* Teacher = NewObject<UTeacher>();

	Student->SetName(TEXT("학생1"));
	UE_LOG(LogTemp, Log, TEXT("새로운 학생 이름 %s"), *Student->GetName());

	// reflection을 이용해 속성 접근/변경
	FString CurrentTeacherName;
	FString NewTeacherName(TEXT("새이름"));
	FProperty* NameProp = UTeacher::StaticClass()->FindPropertyByName(TEXT("Name"));
	if (NameProp)
	{
		NameProp->GetValue_InContainer(Teacher, &CurrentTeacherName);
		UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *CurrentTeacherName);

		// FProperty API는 주소(void*)를 요구하며, 대상 객체 메모리에 값을 복사한다.
		// 단, FString은 내부적으로 버퍼 공유 + 참조 카운트 증가 방식이라 
		// 실제 문자열 데이터는 즉시 복제되지 않는다 (Copy-On-Write 발생 X → O(1), 발생 시 → O(n))
		NameProp->SetValue_InContainer(Teacher, &NewTeacherName);
		UE_LOG(LogTemp, Log, TEXT("현재 선생님 이름 %s"), *Teacher->GetName());
	}

	UE_LOG(LogTemp, Log, TEXT("===================="));

	// reflection을 이용해 함수 사용
	Student->DoLesson();

	UFunction* DoLessonFunc = Teacher->GetClass()->FindFunctionByName(TEXT("DoLesson"));
	if (DoLessonFunc)
	{
		Teacher->ProcessEvent(DoLessonFunc, nullptr); // 인자가 없는 함수라 null 전달
	}
}
```

Person.h
```cpp
#pragma once
#include "CoreMinimal.h"
#include "UObject/Object.h"
#include "Person.generated.h"
UCLASS()
class OBJECTREFLECTION_API UPerson : public UObject
{
	GENERATED_BODY()
public:
	UPerson();

	UFUNCTION()
	virtual void DoLesson();

	const FString& GetName() const;
	void SetName(const FString& InName);

protected:
	UPROPERTY()
	FString Name;

	UPROPERTY()
	int32 Year;

private:
};
```


