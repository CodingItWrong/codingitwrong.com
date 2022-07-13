---
title: "Spring Framework: An Opinionated Intro"
---

Spring Framework offers a lot of powerful features for making web apps simply, but you have to wade through some complexity to find those features. For example, try starting a Spring tutorial and seeing the following Choose Your Own Adventure:

* You can start from scratch or download the starter project.
* If you start from scratch, you can build it with Maven or Gradle or with your IDE
* If you build with your IDE, you can pick Spring Tool Suite or IntelliJ IDEA. Either way, you still need to pick either Maven or Gradle.
* And if you downloaded the starter project, you will still probably need to set it up in an IDE.

Not to mention there is some IDE-specific knowledge needed of how to configure the project and run the app!

That's a lot of decisions to make before you can even try Spring out when you're brand new to the Spring ecosystem.

To make it easier, here's a more opinionated tutorial. We'll build a RESTful JSON web service backed by a Postgres database. We'll use IntelliJ as it's an extremely popular Java IDE, and Gradle as it's the newer dependency and build system.

## Setup

Download and install the latest version of [IntelliJ IDEA][intellij]--be sure to choose the free Community edition rather than the paid Ultimate edition.

Download a copy of [PostgreSQL][postgres] as well--on macOS, an especially easy option is [Postgres.app][postgres.app]. You'll need a Postgres client to connect to the database as well. A nice free macOS GUI client is [Postico][postico]. Start your Postgres server, then connect to it and create a database named `springboot_videogames`.

Next go to <https://start.spring.io> -- this is Spring Initializr, a tool to help configure starter Spring projects. Choose the following:

- Project: Gradle Project
- Language: Java
- Spring Boot: 2.7.1
- Project Metadata
  - Artifact: "video-games"
  - Java: choose the version of Java you have installed
  - Leave other settings as-is
- Dependencies: Search for and add the following:
  - Spring Web
  - Spring Data JPA
  - PostgreSQL Driver
  - Lombok

Click "Generate" to download a zip file of your project. Expand it and put it somewhere you like. Next, open IntelliJ and you should see the Welcome screen. Click "Import Project". Navigate to the `video-games` folder you created, then choose the `build.gradle` file and Open it. Keep all the defaults and click OK, and after a few screens the project should open in IntelliJ. A build process will run to download all the dependencies.

Now we're set to start creating our web service.

## Database Connection

First, let's configure out database connection. Open the `src/main/resources/application.properties` file. Enter the following:

```properties
spring.jpa.hibernate.ddl-auto=create
spring.datasource.url=jdbc:postgresql://localhost:5432/springboot_videogames
spring.datasource.username=postgres
spring.datasource.password=
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults = false
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL9Dialect
```

Here's what these do:

- `spring.jpa.hibernate.ddl-auto=create` tells the Hibernate ORM to automatically create the database for us. Warning: this seems to drop and recreate the database upon starting the app!
- The `spring.datasource` entries specify the connection info for the database. These values should work for the Postgres.app defaults; if you have a different setup, you may need to customize them.
- The other two `spring.jpa` values work around [an error when connecting to Postgres from Spring](https://vkuzel.com/spring-boot-jpa-hibernate-atomikos-postgresql-exception).

## Model Class

Next, let's create the class that represents our data, a video game record. In the left-hand sidebar, open the `src/main/java` folder. You'll see the package name you picked in Spring Initializr. Right-click on it and choose New > Java Class. Name it `Game`. Then add the following to the file. You shouldn't need to type the imports at the top; as you type the other code, IntelliJ will offer you autocomplete options and will add the appropriate import statements as needed. If you don't autocomplete, IntelliJ will show the identifier in red because it isn't imported. You can click on it, then press option-space to automatically add the import after the fact. (Note that your `package` value will be different, depending on the settings you inputted in Spring Initializr.)

```java
package com.codingitwrong.tutorial.springbootvideogames;

import lombok.Data;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Data
@Entity
public class Game {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;

    private String name;

    private Integer releaseYear;
}
```

This defines a class with three fields: an integer ID, the name of the game, and the year it was released. Here's what the annotations you entered do:

- `@Data` tells Lombok to automatically generate field getters, setters, a constructor, and some other utility methods. This is a lot of boilerplate code you don't need to write!
- `@Entity` marks the class as persistable to the database.
- `@Id` marks the `id` field as the database ID, and `@GeneratedValue(strategy=GenerationType.AUTO)` tells Spring to let the database to assign the ID.

## Repository

Next, we'll create the repository that allows us to read from and write to the database. This is actually going to be really trivial. Create another New > Java Class, enter the name `GameRepository`, and choose Kind: Interface. Then make it extend from the following:

```java
package com.codingitwrong.tutorial.springbootvideogames;

import org.springframework.data.repository.CrudRepository;

public interface GameRepository extends CrudRepository<Game, Integer> {
}
```

This says that we want a repository of `Game`s with the IDs being `Integer`s. We don't actually need to implement this interface; Spring will handle that for us!

Rather than tell you the methods `CrudRepository` provides to us, let's demonstrate them by jumping straight into using them.

## Controller

Now we'll create the controller that allows us to access our `Game` data via the web. Create a `GamesContoller` class. Let's do the setup first, then we'll add one action at a time.

```java
package com.codingitwrong.tutorial.springbootvideogames;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/games")
public class GamesController {
    @Autowired
    private GameRepository repository;


}
```

Here's what these annotations do:

- `@RestController` tells Spring this class should function as a controller, and that we will be returning data records to be serialized in the response, rather than manually rendering a view or something.
- `@RequestMapping("/games")` says all the actions in this controller should be nested under the "/games" path.
- `@Autowired` tells Spring to inject an instance of `GameRepository` when it creates the controller.

Now that our controller is set, let's add one action at a time. First, let's allow listing games:

```java
@GetMapping("")
Iterable<Game> index() {
    return repository.findAll();
}
```

This is pretty simple. We create a `@GetMapping` and pass it the empty string, so that this will be available at the "/games" route as configured on the class itself. The method name `index()` is arbitrary; we could name it anything we like, but I've chosen to go with the names Ruby on Rails uses for RESTful actions. We call `findAll()` on the repository to get all games, then return them. That's it--Spring will handle serializing them to JSON for us. Pretty nice!

Next let's allow retrieving a single game by ID:

```java
@GetMapping("/{id}")
Game show(@PathVariable Integer id) {
    return repository.findById(id)
            .orElseThrow(() -> new GameNotFoundException(id));
}
```

We make this endpoint available at a path "/games/{id}", which allows a dynamic ID to be passed in. We try to find the game by that ID, and if it's not found, we throw an exception.

`GameNotFoundException` will appear in red because we haven't created it yet. Click on it, then press option-space. You'll get a few different options, the first of which being "Create class 'GameNotFoundException'". Choose that option, and keep the default package. Implement it like so:

```java
package com.codingitwrong.tutorial.springbootvideogames;

public class GameNotFoundException extends RuntimeException {
    public GameNotFoundException(Integer id) {
        super("Could not find game " + id);
    }
}
```

Next, let's create a POST endpoint to create a game:

```java
@PostMapping("")
@ResponseStatus(HttpStatus.CREATED)
Game create(@RequestBody Game newGame) {
    return repository.save(newGame);
}
```

`@RequestBody` automatically converts passed-in JSON data to a `Game` record, which we then save to the repository. We specify the `@ResponseStatus` should be `CREATED` (status code 201), which is conventional for creation operations. The created `Game` record will also be returned, which will let us see the ID it was assigned.

Now, updating:

```java
@PatchMapping("/{id}")
Game update(@RequestBody Game newGame, @PathVariable Integer id) {
    return repository.findById(id)
            .map(game -> {
                game.setName(newGame.getName());
                game.setReleaseYear(newGame.getReleaseYear());
                return repository.save(game);
            })
            .orElseThrow(() -> new GameNotFoundException(id));
}
```

If a game with the passed-in ID is found, we update its fields and save it; if it's not found, we throw an exception.

You'll notice errors for the getters and setters for name and release year. This is because we're relying on Lombok to generate those accessors, but IntelliJ doesn't know it's going to do so. We can fix this by adding a Lombok plugin to IntelliJ. From the "IntelliJ IDEA" menu, choose "Preferences…", then click "Plugins", then "Marketplace". Search for Lombok. Install it, then click "Restart IDE". IntelliJ will hold off on fully highlighting your file while it runs an Indexing operation. When it finishes, you'll get a warning from the Lombok plugin that it requires an Annotation Processing feature to be enabled. Click on the notice, then click the link for "Settings > Build > Compiler > Annotation Processors". Check the checkbox next to "Enable annotation processing", then click "OK" to save the settings. Now syntax highlighting should be fully enabled on your file, and IntelliJ should recognize the getters and setters as fine. Pretty nice!

Finally, let's add a delete endpoint:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
void destroy(@PathVariable Integer id) {
    repository.deleteById(id);
}
```

## Trying it out!

We should be set to run our web service now. To run it, look for a class whose name ends in `Application` named after the name you gave your app in Spring Initializr. For example, I named the app `spring-boot-video-games` so I have a `SpringBootVideoGamesApplication` class. This class is set up to kick off Spring. Click on it, then press control-shift-R to run the application starting from this class. At the end of a series of log statements you should see something like:

```bash
Tomcat started on port(s): 8080 (http) with context path ''
Started SpringBootVideoGamesApplication in 5.835 seconds (JVM running for 7.221)
```

Let's interact with our web service using the [Postman][postman] API client. Download and start Postman. Create a new POST request to `http://localhost:8080/games/`. From the Body tab, choose "raw", then "JSON", then enter something like the following:

```json
{
	"name": "Final Fantasy 7",
	"releaseYear": 1997
}
```

Click Send and you should see a "201 Created" response with the following:

```json
{
    "id": 1,
    "name": "Final Fantasy 7",
    "releaseYear": 1997
}
```

Now, send a GET request to `http://localhost:8080/games/` and you should see:

```json
[
    {
        "id": 1,
        "name": "Final Fantasy 7",
        "releaseYear": 1997
    }
]
```

You can play with the other endpoints by sending:

- A `GET` to `http://localhost:8080/games/1`
- A `PATCH` to `http://localhost:8080/games/1`
- A `DELETE` to `http://localhost:8080/games/1`

## What's next?

You've got a functioning Spring web service! What's more, now that you're up to speed on working with Spring Initializr apps in IntelliJ, you should be set to more easily pick up any of the other [Spring Guides][spring-guides]. Enjoy!

[intellij]: https://www.jetbrains.com/idea/download/
[initializr]: https://start.spring.io/
[postgres]: https://www.postgresql.org/download/
[postgres.app]: https://postgresapp.com/
[postico]: https://eggerapps.at/postico/
[postman]: https://www.getpostman.com/
[spring-guides]: https://spring.io/guides
