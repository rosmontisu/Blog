---
title: '[UE5 / Part1] 12. UE GC & FGCObject'
published: 2026-02-11
description: '언리얼 GC의 동작 원리와 주의점 및 C++ 클래스를 보호하는 FGCObject를 알아보자'
image: '' 
tags: [Unreal Engine, C++, GC]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.   
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   
>


# C++ 언어 메모리 관리의 문제점과 GC
- 잘못된 포인터 사용 예시
    - 메모리 누수 (Leak) : new <-> delete 짝을 못 맞춤. 힙에 메모리가 그대로 남음
    - 허상 포인터 (Dangling Pointer) : 이미 해제하여 무효화된 오브젝트의 주소를 가리키는 포인터
    - 와일드 포인터 (Wild Pointer) : 값이 초기화되지 않아 엉뚱한 주소를 가리키는 포인터
        

## GC 시스템
- 동적으로 생성된 모든 오브젝트 정보를 모아둔 저장소를 사용해 사용되지 않는 메모리를 추적
- 마크-스윕(Mark-Sweep) 방식
    - GCCycle 기본값 60초
- 병렬처리, 클러스터링 - 성능 향상을 위함
    

## GC를 위한 객체 저장소
- GUObjectArray: 관리되는 모든 UObject 정보를 저장하는 전역 변수
    - <Object, Flag> 구조로 관리
        
- 각 요소에는 플래그(Flag)가 설정
    - Garbage Flag : 참조가 없어 회수 예정
    - RootSet Flag : 참조가 없어도 회수X (특별한 오브젝트)
        

## GC의 메모리 회수
- 지정된 시간에 따라 주기적으로 메모리 회수 (기본 값 60초)
- Garbage Flag로 설정된 오브젝트를 파악하고 메모리를 안전하게 회수
- Garbage Flag는 수동으로 설정 X, 시스템이 알아서 설정함
    
    한 번 생성된 언리얼 오브젝트는 바로 삭제가 불가능함. `Delete` 키워드로 삭제하는 게 아님, 레퍼런스 정보를 없애는 것으로 GC가 자동으로 메모리를 회수하도록 설정
    

## RootSet Flag 설정
- `AddToRoot` 함수를 호출해 루트셋 플래그를 설정하면 최초 탐색 목록으로 설정됨
    - `RootSet`으로 설정된 언리얼 오브젝트는 메모리 회수로부터 보호받음
- `RemoveFromRoot` 함수를 호출해 루트셋 플래그를 제거할 수 있음
    
_콘텐츠를 만들 때 권장하는 방식은 아님_
    

## 언리얼 오브젝트를 통한 포인터 문제의 해결
- 메모리 누수 문제
    - GC를 통해 자동 해결
    - _C++ 오브젝트는 직접 신경 써야 함 (or 스마트 포인터 사용)_
        
- 댕글링 포인터 문제
    - 탐지를 위한 함수 제공 `::IsValid()`
    - _C++ 오브젝트는 직접 신경 써야 함 (or 스마트 포인터 사용)_
        
- 와일드 포인터 문제
    - UPROPERTY 속성을 지정하면 자동으로 `nullptr`로 초기화해 줌
    - _C++ 오브젝트의 포인터는 직접 `nullptr`로 초기화 필요 (or 스마트 포인터 사용)_
        

## 회수되지 않는 언리얼 오브젝트
- 언리얼 엔진 방식으로 참조를 설정한 언리얼 오브젝트
    - UPROPERTY로 참조된 언리얼 오브젝트 (대부분 이걸 사용)
    - AddReferencedObjects 함수를 통해 참조를 설정한 오브젝트
        
- RootSet으로 지정된 언리얼 오브젝트
    
오브젝트 포인터는 가급적 `UPROPERTY`로 선언하고, 메모리는 가비지 컬렉터가 자동으로 관리하도록 위임한다.
    

## 일반 클래스에서 언리얼 오브젝트를 관리하는 경우?
- UPROPERTY를 사용하지 못하는 일반 C++ 클래스가 언리얼 오브젝트를 관리해야 하는 경우
- `FGCObject` 클래스를 상속받은 후 `AddReferencedObjects` 함수를 구현한다
- 함수 구현부에서 관리할 언리얼 오브젝트를 추가해 줌
    
    아래 실습에서 직접 사용해보자.
    

## 언리얼 오브젝트의 관리 원칙
- 생성된 언리얼 오브젝트를 유지하기 위해 레퍼런스 참조 방법을 설계할 것
    - 언리얼 오브젝트 내의 언리얼 오브젝트 : UPROPERTY 사용
    - 일반 C++ 내의 언리얼 오브젝트 : FGCObject의 상속 후 구현
- 생성된 언리얼 오브젝트는 강제로 지우려 하지 말 것
    - 참조를 끊는다는 생각으로 설계
    - GC에게 회수를 재촉 가능 (ForceGarbageCollection 함수)
    - 콘텐츠 제작에서 Destroy 함수를 사용할 수 있으나, 결국 내부 동작은 똑같음 (GC 위임)
        

# 실습

## GC 테스트 환경 제작
- 프로젝트 설정에서 GCCycle 시간을 3초로 단축 설정
- 새로운 GameInstance의 두 함수를 오버라이드
    - Init: 어플리케이션이 초기화될 때 호출
    - Shutdown: 어플리케이션이 종료될 때 호출
        
- 테스트 시나리오
    - 플레이 버튼을 누를 때 Init 함수에서 오브젝트를 생성
    - 3초 이상 대기해 GC 발동
    - 플레이 중지를 눌러 Shutdown 함수에서 생성한 오브젝트의 유효성을 확인
        

## 실습 1. UObject의 UPROPERTY() 유무 차이

**MyGameInstance.h**



```cpp
private:
	TObjectPtr<class UStudent> NonPropStudent;
	
	UPROPERTY()
	TObjectPtr<class UStudent> PropStudent;
};
```

**MyGameInstance.cpp**



```cpp
NonPropStudent = NewObject<UStudent>(); // UPROPERTY() X

PropStudent = NewObject<UStudent>(); // UPROPERTY() O
```

댕글링 포인터 문제 발생. (UPROPERTY가 없어 GC가 정리)



```
LogTemp: NonPropStudent - 널 포인터가 아닌 언리얼 오브젝트
LogTemp: NonPropStudent - 유효하지 않은 언리얼 오브젝트

LogTemp: PropStudent - 널 포인터가 아닌 언리얼 오브젝트
LogTemp: PropStudent - 유효한 언리얼 오브젝트
```

---

## 실습 2. TArray<TPtr*> UPROPERTY() 유무 차이

**MyGameInstance.h**


```cpp
TArray<TObjectPtr<class UStudent>> NonPropStudents;

UPROPERTY()
TArray<TObjectPtr<class UStudent>> PropStudents;
```

**MyGameInstance.cpp**


```cpp
// 실습 2
NonPropStudents.Add(NewObject<UStudent>());

PropStudents.Add(NewObject<UStudent>());
```

UPROPERTY()가 없을 경우 배열 내부에 Ptr에 댕글링 포인터 발생 (GC 정리)


```
LogTemp: NonPropStudents[0] - 널 포인터가 아닌 언리얼 오브젝트
LogTemp: NonPropStudents[0] - 유효하지 않은 언리얼 오브젝트

LogTemp: PropStudents[0] - 널 포인터가 아닌 언리얼 오브젝트
LogTemp: PropStudents[0] - 유효한 언리얼 오브젝트
```

---

## 실습 3. 일반 C++ 객체 안에 있는 UObject

**MyGameInstance.h**


```cpp
class FStudentManager* StudentManager = nullptr;
```

**MyGameInstance.cpp**


```cpp
// 실습 3
// Manager는 일반 C++ 객체
// 안에 있는 UObject를 관리할 수 있는 능력이 전혀 없음
// 이 클래스는 UProperty 같은 걸 쓸 수 없기 때문에
// GC가 발동되면 안에 있는 UObject가 가비지 컬렉션 대상이 되어버림
StudentManager = new FStudentManager(NewObject<UStudent>());
```

댕글링 포인터 발생 (C++ 내부 객체를 GC가 정리)


```
LogTemp: StudentInManager - 널 포인터가 아닌 언리얼 오브젝트
LogTemp: StudentInManager - 유효하지 않은 언리얼 오브젝트
```

## 실습 4. FGCObject로 C++ 객체 내부 보호

`FGCObject`를 상속받고 다음 2개의 함수를 구현해야 한다.

**GCObject.h**


```cpp
virtual void AddReferencedObjects( FReferenceCollector& Collector ) = 0;

/** Overload this method to report a name for your referencer */
virtual FString GetReferencerName() const = 0;
```

아래와 같이 먼저 상속을 받은 후

**StudentManager.h**


```cpp
class UNREALMEMORY_API FStudentManager : public FGCObject
{
public:
	FStudentManager(class UStudent* InStudent) : SafeStudent(InStudent) {}
	
	// 함수 오버라이드 1 
	virtual void AddReferencedObjects(FReferenceCollector& Collector) override;
	
	// 함수 오버라이드 2
	virtual FString GetReferencerName() const override
	{
		return TEXT("FStudentManager");
	}
	
private:
	class UStudent* SafeStudent = nullptr;
};
```

`AddReferencedObjects`를 마저 구현한다: `AddReferencedObject(UStudent*)`

**StudentManager.cpp**


```cpp
#include "StudentManager.h"
#include "Student.h"

void FStudentManager::AddReferencedObjects(FReferenceCollector& Collector)
{
	if (SafeStudent->IsValidLowLevel())
	{
		Collector.AddReferencedObject(SafeStudent);
	}
}
```

# 정리
## 1. C++ 메모리 관리의 한계와 GC의 필요성
- **문제점:** C++의 수동 메모리 관리는 누수(Leak), 댕글링 포인터(Dangling), 와일드 포인터(Wild) 등의 치명적인 오류를 유발하기 쉽다.
- **해결책:** 언리얼 엔진은 가비지 컬렉션(GC) 시스템을 도입하여 사용되지 않는 메모리를 자동으로 회수하고, 포인터의 유효성을 관리한다.
    

## 2. GC(Garbage Collection)의 동작 원리
- **Mark-Sweep 방식:** 지정된 주기(기본 60초)마다 어플리케이션을 일시 정지하고 메모리를 검사한다.
- **RootSet:** GC의 시작점. 여기에 등록된 객체와, 그 객체가 참조하는 모든 객체는 '사용 중'으로 간주하여 회수하지 않는다.
- **회수 대상:** RootSet에서 도달할 수 없는(Unreachable) 객체는 가비지(Garbage)로 간주하여 메모리를 해제한다.
    

## 3. 언리얼 오브젝트(UObject) 보호 방법

GC가 멀쩡한 오브젝트를 회수해가지 않도록 '참조'를 명확히 해야 한다.
1. **언리얼 오브젝트 내부:** 멤버 변수 포인터에 반드시 `UPROPERTY()` 매크로를 붙인다. (가장 권장됨)
2. **일반 C++ 클래스 내부:** `FGCObject`를 상속받고 `AddReferencedObjects` 함수를 오버라이드하여 참조를 알린다.
3. **특수 상황:** `AddToRoot` 함수로 강제 보호할 수 있으나, 관리가 어려워 권장하지 않는다.
    

## 4. 안전한 언리얼 코딩 표준
- **선언:** `UObject`를 가리키는 모든 포인터는 `TObjectPtr<T>`와 `UPROPERTY()`를 조합하여 선언한다.
    - 자동 초기화(nullptr), 댕글링 포인터 감지, 참조 카운팅이 자동으로 처리된다.
- **생성:** `new` 대신 `NewObject<T>()`를 사용한다.
- **소멸:** `delete`를 절대 직접 호출하지 않는다.
    - 필요 없다면 `nullptr`를 대입해 참조를 끊거나, `ConditionalBeginDestroy()` 등을 사용해 GC에게 처리를 위임한다.
- **검증:** 포인터 사용 전 `IsValid()` 또는 `::IsValid()`로 유효성을 검증하는 습관을 들인다.


---
## 실제로 코딩할때..
### 1. UPROPERTY() 필수
- `UObject` 상속 클래스의 멤버 변수 포인터를 선언할 때 `UPROPERTY()` 매크로를 잊지 말자.
    - 이걸 빼먹으면 GC가 마음대로 메모리를 회수해간다.
        
### 2. NewObject\<T\>() 사용
- `UObject`를 생성할 때 습관적으로 C++의 `new`를 쓰지 말고, 반드시 `NewObject<T>()`를 사용하자.
    - `new`로 만든 `UObject`는 GC의 보호를 전혀 받지 못한다.
        

### 3. IsValid() 습관화
- 포인터 변수에 접근(`->`)하기 전, `IsValid()` 혹은 `::IsValid()`로 유효성을 체크하는 습관을 가지자.

### 4. 일반 클래스는 조심 (FGCObject)
- 일반 C++ 클래스(구조체, 매니저 등)에 `UObject` 포인터를 멤버로 넣어야 한다면 `FGCObject` 상속을 고려하자.
    - 일반 클래스는 `UPROPERTY`를 못 쓴다. 그냥 두면 댕글링 포인터가 되어버린다.
        

### 5. Destroy()의 GC 시차
- `Actor->Destroy()`를 호출했다고 해서 그 즉시 메모리가 `null`이 되는 게 아님을 기억하자.
    - GC가 돌기 전까지는 삭제 대기(Pending Kill) 상태로 메모리에 남아있다. 이 시차를 조심하자.



