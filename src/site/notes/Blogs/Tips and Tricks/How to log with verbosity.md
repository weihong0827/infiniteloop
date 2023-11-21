---
{"tags":["blogs"],"Author":"Qiu Weihong","creation date":"2023-11-08 09:54","modification date":"Wednesday 8th November 2023 09:54:11","publish":null,"priority":null,"topics":["Tips and Tricks"],"banner":null,"dg-publish":true,"permalink":"/blogs/tips-and-tricks/how-to-log-with-verbosity/","dgPassFrontmatter":true,"created":"2023-11-08T09:54:11.000+08:00","updated":"2023-11-08T09:54:55.000+08:00"}
---

---

# Simplifying Logging Verbosity in Go

When developing applications in Go, logging is a critical component for debugging and monitoring. However, too much logging can be as problematic as too little. This is where verbosity levels come into play, allowing developers to choose the amount of logging detail they need. In this post, we'll explore a simple method to implement different logging verbosity levels in Go using the standard library.

### Step 1: Define Verbosity Level Flag

Instead of a boolean flag for a simple on/off switch, we use an integer flag to represent the verbosity level:

```go
var verbosity = flag.Int("verbosity", 0, "verbosity level for output")
```

### Step 2: Set Up Loggers

We define three different loggers corresponding to different verbosity levels:

```go
var (
    Info    *log.Logger
    Warning *log.Logger
    Debug   *log.Logger
)
```

### Step 3: Initialize Loggers Based on Verbosity

In the `init` function, we set up the loggers. By default, all loggers discard their output. Depending on the verbosity level set by the user, we change the output destination to `os.Stdout`:

```go
func init() {
    flag.Parse()

    // Default to discard
    Info = log.New(ioutil.Discard, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile)
    Warning = log.New(ioutil.Discard, "WARNING: ", log.Ldate|log.Ltime|log.Lshortfile)
    Debug = log.New(ioutil.Discard, "DEBUG: ", log.Ldate|log.Ltime|log.Lshortfile)

    // Enable based on verbosity level
    switch *verbosity {
    case 2:
        Debug.SetOutput(os.Stdout)
        fallthrough
    case 1:
        Warning.SetOutput(os.Stdout)
        fallthrough
    default:
        Info.SetOutput(os.Stdout)
    }
}
```

### Step 4: Use the Loggers

Now you can use the `Info`, `Warning`, and `Debug` loggers throughout your code without additional verbosity checks:

```go
func main() {
    Info.Println("This is an informational message")
    Warning.Println("This is a warning message")
    Debug.Println("This is a debug message")
}
```

### Running Your Application with Verbosity Levels

To run your application with different levels of verbosity, use the `-verbosity` flag:

```sh
go run yourprogram.go -verbosity=1
go run yourprogram.go -verbosity=2
```

The higher the number, the more detailed the output.

### Conclusion

This simple yet effective approach to controlling verbosity in Go leverages the power of the standard library's `log` package and the `flag` package for runtime configuration. By initializing different loggers with varying output destinations based on the user-defined verbosity level, developers can ensure that their applications log the right amount of information, making debugging and monitoring a breeze.

Remember to test the logging thoroughly to ensure that the verbosity levels are correctly configured and that your application behaves as expected.
