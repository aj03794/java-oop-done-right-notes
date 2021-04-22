# Java OOP Done Right

- https://leanpub.com/javaoopdoneright

## What is an object, anyways?

### Object Oriented Design is Behavior Driven Design

- Key to doing OOPS right is to let behaviors drive the design
- `What does an object need to do?`
    - We represent that as a method, using the name to describe the behavior

## Clean Code

### Good Names

- Choosing *intention revealing names* speeds up your team's ability to read the code

### Method names: Tell me the outcomes

- Effective methodnames explain why you should call them
- Explains what will happen after the method completes
- Should no

t explain what is inside the method

<br>

- Good: `Users findNewUsersSince(Date date)`
- Bad: `Users runDatabaseQueryToSortUsersByJoiningDateThenMap(Date date)`

<br>

- The method name *should be the comment* for a block of code
- The code itself explains how the method works
- The method name serves as a comment explaining what the method does if I call it
- Don't use comments as a subsitute for method names, turn the comment into the method name

### Variable names: Tell me the contents

- **Methods say what they do, variables say what they hold**

### Design methods around behaviors, not data

- Answer `what behaviors should the class have?` before deciding what data those behaviors might need

### Hidden data - No getters, no setters

- Don't privde public getters/setters to clients by default, start with everything encapsulated

## Aggregates: More than one

### Greeting more than one user

```
public class Users {
2 private final List<User> users = new ArrayList<>();
3
4 public void add( User u ) {
5 users.add(u);
6 }
7
8 public void greet() {
9 users.forEach( User::greet );
10 }
11 }
```

- This is an **Aggregate** class
- It aggregates more than one User object into a single Users object

## The SOLID Principles

### The five SOLID principles

- SRP Single Responsibiliy Principle
- OCP Open/Closed Principle
- LSP Liskov Subsitution Principle
- ISP Interface Segregation Principle
- DIP Dependency Inversion Principle

-SDLOI signifies order of importance

### SRP Single Responsbility - do one thing well

- Each class should do one thing

### DIP Dependency Inversion - Bring out the Big Picture

- Using this you create swappable `plugin` style components
- Two big benefits
    - Changes in one component do not ripple through any others
    - You can replace components with ones designed to help TDD tests

### Inversion - Injection: two sides of the same coin

- Dependency Inversion is the *design* technique
    - It's the thought process by which we remove a hard-coded coupling from the software
- Dependency Injection is the *implementation* technique
    - We have inverted dependency
    - Now we need some way of choosing a suitable concrete implementation and plugging it in - this is DI
- DI at it's heart (like in Spring/Java EE/etc) is new-ing up classes and passing them into a constructor

## TDD and Test Doubles

- 

## Hexagonal Architecture

- Solves issue of working with external systems

### The problems of external systems

- Many examples
    - Database
    - Files
    - Rest controllers
    - Web servers
    - I/O hardware
    - Printers
    - GUIs
    - Displays
    - Hardware
- How do we write code to test our application working with these external systems?
    - How do you arrange for keys tobe physically pressed on a keyboard by a unit test?

### The Test Pyramid

- **Integration tests** take several pieces of code, join them up and test them as a whole
- **End-to-end tests** write up the entire systemand test it just as a user would
    - You would drive the user interface and run against actual databases and real web services
    - The whole thingis as `live-like` as possible
- **Contract tests** split a test in two directions
    - They are used to break apart testing against real external services
    - With these you test twice
        - First, you test that your applicationlogic can call a stub for the service correctly
            - This is what `testing the contract` means - the contract beween your application and whatever it talks to
        - Second, you test the contract between the code that talks to the external service and makesure that it does talk correctly
- **Browser Tests** are a form of end-to-end testing
    - Can script mouse movements, taps, and keystrokes as if you were a real user

<br>

- Higher level tends tend to be `flaky` which reduces consistent repeatability
    - Code may be right, but something in the external systemmight still cause the test to fail
    - Can be caused by intermittent problems like network failures and database connection limits

### Removing external systems


```
1 class NotFaceBook {
2 private final UserRepository users ;
3 private final Display display ;
4
5 public NotFaceBook( UserRepository users, Display display ) {
6 this.users = users ;
7 this.display = display;
8 }
9
10 public void showProfile() {
11 String userId = "almellor_19019";
12
13 User u = users.findUserById( userId );
14
15 u.displayProfile(display);
16 }
```

- In this example, UserRepositoru (an interface) is passed in as a dependency through the constructor
    - Classic dependency inversion and it is also dependency injection
- This makes unit testing simple, can just create a `StubUserRepository` class that returns some canned data

### The Hexagonal Model

- Code consists of two main parts
    - Application logic, which is free of external systems
    - External systems, which are free of application code
- Those parts translate to the domain layer (application logic) and adapter layer (external systems)

<br>

- Inner hexagon (domain layer) is where application logic and our abstractions of external systems live (like User Repository in our previous example)
- Outer hexagon contains adapter classes - they hold the details of talking to the external systems, implementing those interfaces

![](./images/1.png)

- The adapter layer implements interfaces and uses classes from the Domain layer
    - It must depend on the domain layer
- The domain layer MUST NOT use classes from the adapter layer
- Adaptter layer is where details of the external systems appear
    - This is the place for SQL, HTTP calls, HTML templating
- Domain layer is open to changes in external systems w/o needing modification (it obeys Open-Closed Principle)

### Inversion/Injection: Two sides of the same coin

- Dependency Inversion and Dependency Injection are closely related but not the same
- Dependency *Inversion* happens at design time - that's when we create abstract interfaces like UserRepository
- Dependency *Injection* happens at run time - when we pass implementations of UserRepository interface to calling code

## TDD and Test Doubles

```
1 class TextConversion {
    2 private Input input ;
    3 private Output output ;
4
5 public TextConversion(final Input input, final Output output){
    6 this.input = input;
    7 this.output = output;
8 }
9
    10 public void showInputInUpperCase() {
        11 String inputText = input.fetch();
        12 String upperCaseText = inputText.toUpperCase();
        13 output.display(upperCaseText);
    14 }
15 }
```

- Code uses interfaces Input and Output to fetch input and display output
- No knowledge of how input is collected or where output is sent to

### Test Doubles - Stubs and Mocks

- We replace the physical keyboard and the console during tests with `Test Doubles`

#### DIP for Unit Tests - Stubs and Mocks

- Input test double is a `stub`
- Output test double is a `mock`

<br>

- A stub is an implementation of one of our dependency interfaces that *returns known data*
- A mock is complementary to a stub
    - The mock implements the interface, and *records any interactions* with it

<br>

- A MockOutput class can implement Output
    - It can capture whatever we send to the `display()` method and then we can assert against this captured data

```
1 class TextConversionTest {
2
    3 @Test
    4 public void displaysUpperCasedInput() {
        5 // Arrange
        6 var in = new StubInput("abcde123");
        7 var out = new MockOutput();
        8
        9 var tc = new TextConversion( in, out );
        10
        11 // Act
        12 tc.showInputInUpperCase();
        13
        14 // Assert
        15 assertThat(out.getActual()).isEqualTo("ABCDE123");
    16 }
17 }
```

- This follows usual Arrange, Act, Assert pattern
    - Set up our stub input value
    - Create a mock to capture output
    - Dependency inject the stub and mock to code under test
    - execute the code we want to test
    - assert against the captured output, given the known input

```
1 class TextConversionTest implements Input, Output {
    2 private final Input input = this ;
    3 private final Output output = this;
    4
    5 private String actualOutput ;
    6
    7 // Input interface stub
        8 public String fetch() {
        9 return "abcde123";
    10 }
    11
    12 // Output interface mock
    13 public void display( String output ) {
        14 this.actualOutput = output;
    15 }
    16
    17 @Test
    18 public void displayUpperCasedInput() {
        19 // Arrange
        20 var tc = new TextConversion(input, output);
        21
        22 // Act
        23 tc.showInputInUpperCase();
        24
        TDD and Test Doubles 85
        25 // Assert
        26 assertThat(actualOutput).isEqualTo( "ABCDE123" );
    27 }
28 }
```

- Test class acts as its own test doubles

### Self-Shunt mocks and stubs

- Make the test class implement the interfaces themselves
- No need for hand made test double classes and no need for mocking libraries
- Get some design feedback as well for when our interfaces get too big (test gets annoying to write )

## Handling Errors

### Three kinds of errors

- Errors we can fix
    - `User typed wrong data`
        - Ask again
- Errors we can't fix
    - `Disk drive has crashed and all data was lost`
- Errors we don't care about
    - `Temperature sensor not working when temperature feature is off`

### Example - UserRepository

- findUserById(id) method should always return one and only one User object
- The User object has one method `showProfile()`, which displays profile information

### The null reference

```
1 User findUserById( String userId ) {
2   return null ;
3 }
```

- Calling code must check for the null return and decide what to do
- Problems include:
    - Cluttering up code with error paths
    - Many extra if statement paths to write and test (and potentially forgetting a test)

```
1 User u = users.findUserById( "alan0127" );
2 u.showProfile(); // BLOWS UP if u is null with NullPointerException
```

### Null object pattern

- Uses polymorphism to return one of two objects
    - Either the valid User or a NullUser object will be returned

```
1 User findUserById(String id) {
    2 return NullUser();
3 }
```

- NullUser is a fully (LSP) subsitutable for User
- Can either do this by:
    - Having a User interface and creating a RealUser and a NullUser implementation
    - Create a User class with a NullUser subclass

```
1 class NullUser extends User {
    2 public void showProfile() {
        3 // No Action
    4 }
5 }
```

- This pattern designs-out null reference bugs by not using null
- Have a NullUser increases number of necessary class to use and maintain

### Zombie Object

- The `Zombie Object` is named because it represents null to be *the walking dead*
- We modify our User class to add a *zombie detection* method

```
1 class User {
    2 boolean isValid() {...}
    3
    4 void showProfile() {...}`
5 }
```

- The new `isValid()` method tells us whether the object has valid data or not
- If `isValid()` returns false, you must not use the object
- Check this every time you use the object
- Calling code has the conditional just like the null check

```
1 User u = findUserById("alan0127");
2
3 if ( u.isValid() ) {
    4 u.showProfile();
5 }
```

- The pattern of `if (object.isValid()) {  /* use object */ }` is the signature of a Zombie object

### Exceptions - a quick primer

```
1 class ExceptionsDemo {
2 public void demonstrateExceptions() {
3 try {
4 System.out.println("This will print out 1");
5 methodWhichThrowsAnException();
6
7 // Line below will never run ...
8 System.out.println("This code will never be run 2");
9 }
10 catch( RuntimeException e ) {
11 System.out.println("You will see this - exception caught");
12
13 System.out.println("Exception message was: " + e.getMessage());
14 }
15 }
16
17 private void methodWhichThrowsAnException() {
18 System.out.println( "This will print out 2" );
19 throw new RuntimeException("Used to report an error");
20
21 // Unreachable code - this will never run
22 System.out.println("This code will never be run 1");
23 }
24 }
```

- The idea here is that the keyword `throw` when it runs, changes the execution flow
- Code will immediately exit whatever method it is in w/o executing any more lines of that function
    - LIke a form of an early return statement
- Works with `try/catch` block
- In practice, there are 3 ways of using exceptions

#### Design By Contract, Bertrand Meyer style

- A method call is a contract between the code inside the method and its calling code
- Our `findUserById(id) method declares a binding contract: if you call that method, you *will* receive a valid User object

<br>

- Meyer saw exceptions as a way to indicate **that contract cannot be satisfied**
- You would throw an exception in `findUserById()` if for any reason a User object could not be returned
    - This would indicate that the contract had been broken

```
1 try {
2 User u = findUserById( "alan0127" );
3
4 // if we reach here, we are guaranteed User u is valid
5 // due to our contract
6 u.showProfile();
7 }
8 catch( Exception e ) {
9 // if we reach here, we know the User object was invalid
10 // we can handle that
11 }
```

- In this way, exceptions clean up calling code
- Eliminate null passing
- Eliminate conditionals
- You either get a result or an exception

<br>

- If exception classes are made to extend `Exception`, the Java compiler will enforce that the exception is either caught at the call site, or it is delcared as being thrown up the call stack
- By contract, anything extending `RuntimeException` doesn't have as much scrutiny
    - This is the preferred way of using exceptions

#### Fatal errors: Stop the world!

- Another view is that you only use exceptions for an unfixable problem that needs to stop the program immediately
- These will be a RuntimeException (or subclass) that is thrown and never caught
- The JVM will then kill whatever thread was running when that uncaught exception happened
    - If that happens to be the main thread, the program exits

#### Combined approach: Fixable and non-fixable errors

- Use exceptions to represent fixable errors
- Use different exceptions to represent non-fixable errors

```
1 for (int attempts = 0; attempts < 3; attempts++) {
2 try {
3 callUnreliableService();
4 return ; // happy path - exit the method
5 }
6 catch( ServiceTemporarilyUnavailableException se ) {
7 Thread.sleep(1000); // wait one second
8 }
9 }
10
11 throw new ServiceUnavailableException();
```

- If code hasn't worked 3 times in arow, we throw the `ServiceUnavailableException`
- This exception tells the calling code that it has a problem, and should work around it, or terminate the program

<br>

- This approach is based on the idea that each software subsystem can only report that it is not working
    - Doesn't know whether this is a problem for the calling code
- The calling code is the best place to know how to handle this
- Our code has handled all the exceptions it knows how to - then delegates upwards when nothing left to attempt

## Design Patterns

### Adapter

- This is used in hexagonal architecture
- Solves the problem where we have calling code expecting one kind of interfae, and a supplier that has another
- We solve this by writing a middle layer that knows how to talk to 

- Suppose we have a `UserRepository` interface that a client is expecting to use

```
1 interface UserRepository {
    2 User findUserById( String userId );
3 }
```

- Suppose we have a small SQL library that we want to use

```
1 class Database {
    2 Rows executeQuery( SqlQuery query ) {
      3 /// ... code
    4 }
5 }
```

- We might create an adapter class for our UserRepository like this:

```
1 class UserRepositoryDatabase implements UserRepository {
    2 private final Database db ;
3
    4 public UserRepositoryDatabase( Database db ) {
        5 this.db = db;
    6 }
7
    8 User findUserById( String userId ) {
        9 SqlQuery userQuery =
        10 new SqlQuery("SELECT * FROM Users WHERE id=" + userId);
        11
        12 Rows r = db.executeQuery(userQuery);
        13 User u = new User( userId, r.getColumn("dateOfBirth") );
        14
        15 return u ;
    16 }
17 }
```

- The key is we haven't altered our calling code that depends on UserRepository to know anything about the SQL database library
- We haven't altered the SQL database librar to know anything about the calling code
- We have written code that *adapts* what our calling code wants into what our existing libraries can provide
- GOF calls this pattern *bridge*


### External System (Proxy)

- Best way to deal with an external system is to
    - Create an interface to represent it
    - Ue only our domain terms to abstract away the details
- This pattern is used heavily in Hexagonal architecture
- This allows us to use simple Fakes during development when necessary
    - A Fake is a stub used in development whilst we build the real thing
- An external system object represents *only what we need* of that system
    - Google maps has a large number of feature but our app may only need a small subset of those features

```
1 interface Geography {
    2 String getNameOfTownNearestTo( float lat, float long );
3 }
```

- The `GoogleMapsGeography` would implement that interface and would connect to the Google Maps API over HTTP
- This is part of a useful design idea known as *consumer driven contracts*

<br>

### Request-Service-Response

- Describes a flow to get a response from a service
- This crops up a lot in Hexagonal Architecture
- In a web request example to a service we are building:
    - Application makes request to our service - GET /users/alan0217
    - Our service extracts the user id from the request (for example)
        - alan0217
    - Our inner hexagon receives just the string of alan0217
    - Our inner hexagon uses a `UserRepository` to get that User
    - Inner hexagon returns User to web adapter layer
    - Web adapter layer than formats it to be sent back to the clinet