# Stream
## 개념
- 데이터 처리용 API
- 배열, 컬렉션 데이터를 선언적, 함수형 방식으로 처리 가능
- 원본 데이터를 변경하지 않고 가공 처리

## 특징
- 데이터를 한 줄로 처리(map, filter, sorted 등)
- 원본 데이터 변경 없음
- 중간 연산 + 최종 연산으로 구성
- 지연처리(Lazy Evaluation) 가능 -> 최종 연산 시 실제 동작

## 예제
### 기본 방식

```java
List<String> names = Arrays.asList("홍길동", "김철수", "이영희");
for (String name : names) {
    if (name.startsWith("김")) {
        System.out.println(name);
    }
}
```

### Stream 사용

```java
names.stream()
     .filter(name -> name.startsWith("김"))
     .forEach(System.out::println);
```

# Optional(옵셔널)

## 개념
- null을 안전하게 처리하기 위한 Wrapper 클래스
- null 체크를 간단하게 만들고 NPE(NullPointerException) 예방

## 특징
- Optional<T> 형태로 값이 있을 수도 있고 없을 수도 있음
- 메서드 체이닝으로 null-safe 코드 작성
- orElse, orElseGet, orElseThrow 등을 활용

## 예제
### 기본 방식

```java
String name = getName();
if (name != null) {
    System.out.println(name.toUpperCase());
}
```

### Optional 사용

```java
Optional<String> name = Optional.ofNullable(getName());

name.ifPresent(n -> System.out.println(n.toUpperCase()));  // 값 있을 때만 실행
String result = name.orElse("이름없음");                   // 값 없으면 기본값
```

## 핵심 메서드
|메서드|설명|
|---|----|
|of(value)|null 아닌 값 감싸기|
|ofNullable(value)|null 가능 값 감싸기|
|isPresent()|값이 있으면 true|
|ifPresent(Consumer)|값이 있으면 실행|
|orElse(default)|값이 없으면 기본값 반환|
|orElseThrow()|값이 없으면 예외 발생|
