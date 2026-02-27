---
title: '[UE5/Part1] 3. 기본타입과 문자열'
published: 2026-02-02
description: '언리얼 C++의 Int, TChar, FString, FName과 같은 타입과 문자열 처리 방식을 알아보자'
image: '' 
tags: [Unreal Engine, C++]
category: 'Unreal Engine'
draft: false 
lang: 'ko'
---

> "이득우의 언리얼 프로그래밍 Part1 - 언리얼 C++의 이해" 학습 내용을 정리한 강의 노트입니다.   
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다.   
>
> 이번 노트는 언리얼 표준 컨벤션을 요약 정리한 내용입니다.

# 강의 목표
- 언리얼 환경에서 알아두어야 할 기본 타입과 고려할 점
- 캐릭터 인코딩 시스템에 대한 이해
- 언리얼 C++이 제공하는 다양한 문자열 처리 방법과 내부 구성의 이해

---
# 언리얼 C++ 기본 타입
## int32
- 언리얼은 `int`를 사용하지 않고 `int32`를 사용함
## float & double
- 국제 표준으로 4byte 8byte가 정의되어 있기때문에
- `float` `double` 그대로 사용
## bool
- `bool`은 크기가 명확하지 않음
- 헤더에는 가급적 `uint8` 타입을 사용한다. but) 1byte는 너무 크다.
	- 그래서 Bit Field 오퍼레이터를 사용 : `uint8 bNetTemporary:1;`
	- 일반 `uint8`과의 구분을 위해 b접두사 사용
- Cpp 로직에서는 자유롭게 `bool` 사용

---

# 언리얼 문자 인코딩
## UE 내부 문자열 표현
- `FStrings` : 내부적으로 `TArray<TCHAR>`을 사용해 문자열을 관리하는 클래스
- `TCHAR` : 언리얼 표준 문자 타입
	- Windows 기준으로 `wchar_t`와 매핑되어 `UTF-16`으로 저장
	- 플랫폼에 따라 `UTF-8`로 매핑될 수도 있음
-  `UTF-16` 인코딩 : 윈도우 기준 2byte로 사이즈가 균일
- `TEXT()` 매크로 : 문자열 리터럴을 항상 `TCHAR`배열로 변환
	- `TEXT("unreal")` -> `const TCHAR[] = {'u', 'n', 'r', 'e', 'a', 'l', '\0'}`
	- c언어 스타일로 `\0`이 붙음
- `FString` : `TChar` 배열을 포함하는 헬퍼 클래스
	- 문자열 생성, 비교, 변환 등 다양한 기능 제공
	- 단순 포인터`TCHAR*`보다 안전하고 편리하게 문자열을 다룰 수 있음
## UE에서 TEXT파일 로드 방식
- `UTF-16` 으로 파일을 로드한다.
	- `UnMisc.cpp` 에 구현된 `appLoadFileToString()` 함수로 처리
	- 위 함수는 파일에 포함된 유니코드 BOM을 인식하여 문자열을 로드
	- 만약, BOM이 없다면 플랫폼별 인코딩 규칙에 따라 해석
		- Windows라면 UTF-16
		- 다른 플랫폼은 UTF-8로 처리될 수 있음
## UE에서 사용되는 TEXT 파일 추천 인코딩
- INI/INT 파일
	- 지역화(INI) 및 설정(INT) 파일은 `UTF-16` 인코딩을 권장
	- Windows환경에 안정적, 다국어 지원에 적합
- 소스 코드 내 문자열 처리
	- C++ 소스 코드 안에서 직접 문자열 하드코딩은 권장 X
	- 문자열은 별도의 INT/INI 파일로 분리하는걸 권장
		- 지역화 & 유지보수에 유리
- 소스 코드에서 한글 사용 시 주의점
	- 소스에 꼭 한글을 사용하겠다면, `UTF-8` 방식 저장 (권장x)

---

# 실습
## 1. TCHAR & TEXT() 매크로
- `TCHAR` 배열은 C 스타일 문자열 포인터 `const TCHAR*` : `%s` 를 충족한다.
	- 내부적으로 `\0`로 끝남 
- 즉, `*`포인터 없이 `TCHAR[]` -> `TCHAR*`로 decay되므로 바로 전달 가능하다.
---

MyGameInstance.h
```cpp
UCLASS()
class UNREALSTRING_API UMyGameInstance : public UGameInstance
{
	GENERATED_BODY()
public:
	virtual void Init() override;

private:
};
```
MyGameInstance.cpp
```cpp
#include "MyGameInstance.h"

void UMyGameInstance::Init()
{
	Super::Init();

	TCHAR LogCharArray[] = TEXT("Hello Unreal");
	UE_LOG(LogTemp, Log, TEXT("%s"), LogCharArray);
}
```
Output Log
```bash
LogTemp: Hello Unreal
```

## 2. FString 클래스
- [FString](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/fstring-in-unreal-engine?application_version=5.7) : 언리얼 문자열 클래스
	- 내부적으로 `TArray<TCHAR>`를 관리한다.
- `FString`자체를 `%s`에 넘길 수 없다. `%s`는 객체가 아니라 포인터`*`를 요구하므로
- `FString`은 `operator*`를 오버로드해서 내부 버퍼의 `const TCHAR*`를 반환하도록 되어있다.
---
MyGameInstance.cpp
```cpp
#include "MyGameInstance.h"

void UMyGameInstance::Init()
{
	Super::Init();

	TCHAR LogCharArray[] = TEXT("Hello Unreal");
	UE_LOG(LogTemp, Log, TEXT("%s"), LogCharArray);

	FString LogCharString = LogCharArray;
	UE_LOG(LogTemp, Log, TEXT("%s"), *LogCharString); // 포인터 필요
}
```
Output Log
```bash
LogTemp: Hello Unreal
```


---
# FString
![[Pasted image 20260201225015.png]]


# FName
- [FName](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/fname-in-unreal-engine?application_version=5.7) : 에셋 관리를 위해 사용되는 문자열 체계 (해시)
	- 대소문자 구분 X
	- 한선 선언되면 변경 X
	- 가볍고 빠름
	- 문자표현용 X, 에셋 키를 지정하는 용도. 빌드시 해시값으로 변환
- [FText](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/ftext-in-unreal-engine) : 다국어 지원을 위한 문자열 관리 체계
	- 일종의 키로 작용함
	- 별도의 문자열 테이블 정보가 추가로 요구됨
	- 게임 빌드 시 자동으로 국가별 언어로 변환
## 구조 
- 사진 첨부 필

---

# 정리
1. 언리얼이 C++ 타입 Int를 사용하지 않는 이유
2. 다양한 캐릭터 인코딩 시스템 이해
3. 언리얼의 문자열 처리
4. FString의 구조와 사용 방법
5. FName의 구조와 사용 방법