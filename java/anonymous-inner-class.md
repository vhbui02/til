# Anonymous Inner Class

Before:

```java
// Step 1
public class MyCustomWrapper extends HttpServletRequestWrapper {
  public MyCustomWrapper(HttpServletRequest request) {
    super(request);
  }

  @Override
  public String getHeader(String name) {
    // ... custom logic
  }
}

// Step 2
HttpServletRequest wrappedRequest = new MyCustomWrapper(request);
```

After:

```java
HttpServletRequest wrappedRequest = new MyCustomWrapper(request) {
  @Override
  public String getHeader(String name) {
    // ... custom logic
  }
}
```
