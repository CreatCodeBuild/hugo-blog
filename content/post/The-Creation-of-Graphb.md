---
title: "The Creation of Graphb"
date: 2018-06-07T20:06:46-07:00
draft: false
---
Recently, I created a Go GraphQL client library [graphb](https://github.com/udacity/graphb). It is dedicated to solve the string building problem when a developer wants to query from a GraphQL server.

You can see many examples in the repo, here I would like to discuss some of the techniques I used to build this library, which I consider idiomatic Go.

## Error Handling
### A mindless version
```go
func YourFunc() error {
    ...
	ret, err := SomeFunc()
    if err != nil {
        return err
    }
    ...
}
```

### A slightly better version
```go
import "github.com/pkg/errors"
func YourFunc() error {
    ...
    ret, err := SomeFunc()
    if err != nil {
        return errors.WithStack(err)
    }
    ...
}
```

The above 2 versions are only good if you:
1. Do care about the error
2. Don't know what to do

### 2 kinds of error handling
Before we proceed, we must notice that there are 2 kinds of error handlings:
1. Handle the error you get from other APIs.
```go
ret, err := SomeFunc()
// what to do with err?
```
2. Providing an error API for users of you APIs.
```go
func YourFunc() error
// what to return?
// Is error even necessary in the signature?
```
When you are a library author, you need to think about both.

### Handle the error you get from other APIs
```go
// type assertion vs. string search
// The errors in std libs are usually raw error type with a string value.
// It sucks but you have to do string search/match
switch err.Error() {
case "xxx":
    ...
case "xxx":
    ...
}
// The application 
```


## String Manipulation - Concurrent design without parallelism

## Functional Options (a form of meta programming) and Flexible, Extendable API

## Interface to the Extreme
