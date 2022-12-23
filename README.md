# Validation

## Learning Goals

- Explain why validations are important.
- Define common validations in Spring.

## Introduction

A client can send invalid data to the server. It is important to implement
server side validation to make sure no invalid data is stored in the database.

The `javax.validation` package provides annotations for different types of
validations. We can either:

1. Add the dependency to an existing Maven project with a pom.xml.
2. Create a new Spring Boot application using the Spring Initializr.

We'll cover both options in this lesson in case you want to add a dependency to
an existing project or create a new one!

### Adding the Validation Dependency to an Existing Maven Project

1. Open up the pom.xml in the Maven project.
2. Under the `<dependencies>` header, add the following section to add the
   `javax.validation` package:

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

3. Reload the Maven project to finish adding the dependency.

### Adding the Validation Dependency to a New Spring Project

1. Navigate to the [Spring Initializr](https://start.spring.io/).
2. Select the project properties.
    1. Select "Maven Project", as we will use Maven as the build tool.
    2. Select "Java" as the language.
    3. Select the most recent release version of Spring Boot 2. (Make sure it does
       not have "SNAPSHOT" listed after it.)
    4. Select the appropriate Java JDK version.
3. Add dependencies.
    1. Click "ADD DEPENDENCIES".
    2. Search for "validation".
    3. Select "Validation" from the list.
    4. Add any other dependencies by repeating this process.
4. Click on the “Generate” button on the bottom. This will download a zip file
   containing the Spring Boot project.
5. Unzip the archive and open it in a preferred code editor or IDE.

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
@Pattern(regexp = "[0-9]{4, 6}")
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
a `@Valid` annotation to the `FootballTeamController` controller we built in
this section.

Open up the `FootballTeamWithChampionDTO` class and add the following
annotations to the `teamName` field:

```java
package com.example.springdatademo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import lombok.Data;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;

@Data
@JsonPropertyOrder({"team_name", "wins", "losses", "current_super_bowl_champion"})
public class FootballTeamWithChampionDTO {

   @NotNull
   @NotEmpty
   @JsonProperty("team_name")
   private String teamName;

   private int wins;

   private int losses;

   @JsonProperty("current_super_bowl_champion")
   private boolean currentSuperBowlChampion;
}
```

Now let's make an edit to the `FootballService` class to take in a
`FootballTeamWithChampionDTO`:

```java
// FootballService.java - just a small change to the parameter footballTeam

    public String addFootballTeam(FootballTeamWithChampionDTO footballTeam) {
        FootballTeam footballTeamEntity = modelMapper.map(footballTeam, FootballTeam.class);
        footballRepository.save(footballTeamEntity);
        return String.format("%s has been added!", footballTeam.getTeamName());
    }
```

Now open up the `FootballController` class and modify the `addFootballTeam`
method:

```java
package com.example.springdatademo.controller;

import com.example.springdatademo.dto.FootballTeamDTO;
import com.example.springdatademo.service.FootballService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@RestController
public class FootballController {

    private final FootballService footballService;

    @Autowired
    public FootballController(FootballService footballService) {
        this.footballService = footballService;
    }

    @PostMapping("/football-team")
    public ResponseEntity<String> addFootballTeam(@Valid @RequestBody FootballTeamWithChampionDTO footballTeam) {
        String status = footballService.addFootballTeam(footballTeam);
        return ResponseEntity.ok(status);
    }

    // Other methods for GET, PUT, and DELETE requests
}
```

Notice that we are returning a `ResponseEntity` instead of a `String`.
The `ResponseEntity` allows us to modify response information, like status
codes and headers, before sending them back to the client.

Now open up Postman and make the following `POST` request to
`http://localhost:8080/football-team`:

```json
{
   "team_name":"",
   "wins":7,
   "losses":3,
   "current_super_bowl_champion":0
}
```

Since the `team_name` property is empty the server will send back the following
response:

```java
{
    "timestamp": "2022-07-03T14:43:26.636+00:00",
    "status": 400,
    "error": "Bad Request",
    "path": "/football-team"
}
```

### `@Validated`

The `@Validated` annotation is applied to class controller to enable validation
annotations for parameters marked with `@PathVariable` and `@RequestParam` as
described above.

## Conclusion

We have learned how to enable validation in our models and validate requests. It
is important to add validation to ensure data integrity in our application.
