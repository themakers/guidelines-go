# guidelines-go



# Rationale

// What is this?

# Golang

`//TODO: Add reference example`
`// Example of what?`

## General Requirements

#### The whole system should maintain maximum possible level of decomposition
regardless of the layer we’re talking about
// Sounds like a biblical covenant. What do you mean in particular?

#### Follow Package/Microdomain concept
`// TODO: explain`
`// What is this? I have no idea`

### Packages should not have singletons, global variables with modifiable values (except errors) or any other global state

* Scopes should have as few symbols as possible (especially package scope)
* Declarations should have the narrowest possible scope it could have // what does it mean?

#### Control-flow
Control-flow becomes more observable/understandable if it forms
*a directed graph with only one vertex which has no incoming (and only outgoing) edges*.
This vertex is `entry point`, the `main()` function.
`// WTF? Are you talking about entry points of programs here?`

#### Data-flow
Data-flow can also be considered as a graph. Every stateful object there is a vertex.
Fewer vertices, fewer and shorter edges => more observable/understandable teh data-flow is.
`// What are those graphs you're talking about? This concept exists only in auhtor's head `

In practice all the state should be packed into the package-level entities (structs). With this approach 
we also understand the ownership better (i.e.: this entity owns this logger instance, etc...). Storing state in closures is also acceptable.

## Error Handling

#### Functions like os.Exit() and log.Fatal() should never be used.
As these functions calls POSIX's `exit()` internally.
Which, in turn, makes any kind of deinitialization impossible.
`// I suggest to reconsider. These functions can be used in tests and entry points.`


#### All errors should be handled. Absolutely. Even in POCs.
In rare cases, if the error is non-critical, then it should be at least reliably reported. Error handling can be avoided only in cases like marshaling hard-coded JSON, locolized type assertion of a known value etc.

#### Errors should be typed
so we can use type switch to implement different recovery scenarios
`// Wrong. In my opinion, inline usage of errors.New is a good way to handle errors. At least in some cases.`


## Handlers (HTTP/gRPC/etc)

#### Handlers should be thin.
Most of the logic should be encapsulated into a subordinate entities.

In some cases it's even acceptable to use unexported functions:
```golang
func (Service) SomeHandler1(ctx context.Context, req Request) (resp Response, err error) {
    return someHandler1(ctx, req)
}

func (Service) SomeHandler2(ctx context.Context, req Request) (resp Response, err error) {
    return someHandler2(ctx, req)
}

// another file

func (Service) someHandler1(ctx context.Context, req Request) (resp Response, err error) {
    // ...
}

func (Service) someHandler2(ctx context.Context, req Request) (resp Response, err error) {
    // ...
}
```

#### Handlers should be bound to a structs (except some special cases). Also, these structs should have constructors.

```golang
type CarService struct {
    carStorage  storage.Cars
    log         zerolog.Logger
    debugMode   bool
}

type Options struct {
    // ...
}

func New(opts Options) *CarService {
    return &CarService{
        carStorage: opts.CarStorage,
        // ...
    }
}

func (cs *CarService) OnOff(ctx context.Context, req OnOffReq) (OnOffResp, error) {
    // ...
}

func (cs *CarService) BlowUp(ctx context.Context, req BlowUpReq) (BlowUpResp, error) {
    // ...
}

```

#### There should be one and only one place in domain's code from where you send responses
`// TODO: explain`
`// You definetly should. I have no idea what you're talking about`


## Observability

### Logging

#### All log messages with WARN level and above should have stacktraces attached

#### Structured logging approach should be implemented project-wide
`// TODO: explain`
`// Do you mean using something like log.Error().Str('token', token).Str('userID', req.UserID).Msg('Failed to make request')?`

[github.com/uber-go/zap](https://github.com/uber-go/zap)

`// what about zerolog?`

### Tracing. OpenTracing-compatible framework should be used.
`// TODO`

## Executable Package Design

#### Entry point should be as small as possible. It’s only a glue for subordinate entities.

```golang
package main

// ...

func main() {
    ctx, cancel := signal.NotifyContext(context.Background(), os.Interrupt)
	defer cancel()
    
    // log := logger
    // opts := something
    
    app := app.New(opts)
    
    go func() {
        defer cancel()
        log.Msg("Application is starting')
        if err := app.Start(ctx, opts); err != nil {
            log.Error().Err(err).Msg("Application failed")
        }
    }
    
    select {
    case <-ctx.Done():
        log.Msg("Application is stopping gracefully")
        time.Sleep(stopTimeout)
    }
    log.Msg("Application is stopped")
}
```

### Proper lifecycle management

#### All service-wide entities should be instantiated in the entry point and passed through the call stack
i.e.: logger, data access layer instance, etc


#### At the very beginning of the entry point we should have a global Context object instantiated
Later this object should be passed through the call stack accompanying the control flow.

This object should be nested and passed to every HTTP/gRPC/etc handler call using specific middlewares or `http.Server.BaseContext` callback in case of HTTP server.
    
#### Every service’s entry point should handle SIGINT|SIGTERM|SIGKILL
When the signal is caught, it should cancel the global context to initiate graceful shutdown

#### Graceful shutdown
The graceful shutdown should be implemented by every internal process.

During the graceful shutdown entry point should wait for them to finish (with timeout).

Graceful shutdown process should have a reasonable timeout, to not last forever.

## Library Package design

#### Architectural entities should represent real-world objects
`// TODO: explain`

#### Every entity should have a constructor defined as a package level function
If there is the only entity in a package, then the constructor function should be named `New`

If there are multiple entities in a package, then constructor functions should be named `New%EntityName%`

## Data modeling

#### Never describe/use same model for different domains
i.e.: DAO should not be used as a DTO

`// TODO: explain`

#### Maintain data converters in a separate package

`// TODO: explain`


## Automated Tests

#### Coverage should be 70 percent or higher

#### Tests should be coded in DDL-like style
to be easily readable/understandable

#### Apply Unit approach to simple stateless packages

#### Apply BDD-like approach to more complex stateful APIs
BDD-style tests fits better when it comes to testing stateful APIs (i.e. networked APIs),
because it has the state (i.e. user's session in networked APIs) and standardized working routines (or scenarios).
BDD tests also provides structured usage examples of your APIs.

[github.com/themakers/bdd](https://github.com/themakers/bdd)

## Documentation

### In-code comments
`// TODO`

#### No redundant comments
Code comments should be applied only when necessary. **Redundant == wrong**.

You don't need comments you have exhaustive naming and clean architecture. (IRL it is rather not 100% achievable)


## Glossary

* DDL -
* BDD -
* DAL - Data Access Layer
* DAO - Data Access Object
* DTO - Data Transfer Object
