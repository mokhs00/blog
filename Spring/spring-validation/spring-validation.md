# Spring-Validation

# 개요

Spring-validation에 대해서 학습하고 정리한 글입니다.

**해당 글은 spring-boot 2.5.2 에서 진행되었습니다**.

Spring-validation은 controller에서 어노테이션만으로 쉽게 값을 validation할 수 있도록 도와줍니다.

# Gradle

spring validation 사용을 위해서 그래이들 의존성을 추가합니다.

```groovy
dependencies {
		...
    implementation 'org.springframework.boot:spring-boot-starter-validation'
		...
}
```

# 예시

spring-validation은 다음과 같이 각각에 맞는 **Annotation**을 붙여줌으로써 값을 검증할 수 있습니다.

먼저 아래 예시 코드를 보고 어떻게 사용하는지 감을 잡아보세요.
작명이 잘 되어있어서 어느정도 유추할 수 있을 겁니다!

이후 내용에서 관련 **Annotation**에 대해 다룹니다.

```java
// Api.java

@GetMapping
public User get(@RequestParam @NotBlank String name,
                @RequestParam @Min(value = 1, message = "age는 1이상이어야 합니다.") Integer age) {
    User user = new User();
    user.setName(name);
    user.setAge(age);

    return user;
}

@PostMapping
public User post(@RequestBody @Valid UserDto user) {
    System.out.println(user);

    return user;
}

// UserDto.java

public class UserDto {

    @Size(min = 1, max = 10)
    private String name;

    @NotNull
    @Min(1)
    private Integer age;

		public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

}
```

# 관련 Annotation

- `@Valid`

해당 object에 validation을 하겠다는 표시

- `@NotNull`

null 불가능

- `@NotEmpty`

null, ""(공백) 불가능

- `@NotBlank`

null, "", " " 불가능

- `@Pattern`

정규식을 이용해서 검증 가능

- `@Size`

문자 길이 검증, 숫자 type은 불가능

**ex) @Size(min = 0, max = 11)**

- `@Max`

최대값 검증

- `@Min`

최소값 검증

- `@AssertTrue` / `@AssertFalse`

별도의 로직으로 검증 가능 → 조금 아래에서 살펴봅니다.

- `@Past`

과거 날짜 (오늘 이전)

- `@PastOrPresent`

오늘이거나 과거

- `@Future`

미래 날짜 (오늘 이후)

- `@FutureOfPresent`

오늘이거나 미래

# 검증 실패 시 Message

Validation에 실패하면 message를 통해서 검증 실패 메시지를 출력할 수 있습니다.

검증 실패 시 아래와 같이 server error가 발생하고 message가 출력되는 것을 알 수 있습니다.

![Spring-Validation/Untitled.png](Spring-Validation/Untitled.png)

## Message 변경하기

Validation 관련 Annotation을 살펴보면

아래와 같이 message라는 멤버를 가진 걸 알 수 있습니다.

이를 한 번 커스텀 해볼게요

```java
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
@Repeatable(List.class)
@Documented
@Constraint(validatedBy = { })
public @interface Min {

	String message() default "{javax.validation.constraints.Min.message}";

	Class<?>[] groups() default { };

	Class<? extends Payload>[] payload() default { };

	/**
	 * @return value the element must be higher or equal to
	 */
	long value();
```

아래와 같이 간단한 api를 만들고 `@Min` 을 이용해서 age가 19 미만일 때를 검증하고 error 발생 시 `message` 를 다음과 같이 설정해 보겠습니다.

```java
@GetMapping
public User get(@RequestParam @NotBlank String name,
                @RequestParam @Min(value = 19, message = "해당 컨턴츠는 19세 이상 접근 가능합니다.")
												Integer age) {
    User user = new User();
    user.setName(name);
    user.setAge(age);

    return user;
}
```

이제 해당 uri로 접근하면?

다음과 같이 message가 변경된 것을 확인할 수 있습니다

![Spring-Validation/Untitled%201.png](Spring-Validation/Untitled%201.png)

# 검증 로직 커스텀하기

원하는 검증 로직을 설정할 수도 있습니다.

그 방법은 다음과 같습니다.

- `@AssertTrue`와 `@AssertFalse` 이용하기

  `@AsserFalse`는 여기선 다루지 않습니다.
  간단하게 @AssertTrue와 반대로 return값을 주면 됩니다.
  return값에 대한 자세한 내용은 아래 **@AssertTrue**를 확인해주세요

- `Annotation` + `Validator` 정의하기
