---
title: '번외 1. 언리얼 클래스 문법과 필수 매크로'
published: 2026-02-02
description: '언리얼 C++에서 클래스 생성시에 보이는 UCLASS()와 UNREALSTRING_API, GENERATED_BODY()'
image: ''
tags: [Unreal Engine, C++]
category: 'Unreal Engine'
draft: false 
lang: ''
---
# 번외 1. 언리얼 클래스 문법과 _API 매크로
> 강의 외적인 의문을 정리한 번외 파트입니다.      
> 옵시디언에 정리한 마크다운 문서라 블로그 마크다운 양식에 일부 맞지 않을 수 있습니다. 

```cpp
UCLASS()
class UNREALSTRING_API UMyGameInstance : public UGameInstance
{
    GENERATED_BODY()
public:
    virtual void Init() override;
};
```
위와 같은 언리얼 C++을 처음 접하면 기존 C++과 다른 클래스 문법이 몇가지 보인다. 
- UCLASS()
- UNREALSTRING_API
- GENERATED_BODY()
--- 

# 1. 기존 C++ 클래스 문법
C++에서 클래스는 아래와 같이 선언한다.
```cpp
class 클래스명 : 접근제한자 부모클래스명
{
    // 멤버 변수와 함수
};
```
예시 
```cpp
class MyClass : public BaseClass
{
    void Hello();
};
```
---
# 2. UCLASS() 매크로
```cpp
UCLASS()
class ... : ...
{
}
```
- 언리얼 엔진은 자체 오브젝트 시스템을 가지고 있다.
- `UCLASS()` 매크로는 아래 클래스가 언리얼 리플렉션 시스템에 등록된다는 표시
- 즉, `블루프린트`에서 사용하거나 `직렬화` 같은 기능을 쓸 수 있게된다.
---
# 3. UNREALSTRING_API 매크로
`UNREALSTRING_API`같은 매크로는 모듈 간 심볼 `export` `import`를 처리하기 위한 장치이다.
```cpp
class UNREALSTRING_API UMyGameInstance : public UGameInstance
```
위 코드에 `UNREALSTRING_API`를 VS에서 추적해보면 아래와 같이 잡힌다.
```cpp
// Windows (MSVC)
#define UNREALSTRING_API DLLEXPORT
// ->
#define DLLEXPORT __declspec(dllexport)
```
- 언리얼 프로젝트는 여러 모듈로 나뉘어 있고, 각 모듈은 `DLL`로 빌드된다.
- 다른 모듈에서 정의한 클래스를 사용하려면 `export` `import` 가 필요하다.
- 언리얼은 멀티 플랫폼을 지원하므로,  `XXXXX_API` 매크로를 두고 환경에 맞게 아래와 같이 치환한다.
	- MSVC :  `__declspec(dllexport)` / `__declspec(dllimport)`
	- GCC/Clang : `__attribute__((visibility("default")))` 
---
# 4. GENERATED_BODY() 매크로
```cpp
UCLASS()
class UNREALSTRING_API UMyGameInstance : public UGameInstance
{
    GENERATED_BODY()
};
```
- `GENERATED_BODY()`는 엔진이 자동으로 필요한 내부 코드를 삽입하게 해주는 매크로
	- 리플렉션, 직렬화, 블루프린트 지원 등..
- 클래스마다 고유한 함수/메타데이터를 생성하는 코드가 들어간다.
	- `StaticRegisterNativesUMyGameInstance()`
	- `DECLARE_CLASS(...)`
	- `DECLARE_SERIALIZER(...)`
## 리플렉션
리플렉션(Reflection)은 프로그램이 자기 자신을 메타 데이터로 인식하고 다룰 수 있는 기능
- 언리얼에서는 리플렉션을 통해 다음이 가능하다.
	- 블루프린트 노출
	- 직렬화(저장/로드)
	- 에디터에서 프로퍼티 수정
	- 네트워킹/레플리케이션
---



# 5. 결론
언리얼 클래스 선언부는 아래와 같이 이해하면 된다
```cpp
UCLASS()
class [모듈 매크로] [클래스명] : [접근제한자] [상속클래스명]
{
    GENERATED_BODY()
}
```

매크로는 정리하면 아래와 같다.
- `UCLASS()` : 언리얼 리플렉션 시스템 등록용 매크로
- `UNREALSTRING_API` : 모듈 간 심볼 export/import 처리용 매크로 (멀티 플랫폼 호환성)
- `GENERATED_BODY()` : 엔진이 자동으로 필요한 내부 코드를 삽입하는 매크로
