# RetrofitFX
RetrofitFX aims to provide a library of utilities for using the [Retrofit framework](https://square.github.io/retrofit/)
with JavaFX. Retrofit is a type-safe HTTP client for Android and Java.

## ServiceInvocation

The main purpose of the `ServiceInvocation` class is to allow your application to perform a very elegant server call via 
Retrofit and to process the result on the JavaFX UI thread. This utility class also allows you to define all kinds of
handlers that will be invoked in various situations, e.g. when the server returns an error code such as 500. 

### Example

Example: assume that `myEndpoints` defines a method called `getAllCustomers()`:

```java
public interface MyService {
    
    @Get("customers/all-customers")
    public Call<List<Customer>> getAllCustomers();
}
```

Then you can create an implementation of `MyService` via `Retrofit` like this:

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.domain.com/")
    .build();

MyService service = retrofit.create(MyService.class);
```

You can now invoke the "all customers" endpoint using the `ServiceInvocation` class like this:

```java
ServiceInvocation.create("Load customers ...", () -> myService.getAllCustomers().execute())
        .onSuccess(customers -> {
            // here we are back on the JavaFX thread, now do something with the customers
        })
        .execute();
```

This code creates the invocation and executes it immediately. However, in most cases you would want to pass the newly
created invocation to the UI / the main window and execute it via a central `execute(ServiceInvocation)` method. This allows
the UI to observe the invocation and configure it properly (for example to show an error dialog whenever the call to the
endpoint failed).

### Handlers / Configuration

Handlers are invoked during the lifecycle of a service invocation. They might be called because the invocation has
started or finished or an exception has occurred or a result has been received. The following lists all available handlers.

* `withDelay()` - define a threshold in milliseconds before the call gets executed
* `withSimulatingFailure(boolean)` - intentionally make the call fail and trigger the onFailure() handler
* `onStart(Consumer<String>)` - call the consumer with the name of the service invocation
* `onSuccess(Consumer<T>)` - handler receiving the wrapped result object from the call
* `onSuccessDetailed(Consumer<Response<T>>)` - handler receiving the "raw" response object from the call
* `onAnyStatusCode(BiConsumer<String, HttpStatusCode)` - handler receiving the name of the call and the status code
* `onFailure(BiConsumer<String, String)` - handler receiving the name of the call and the error message returned from the server
* `onFailureDetailed(BiConsumer<String, Response<T>)` - handler receiving the name of the call and the raw response object
* `onStatusCode(HttpStatusCode, BiConsumer<String, String)` - handler that gets invoked when the call received the given status code
* `onException(BiConsumer<String, Exception)` - handler that gets invoked when an exeption gets thrown
* `onFinally(Runnable)` - handler that always gets called once the service invocation is done

### Default Handlers / Configuration

Default handlers are very useful when service invocations are create in different places of the application but executed
in a single central place. It allows the central execution place to add handlers without overriding handlers defined in
the place where the invocation was initially created.

* `onStartDefault(Consumer<String>)` - call the consumer with the name of the service invocation
* `onAnyStatusCodeDefault(BiConsumer<String, HttpStatusCode)` - handler receiving the name of the call and the status code
* `onFailureDefault(BiConsumer<String, String)` - handler receiving the name of the call and the error message returned from the server
* `onFailureDetailedDefault(BiConsumer<String, Response<T>)` - handler receiving the name of the call and the raw response object
* `onStatusCodeDefault(HttpStatusCode, BiConsumer<String, String)` - handler that gets invoked when the call received the given status code
* `onExceptionDefault(BiConsumer<String, Exception)` - handler that gets invoked when an exeption gets thrown
* `onFinallyDefault(Runnable)` - handler that always gets called once the service invocation is done

## Installation

These are the instructions on how you can add RetrofitFX to your project.


#### Using Maven

Add the following dependency to your pom.xml:
```xml
<dependency>
    <groupId>com.dlsc.retrofitfx</groupId>
    <artifactId>retrofitfx</artifactId>
    <version>1.0.0</version>
</dependency>
```

#### Using Gradle

Add the following dependency to your build.gradle:
<span id="gradle-dependency"></span>
```groovy
implementation 'com.dlsc.retrofitfx:retrofitfx:1.0.0'
```
