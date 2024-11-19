Mockito has a set of powerful methods that make mocking straightforward and effective. Here’s an overview of how to use Mockito and some of the most commonly used methods:

### 1. Basic Setup and Annotations
First, make sure Mockito is in your project dependencies (e.g., Maven, Gradle). For unit tests, you can use JUnit with Mockito for smooth integration. Common annotations include:

- **@Mock**: Marks a field as a mock object.
- **@InjectMocks**: Injects mocks (created with `@Mock`) into the object under test.
- **@BeforeEach**: Used for setting up mocks before each test.

Example setup:

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.mockito.Mockito.*;

public class ServiceTest {

    @Mock
    private Dependency dependency;

    @InjectMocks
    private Service service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this); // Initializes the mocks
    }
}
```

### 2. Stubbing Methods
**Stubbing** is where you define the behavior of a mock when a particular method is called.

- **when(...).thenReturn(...)**: Specifies what a method should return when called with specific arguments.
- **when(...).thenThrow(...)**: Specifies that an exception should be thrown for a specific method call.

Example:

```java
when(dependency.getData()).thenReturn("mocked data");
when(dependency.getData()).thenThrow(new RuntimeException("Error occurred"));
```

You can also use **Argument Matchers** like `anyInt()`, `anyString()` to handle dynamic arguments:

```java
when(dependency.process(anyInt(), anyString())).thenReturn("processed data");
```

### 3. Verification
**Verification** is used to check if specific methods were called on the mock and if they were called with the expected arguments.

- **verify(...)**: Verifies that a method was called.
- **verify(..., times(n))**: Verifies that a method was called a specific number of times.
- **verifyNoMoreInteractions(...)**: Verifies that no other methods were called on the mock.

Example:

```java
service.performAction();

verify(dependency).someMethod(); // Verify someMethod() was called
verify(dependency, times(1)).someMethod(); // Verify it was called exactly once
verify(dependency, never()).anotherMethod(); // Verify anotherMethod() was never called
```

### 4. Commonly Used Methods in Mockito
Here are some of the most commonly used methods in Mockito and what they do:

#### `mock()`
Creates a mock object of a class or interface.

```java
Dependency mockDependency = mock(Dependency.class);
```

#### `when(...).thenReturn(...)`
Defines what a mock should return when a specific method is called.

```java
when(mockDependency.someMethod()).thenReturn("value");
```

#### `when(...).thenThrow(...)`
Defines that a specific method call on the mock should throw an exception.

```java
when(mockDependency.someMethod()).thenThrow(new RuntimeException());
```

#### `verify()`
Checks that a specific method was called on the mock. Useful for asserting behavior rather than results.

```java
verify(mockDependency).someMethod();
```

#### `verify(..., times(n))`
Checks that a method was called exactly `n` times.

```java
verify(mockDependency, times(2)).someMethod();
```

#### `verifyNoMoreInteractions()`
Ensures that no other interactions happened on the mock beyond the verified ones.

```java
verifyNoMoreInteractions(mockDependency);
```

#### `doThrow(...).when(...)`
Defines a method to throw an exception in scenarios where you cannot use `when(...).thenThrow(...)`.

```java
doThrow(new RuntimeException()).when(mockDependency).someMethod();
```

#### `doReturn(...).when(...)`
Sets a return value for methods that cannot be stubbed with `when(...)`.

```java
doReturn("value").when(mockDependency).someMethod();
```

### Example: Putting It All Together
Here’s an example test case with Mockito for a `UserService` class that interacts with a `UserRepository` dependency:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

public class UserServiceTest {

    @Mock
    UserRepository userRepository;

    @InjectMocks
    UserService userService;

    @BeforeEach
    public void setup() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetUser() {
        // Arrange
        User mockUser = new User("John", "Doe");
        when(userRepository.findByName("John")).thenReturn(mockUser);

        // Act
        User result = userService.getUser("John");

        // Assert
        assertEquals("John", result.getFirstName());
        verify(userRepository).findByName("John"); // Verify method interaction
        verifyNoMoreInteractions(userRepository);
    }
}
```

In this example:
- **`@Mock`** creates a mock `UserRepository`.
- **`@InjectMocks`** injects the mock into `UserService`.
- **`when(...)`** defines behavior for `findByName`.
- **`verify(...)`** ensures `findByName("John")` was called.

### Summary of Most-Used Mockito Methods:
1. `mock(...)`: Create a mock.
2. `when(...).thenReturn(...)`: Stub a method to return a specific value.
3. `when(...).thenThrow(...)`: Stub a method to throw an exception.
4. `verify(...)`: Verify a method was called.
5. `doReturn(...).when(...)`: Alternative stubbing syntax for special cases.
6. `anyInt()`, `anyString()`: Matchers for flexible argument handling.

Mockito’s power comes from these simple methods that let you create complex, isolated tests. It’s a great toolkit for unit testing, especially for Java backend projects where services often interact with multiple dependencies.
