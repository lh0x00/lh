## A Detailed Guide to Golang Coding Standards

This document compiles best practices for writing Golang code, based on Google's style guide. It focuses on common scenarios and techniques that promote readability, maintainability, and ease of use. Each section provides in-depth analysis and illustrative examples.

### 1. Naming

Choosing descriptive and consistent names for variables, functions, and packages is crucial for expressing the intent and purpose of your code. 

#### 1.1. Function and Method Names

##### 1.1.1. Avoid Repetition

Function and method names should be concise and focus on the action being performed or the result returned. Avoid repeating information already available from the package name, type, pointer type, input variable names, and return types.

**Example:**

```go
// Bad: Repeats package name "yamlconfig" and type "Config"
package yamlconfig

func ParseYAMLConfig(input string) (*Config, error)

// Good: Concise, focuses on action "Parse"
package yamlconfig

func Parse(input string) (*Config, error) 
```

**Analysis:**

* The package name `yamlconfig` clearly indicates the function's relation to YAML configuration.
* The return type `*Config` is already defined in the function signature.
* Repeating this information in the function name makes it verbose and less readable.

##### 1.1.2. Naming Conventions

Beyond avoiding repetition, adhere to these naming conventions:

* **Functions returning values:** Use nouns to represent the returned result. Examples: `JobName`, `UserData`.
* **Functions performing actions:** Use verbs to express the action of the function. Examples: `WriteTo`, `ProcessData`.
* **Identical functions differing by type:** Append the type name to the function name to distinguish them. Examples: `ParseInt`, `ParseInt64`, `ConvertToString`, `ConvertToInt`.
* **Primary functions:** You can omit the type name if the function is the primary version, commonly used. Examples: `Marshal`, `Unmarshal`.

**Example:**

```go
// Function returning a value:
func (c *Config) JobName() (string, error)

// Function performing an action:
func (u *User) UpdateProfile(data *ProfileData) error

// Identical functions, differing by type:
func ParseInt(input string) (int, error)
func ParseInt64(input string) (int64, error)

// Primary function:
func (d *Data) Marshal() ([]byte, error)
```

#### 1.2. Test Double Packages and Types

Testing is essential for ensuring code quality. Utilizing test doubles (stubs, fakes, mocks, spies) helps isolate and test code independently of other components.

##### 1.2.1. Creating Test Helper Packages

Packages containing test doubles should reside in a separate package from the production code, typically by appending `_test` to the package name.

**Example:**

Production package: `creditcard`

Test package: `creditcard_test`

##### 1.2.2. Naming Test Doubles

* **Simple:** If there's only one type requiring a double, use simple names like `Stub` or `Fake`.
* **Behavioral:** For multiple behaviors requiring simulation, name them based on the desired behavior. Examples: `AlwaysCharges`, `AlwaysDeclines`, `ReturnsError`.
* **Explicit:** If multiple types require doubles, use explicit names based on the type. Examples: `StubService`, `StubStoredValue`, `FakeDatabase`.

**Example:**

```go
// Package creditcard_test
package creditcard_test

// Simple Stub for creditcard.Service
type Stub struct{}

func (Stub) Charge(*creditcard.Card, money.Money) error { return nil }

// Stub with specific behavior:
type AlwaysDeclines struct{}

func (AlwaysDeclines) Charge(*creditcard.Card, money.Money) error {
    return creditcard.ErrDeclined
}

// Stubs for multiple types:
type StubService struct{}

func (StubService) Charge(*creditcard.Card, money.Money) error { return nil }

type StubStoredValue struct{}

func (StubStoredValue) Credit(*creditcard.Card, money.Money) error { return nil }
```

##### 1.2.3. Local Variables in Tests

When using test doubles within test functions, prepend a prefix to the variable name to distinguish it from the production type.

**Example:**

```go
// Package payment_test
package payment_test

func TestProcessor(t *testing.T) {
    // Prefix "spy" for the test double variable
    var spyCC creditcard_test.Spy 
    proc := &Processor{CC: spyCC}

    // ...
}
```

#### 1.3. Stomping and Shadowing

Golang allows assigning new values to declared variables (stomping). When using the `:=` operator within a new scope, a new variable with the same name is created, obscuring the original variable (shadowing).

**Stomping:**

```go
// Value of i is overwritten
i := 10
i := 20 
```

**Shadowing:**

```go
x := 10

if true {
  x := 20 // New x variable is created, shadowing the outer x
  fmt.Println(x) // Prints 20
}

fmt.Println(x) // Prints 10
```

**Analysis:**

* Stomping provides code conciseness when the old value is no longer needed.
* Shadowing can lead to confusion and requires careful usage.

#### 1.4. Util Packages

Util packages contain functions and types shared by multiple packages. Avoid generic names like `util`, `helper`, `common`, and instead use descriptive names that reflect the package's specific functionality.

**Example:**

```go
// Bad: Generic package name
package util

// Good: Package names reflecting functionality
package stringutil
package timeutil
```

#### 1.5. Package Size

Decompose packages based on functionality for better manageability and usability. Each package should focus on a specific purpose, avoiding overly large packages containing unrelated functionalities.

### 2. Imports

The `import` section declares packages used within the current file. Follow naming conventions, ordering, and grouping practices for readability and maintainability.

#### 2.1. Protos and Stubs

When working with Protobuf, generated packages need to be renamed to align with Golang naming conventions.

**Naming Conventions:**

* `pb`: Suffix for packages generated by `go_proto_library`.
* `grpc`: Suffix for packages generated by `go_grpc_library`.

**Prefixes:**

* Use short prefixes (1-2 letters) to distinguish packages. Examples: `fspb`, `fsgrpc`.
* Omit the prefix if only using one proto or the package is closely related to the proto.
* Use short word prefixes if proto names are generic or unclear. Example: `mapspb`.

**Example:**

```go
// Importing proto and gRPC packages
import (
    fspb "path/to/package/foo_service_go_proto"
    fsgrpc "path/to/package/foo_service_go_grpc"
)
```

#### 2.2. Import Ordering

Group and arrange imported packages in the following order:

1. **Standard library packages:** Examples: `fmt`, `os`, `io`.
2. **Project packages:** Examples: `github.com/user/myproject`, `internal/mypackage`.
3. **(Optional) Protobuf packages:** Examples: `pb "path/to/foo_go_proto"`.
4. **(Optional) Side-effect packages:** Examples: `_ "path/to/package"`.

**Example:**

```go
import (
    "fmt"
    "os"

    "github.com/user/myproject"

    pb "path/to/foo_go_proto"

    _ "path/to/package"
)
```

#### 2.3. Blank Imports

Blank imports (`import _`) should only be used within the `main` package or for testing purposes. Avoid using them in libraries to control dependencies and facilitate testing.

**Example:**

```go
// Within the main package, importing image/jpeg for its side-effects
package main

import (
    _ "image/jpeg"
)
```

#### 2.4. Dot Imports

Dot imports (`import .`) allow using functions and variables from another package without prefixes. Avoid this practice as it hinders identifying the origin of functions and variables.

### 3. Error Handling

Golang uses `error` to signal errors during execution. Adhere to conventions for returning errors, constructing error strings, and handling errors for readability and maintainability.

#### 3.1. Returning Errors

* Use `error` as the last return parameter of a function to indicate possible errors.
* Return `nil error` to signal successful execution.
* Avoid returning concrete error types; use `error` to prevent `nil` pointer issues within interfaces.

**Example:**

```go
// Function returning an error:
func ReadFile(filename string) ([]byte, error)

// Function returning nil error:
func ConnectToDatabase() error
```

#### 3.2. Error Strings

* Do not capitalize error strings unless they start with a proper noun, acronym, or exported name.
* Do not end error strings with punctuation.

**Example:**

```go
// Bad: Capitalized and ends with a period
return fmt.Errorf("Error: file not found.")

// Good: Not capitalized and no punctuation
return fmt.Errorf("file not found") 
```

#### 3.3. Handling Errors

Upon encountering an error, handle it in one of these ways:

* Handle the error immediately.
* Return the error to the caller.
* In exceptional cases, use `log.Fatal` or `panic`.

**Example:**

```go
// Handling error immediately:
data, err := ReadFile("data.txt")
if err != nil {
    fmt.Println("Error reading file:", err)
    return
}

// Returning error to caller:
func ProcessData(data []byte) error {
    // ...
    if err != nil {
        return fmt.Errorf("error processing data: %w", err)
    }
    // ...
}

// Using log.Fatal:
if err := ConnectToDatabase(); err != nil {
    log.Fatal("Error connecting to database:", err)
}
```

#### 3.4. In-Band Errors

Avoid using special values (e.g., -1, `nil`, empty string) to signal errors. Instead, return an additional value to indicate the validity of the other return values.

**Example:**

```go
// Bad: Returning -1 when key is not found
func Lookup(key string) int

// Good: Returning a boolean value to signal key presence
func Lookup(key string) (string, bool)
```

#### 3.5. Indent Error Flow

Handle errors before proceeding with the rest of the code. Indenting the error handling flow improves readability and comprehension.

**Example:**

```go
// Bad: Error flow is indented, hindering readability
if err != nil {
    // Handle error
} else {
    // Normal flow
}

// Good: Handle error first, normal flow remains unindented
if err != nil {
    // Handle error
    return
}

// Normal flow
```

### 4. Logging

Logging is an effective way to track and debug program execution. Follow conventions for logging, log levels, and error handling during logging.

#### 4.1. Avoid Duplication

Only log errors at the calling function, avoid redundant logging in child functions.

**Example:**

```go
// Bad: Logging error in both child and calling function
func processData(data []byte) error {
    if err != nil {
        log.Error("Error processing data:", err)
        return err
    }
}

func main() {
    // ...
    if err := processData(data); err != nil {
        log.Error("Error processing data:", err)
        // ...
    }
}

// Good: Only log error in calling function
func processData(data []byte) error {
    if err != nil {
        return err
    }
}

func main() {
    // ...
    if err := processData(data); err != nil {
        log.Error("Error processing data:", err)
        // ...
    }
}
```

#### 4.2. PII

Avoid logging Personally Identifiable Information (PII).

#### 4.3. Log Levels

* Use `log.Error` sparingly: `ERROR` level logging is more expensive than other levels.
* Custom verbosity levels: Use `log.V` for detailed logging during development and debugging.

#### 4.4. Program Initialization

When encountering errors during program initialization, log them using `log.Exit` and provide clear instructions on how to fix them.

#### 4.5. Program Checks and Panics

* Prefer returning errors over `panic` for handling errors.
* Use `log.Fatal` to terminate the program upon critical errors.
* Avoid recovering from `panic` to prevent the propagation of erroneous states.

### 5. Documentation

Documenting your code effectively is crucial for others to understand and use it.

#### 5.1. Conventions

* Write clear, comprehensive, and easy-to-understand documentation.
* Utilize runnable examples to illustrate code usage.
* Document fields and parameters prone to errors or with unclear behavior.
* Document the behavior of `context`, `concurrency`, `cleanup`, and `errors` explicitly.
* Preview Godoc documentation to ensure proper formatting.

#### 5.2. Godoc Formatting

* Use blank lines to separate paragraphs.
* Place runnable examples within test files.
* Indent code snippets with two spaces for verbatim formatting.
* Lines starting with a capital letter are formatted as headings.

### 6. Variable Declarations

#### 6.1. Initialization

* Use `:=` when initializing variables with non-zero values.
* Use `var` when initializing variables with zero values.

#### 6.2. Composite Literals

* Use composite literals when the initial values of variables are known.
* Avoid composite literals for empty values, use zero values instead.

#### 6.3. Size Hints

* Use `make` with size hints when the size of slices or maps is known beforehand.
* Most code doesn't require size hints; let the runtime handle automatic size adjustments.

### 7. Channel Direction

* Specify channel direction (`<-chan` or `chan<-`) to facilitate compiler error detection and clarify intent.

### 8. Function Argument Lists

* Limit the number of function arguments for readability and memorability.
* Use `option structs` or `variadic options` for functions with numerous parameters.

### 9. Command Line Interfaces (CLIs)

* Utilize libraries like `cobra` or `subcommands` to build complex CLIs.

### 10. Testing

#### 10.1. Test Helpers and Assertion Helpers

* Test helper functions perform setup or cleanup tasks, utilize `t.Helper` for error reporting.
* Avoid assertion helper functions; use `cmp` and `fmt` instead.

#### 10.2. Error Messages

* Write clear and comprehensive error messages, including:
    * Cause of the error
    * Input values
    * Actual results
    * Expected results
* Identify the function and input values in error messages.
* Display actual values (`got`) before expected values (`want`).

#### 10.3. Comparisons

* Compare full structures using `cmp.Equal` or `cmp.Diff`.
* Avoid comparing results that depend on unstable outputs.
* Continue testing after encountering errors, use `t.Error` instead of `t.Fatal`.

#### 10.4. Subtests

* Use `t.Run` to create subtests.
* Give subtests clear, descriptive, and filterable names.

#### 10.5. Table-Driven Tests

* Use table-driven tests when multiple similar test cases exist.
* Separate test functions when test logic differs.
* Isolate test configuration from test logic.

### 11. String Concatenation

* Use `+` for concatenating a few strings.
* Use `fmt.Sprintf` for formatting complex strings.
* Use `strings.Builder` for piecemeal string construction.
* Use backticks (``) for multi-line string literals.

### 12. Global State

* Avoid using global state due to difficulties in testing, error-proneness, and potential conflicts.
* Prefer instance values, allowing clients to create and use their own isolated values.

### 13. Common Libraries

* **`flag`**: Use snake case for flag names, camel case for variable names.
* **`glog`**: Google's logging library, use `log.Fatal` and `log.Exit` to terminate the program.
* **`context`**: Always the first parameter of a function, use `context.Background()` within entrypoint functions.

### 14. Conclusion

Adhering to these standards leads to Golang code that is readable, maintainable, and easy to use, ultimately contributing to more stable and efficient project development.

**References:**

* [Go Style Guide](https://google.github.io/styleguide/go/guide)
* [Go Style Best Practices](https://google.github.io/styleguide/go/best-practices)
* [Go Style Decisions](https://google.github.io/styleguide/go/decisions)
