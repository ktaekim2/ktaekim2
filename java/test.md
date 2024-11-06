# test

## @DisplayName
단순한 테스트의 경우 매서드 이름으로 나타내지만, 복잡한 설명이 필요한 경우 @DisplayName을 사용한다.
```java
@Test
@DisplayName("문자열을 ','로 분리했을 때, 각 요소를 배열로 반환한다")
void splitStringByComma() {
    String input = "1,2";
    String[] result = input.split(",");
    assertThat(result).containsExactly("1", "2");
}

@Test
@DisplayName("문자열 인덱스 초과 시 StringIndexOutOfBoundsException 발생")
void charAtThrowsExceptionWhenIndexIsOutOfBounds() {
    String input = "abc";
    assertThatThrownBy(() -> input.charAt(3))
            .isInstanceOf(StringIndexOutOfBoundsException.class);
}
```

## JUnit 5 Parameterized Test

### 1. ParameterizedTest와 CsvSource 활용
- `@ParameterizedTest`: 반복적인 테스트를 단순화하여 코드를 간결하게 작성할 수 있음.
- **@ValueSource**: 단일 타입을 대상으로 테스트할 때 유용.
- **@CsvSource**: 여러 인자와 기대 결과를 전달할 수 있어 복잡한 테스트에도 적합.

### 2. 학습 중 이해한 점
- JUnit의 `@ParameterizedTest`를 활용하면 테스트 코드 중복을 줄일 수 있어 유지보수에 용이함.
- 특히 `@CsvSource`는 여러 인자와 기대 결과를 한 번에 관리할 수 있어, 다양한 테스트 케이스를 작성할 때 편리함을 느꼈다.


## JUnit 5 - Parameterized Test
- JUnit 5의 `@ParameterizedTest`를 사용해 **다양한 입력 값을 테스트할 수 있는 방법**을 배움.
- **@ValueSource**: 단일 타입의 여러 값을 반복 테스트.
- **@CsvSource**: 여러 인자와 기대 결과를 함께 제공해 테스트.

```java
@ParameterizedTest
@CsvSource({
  "1, true",
  "2, true",
  "3, true",
  "4, false",
  "5, false"
})
void containsTest(int input, boolean expectedResult) {
  assertThat(numbers.contains(input)).isEqualTo(expectedResult);
}
```
