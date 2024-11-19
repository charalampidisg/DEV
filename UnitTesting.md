### Unit Testing Guidelines

While it is ideal to maintain unit tests for all branches, there are certain branches where working unit tests may not be present:

1. **Branches Without Working Unit Tests**:
   - `customer`
   - `merge`
   - `group/develop`

2. **Branches Requiring 100% Working Unit Tests**:
   - All other branches must ensure that all unit tests pass. It is important to validate your changes against existing tests and to add new tests as needed.

### **Test Suites**

Before running unit tests, especially for transport-related tests, ensure that the `omniNames` service is started with the specified options:

```sh
sudo omniNames -always -start 9000
```

Select the appropriate test suite based on the component you are working on:

- `SpiderRegressionTests` <-- `Spider`
- `HiveTestSuite` <-- `HiveMaster/Hiveling`
- `HiveWithSpiderTestSuite` <-- `Hive/Spider`
- `OpaTestSuite` <-- `OPA`
- `TestSteuer` <-- `Steuer`
- `TestEsc` <-- `ESC`
- `GridAdapterTestSuite` <-- `Grid`

### **Handling Existing Test Failures**

If you encounter any broken tests that were already present, try to fix them or notify the relevant team members.
