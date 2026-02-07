---
title: '[이득우의 UE5 C++] 8. Composition'
published: 2026-02-07
description: 'CreateDefaultSubobject로 SubObject를 생성, UE5의 TObjectPtr활용, Enum+리플렉션으로 메타데이터를 다루는 방법을 알아보자'
image: '' 
tags: [Unreal Engine, C++, OOP]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.   
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   
>
> # 강의 목표

- 언리얼 C++의 컴포지션 기법을 사용해 오브젝트의 포함 관계를 설계한다.
- 언리얼 C++이 제공하는 열거형(Enum) 타입의 선언과 메타데이터 활용 방법을 학습한다.
    

# 컴포지션 (Composition)
객체 지향 설계에서 상속이 가진 `Is-A` 관계에만 의존해서는 복잡한 기능을 설계하고 유지 보수하기 어렵다.

![[Pasted image 20260207083516.png]]

- 컴포지션은 `Has-A` 관계를 구현하는 설계 방법이다. 복합적인 기능을 가진 거대한 클래스를 상속으로만 처리하지 않고, 필요한 기능을 가진 부품(오브젝트)을 조립해서 만든다.
    

![[Pasted image 20260207083546.png]]

- 하나의 언리얼 오브젝트에는 항상 클래스 기본 오브젝트인 `CDO(Class Default Object)`가 존재한다.
- 언리얼 오브젝트에 다른 언리얼 오브젝트를 조합(Composition)할 때, CDO 생성 시점에 따라 두 가지 선택지가 존재한다.
    - `생성자` CDO 생성 시점에 미리 산하 오브젝트를 생성해 조합한다. (`CreateDefaultSubobject` API 활용)
    - `런타임` CDO에는 빈 포인터만 두고, 런타임(게임 실행 중)에 필요할 때 오브젝트를 생성해 조합한다. (`NewObject` API 활용)
        
- 언리얼 오브젝트 간의 컴포지션 관계 용어
    - `SubObject` 내가 소유한 산하 언리얼 오브젝트 (그림의 보라 상자 → 초록 상자)
    - `Outer` 나를 소유한 상위 언리얼 오브젝트 (그림의 초록 상자 → 보라 상자)
        

# 실습

## 1. Card 클래스 생성 (Enum 활용)

언리얼 엔진의 리플렉션 기능을 활용하기 위해 `UENUM` 매크로와 `UMETA`를 사용한다.
**Card.h**
```cpp
UENUM()
enum class ECardType : uint8
{
	Student = 1 UMETA(DisplayName = "For Student"),
	Teacher UMETA(DisplayName = "For Teacher"),
	Staff UMETA(DisplayName = "For Staff"),
	Invalid
};
```
- `UCard` 클래스 생성 및 `ECardType` 멤버 변수 추가
- CardType에 대한 Getter/Setter 구현
    

## 2. Person has-a Card (컴포지션 구현)

`Person` 클래스가 `Card` 객체를 소유하도록 구현한다.

**Person.h**
```cpp
UCLASS()
class UNREALCOMPOSITION_API UPerson : public UObject
{
	GENERATED_BODY()

public:
	UCard* GetCard() const { return Card; }
	void SetCard(UCard* InCard) { Card = InCard; }

protected:
	// Person has-a Card
	// 언리얼 5부터는 원시 포인터 대신 TObjectPtr<T> 사용 권장
	UPROPERTY()
	TObjectPtr<UCard> Card; 
	
	// class UCard* Card; // 언리얼 4 방식
};
```

- `TObjectPtr` 언리얼 5로 넘어오면서 `UPROPERTY`로 지정된 멤버 변수는 원시 포인터 대신 `TObjectPtr` 래퍼 클래스를 사용하는 것으로 변경되었다. (에디터 빌드 시 동적 로딩 등 이점 제공, 런타임에서는 원시 포인터와 동일)
    

## 3. GameInstance에서 활용

### 컴포지션 객체 접근

**MyGameInstance.cpp**
```cpp
const UCard* OwnCard = Person->GetCard();
if (OwnCard)
{
	ECardType CardType = OwnCard->GetCardType();
	UE_LOG(LogTemp, Log, TEXT("%s님이 소유한 카드 종류 %d"), *Person->GetName(), (int32)CardType);
}
```

### Enum 메타데이터 접근 (리플렉션)

`FindObject`를 사용해 열거형의 메타 정보(`UMETA`로 지정한 DisplayName)를 가져올 수 있다.

**MyGameInstance.cpp**
```cpp
const UEnum* CardEnumType = FindObject<UEnum>(nullptr, TEXT("/Script/UnrealComposition.ECardType"));
if (CardEnumType)
{
	// Enum의 Value를 통해 DisplayName(MetaData)을 문자열로 추출
	FString CardMetaData = CardEnumType->GetDisplayNameTextByValue((int64)CardType).ToString();
	UE_LOG(LogTemp, Log, TEXT("%s님이 소유한 카드 종류: %s"), *Person->GetName(), *CardMetaData);
}
```

### 출력 결과
```
LogTemp: ===============================
LogTemp: 학생님이 소유한 카드 종류 1
LogTemp: 선생님님이 소유한 카드 종류 For Teacher
LogTemp: 직원님이 소유한 카드 종류 For Staff
```

# 정리

- 언리얼 C++는 객체 간의 포함 관계(Has-A)를 구현하는 컴포지션 패턴을 지원한다.
- 생성자 단계에서 `CreateDefaultSubobject`를 사용해 필수적인 SubObject를 미리 생성하고 조립할 수 있다.
	- `SubObject` 내가 소유한 하위 오브젝트
	- `Outer` 나를 소유한 상위 오브젝트
- `TObjectPtr`는 UE5부터 도입된 표준 객체 포인터 래퍼 클래스이다. (헤더에서 `TObjectPtr<UCard> Card;` 형태로 선언하여 컴포지션 구현)
- `enum class`와 `UMETA`매크로를 조합하면 열거형의 메타 정보를 런타임에 활용 가능하다.
