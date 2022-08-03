# Validation

## Learning Goals

- Explain why validations are important.
- Define common validations in Spring.

## Introduction

A client can send invalid data to the server. It is important to implement
server side validation to make sure no invalid data is stored in the database.

The `javax.validation` package provides annotations for different types of
validations. We need to add the following dependency to enable validations in
Spring:

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Let’s look at the common annotations we can use in our application.

## Emptiness Check

The following annotations are used for checking `null` or empty values:

- `@NotNull`: Ensures a field is not `null`.
- `@NotEmpty`: Ensures a field is not `null` and its length is greater than 0.
- `@NotBlank`: Ensures a field is not empty even after trimming. For example,
  `" "` will be considered empty.

```java
@NotNull
private int age;

@NotNull
@NotEmpty
private String username;
```

## Length and Value Validation

Let’s look at how to validate the length and the size of integers. We will use
the following annotations:

- `@Size`
- `@Min`
- `@Max`

### `@Size`

This annotation takes `min` and `max` parameters which define the minimum and
maximum length of `String` or the number of elements in a `Collection`.

```java
@Size(min = 1, max = 10)
private List<User> users;
```

### `@Min` and `@Max`

These annotations define the boundaries of numeric values. The annotations take
the `value` parameter but it can also be left out.

```java
@Min(value = 5)
@Max(20)
private int slots;
```

## Pattern Matching

The `@Pattern` annotation allows us to use regular expressions to validate
fields. We define the regular expression in the `regexp` parameter of the
`@Pattern` annotation.

```java
@Pattern(regexp = "[0-9]{4, 6}"
private String pin;
```

The above annotation ensures that the `pin` field must have between 4 to 6
digits and the values between 0 and 9.

## Use Validation in Controller

We have learned how to add validation to our models but we need to add
validation annotations in controllers to validate the request body. We will be
looking at the `@Valid` and `@Validated` annotations.

### `@Valid`

The `@Valid` annotation is added before a controller method parameter. Let’s add
a `@Valid` annotation to the `Member` controller we built in this section.

Open up the `Member` class and add the following annotations to the `name`
field:

```java
// Member.java

@Entity
public class Member {
    @Id
    @GeneratedValue
    private int id;

    @NotNull
    @NotBlank
    private String name;

    private String email;

    // getters and setters
}
```

Now open up the `MemberController` class and modify the `createMember` method:

```java
// MemberController.java

package org.example.springwebdemo.controller;

import org.example.springwebdemo.model.Member;
import org.example.springwebdemo.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api")
public class MemberController {
    @Autowired
    MemberService memberService;

    @PostMapping("/members")
    public ResponseEntity<Member> createMember(@Valid @RequestBody Member member) {
        Member newMember = memberService.createMember(member);
        return ResponseEntity.ok(newMember);
    }

    @GetMapping("/members")
    public List<Member> readMembers() {
        return memberService.getMembers();
    }

    @GetMapping("/members/{memberId}")
    public Member readMember(@PathVariable(value = "memberId") Integer id) {
        return memberService.getMember(id);
    }

    @PutMapping("/members/{memberId}")
    public Member updateMember(@PathVariable(value = "memberId") Integer id, @RequestBody Member memberData) {
        return memberService.updateMember(id, memberData);
    }

    @DeleteMapping("/members/{memberId}")
    public void deleteMember(@PathVariable(value = "memberId") Integer id) {
        memberService.deleteMember(id);
    }
}
```

Notice that we are returning a `ResponseEntity` instead of a `Member` object
like earlier. The `ResponseEntity` allows us to modify response information
(status code, headers) before sending them back to the client.

Now open up Postman and make the following `POST` request to
`http://localhost:8080/api/members`:

```java
{
    "name": "",
    "email": "sokka@example.com"
}
```

Since the `name` property is empty the server will send back the following
response:

```java
{
    "timestamp": "2022-07-03T14:43:26.636+00:00",
    "status": 400,
    "error": "Bad Request",
    "path": "/api/members"
}
```

### `@Validated`

The `@Validated` annotation is applied to class controller to enable the
`@Valid` annotation for parameters marked with `@PathVariable` and
`@RequestParam` in addition to the `@RequestBody` as described above.

## Conclusion

We have learned how to enable validation in our models and validate requests. It
is important to add validation to ensure data integrity in our application.
