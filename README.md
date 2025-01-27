# Unit Testing on Platform IO based ESP IDF

This codebase is a simple example on how to do automated testing directly in PlatformIO, for ESP IDF based development.

> The `main.c` is completely empty, but still there to emphasize that it is NOT NEEDED AT ALL to run Unit Tests. This is due to the spirit of testing is to test each individual component seperately, regardless of the `app_main()` logic. Hence, here we want to test out `AppLight.hpp`, which consists of an abstraction of the `LED_BUILTIN` on the DOIT ESP32 DEVKIT V1 (it will be on `GPIO2`).

## Get Started
- Install [VSCode](https://code.visualstudio.com/download)
- Install PlatformIO (PIO) Extension in VSCode
    - Go to `Extensions` on the left side bar, search for PlatformIO
- Choose which `env` (environment) you are trying to run
    - here we can see there are 3 types of environments that I have prepared
        1) `env:esp32`: application to build & flash. this is the default environment which is meant for development and production
        2) `env:unit_tests__xxxx`: unit testing suite/on desktop/native tests. these are an array of unit testing suites dedicated to for quick response tests. it is defined however you define through `test_filter` and `test_ignore`. DO NOT build this environment, it will do nothing. instead, it is dedicated for the `pio test -e unit_tests__xxxx -vvv` command, and all tests are expected to be run natively (obviously without needing for any development board plugged in because no flashing)
        3) `env:integration_tests__xxxx`: integration testing suite/on device tests. these are an array of integration tests expected to be long in response (need to flash first) tests. it is defined however you define through `test_filter` and `test_ignore`. DO NOT build this environment, it will do nothing. instead, it is dedicated for the `pio test -e unit_tests__xxxx -vvv` command, and will then do the flashing to the development board, and after it be flashed, the tests will run

            ![choose environment](docs/environment.png)

### Building & Flashing
- Open the `platform.ini` file, let PIO to load and install the required platform (espressif32@5.3.0)
    - This will take some time, because (if not yet) you need to download the whole choice of platform, in this case is `espressif32@5.3.0` (in which already has ESP IDF within). I had specified to use the version `5.3.0`, because I tried the current latest version, which is `6.2.0`, and it keeps on failing to compile with a certain error. Hence, I specified an older version which actually works
    - After the PIO Task of Downloading the platform is done, usually it does another PIO task, which is to "Configure project". Just wait for another while, after done, continue to the next step
- Open the PlatformIO tab, the image below will show up (if not yet, just try opening up the `platform.ini` file again, it sometimes fail to load)

    ![](docs/project-tasks.png)

    - `General` > `Build` Just wait until it successfully to Builds
        ![](docs/success-build.png)

### Testing
- `Advanced` > `Test`
    -  Just wait until this text shows up
    ```
        Testing...
        If you don't see any output for the first 10 secs, please reset board (press reset button)
    ```
    - Press the reset button on the ESP32, then the Unit Test will run, the `LED_BUILTIN` will flash really briefly, and this final success test message will popup
    
    ![](docs/success-test.png)

## A Guide on Unit Testing
### GoogleTest Testing Framework
- [A guide to using Google Test](https://google.github.io/googletest/primer.html)
    - `TEST()` -- a simple test, without fixtures
    - `TEST_F()` -- test with fixtures `When multiple tests in a test suite need to share common objects and subroutines, you can put them into a test fixture class.`
    - `testing::Test` -- [creating test fixtures](https://google.github.io/googletest/primer.html#same-data-multiple-tests)
        - `void SetUp() override {...}`
        - `void TearDown() override {...}`
    - running tests
        ```cpp
        // main.test.cpp
        int main(int argc, char **argv) {
            testing::InitGoogleTest(&argc, argv);
            return RUN_ALL_TESTS();
        }
        ```
    - fakes/mocks/stubs -- [gmock for dummies](https://google.github.io/googletest/gmock_for_dummies.html)

### Test Naming Conventions
- BDD Style: The BDD-style naming convention (should_do_x_when_being_done_y). This naming convention makes the test cases more readable and aligns with the BDD philosophy of focusing on the behavior of the system.

```cpp
#include <gtest/gtest.h>

// Test case using the BDD naming convention in snake_case
TEST(AddFunctionTest, should_return_sum_when_given_two_positive_numbers) {
    // Arrange
    int a = 5;
    int b = 10;

    // Act
    int result = Add(a, b);

    // Assert
    EXPECT_EQ(result, 15);
}
```

### Foldering
- `lib`: for all components that are expected to be tested, make sure are placed here. components in `src/` are by default NOT seen in the test suite
- `test`
    - `integration_tests`: all tests that will be tested after flash. I named it as integration tests, because it will take a while to execute and verify
    - `unit_tests`: all tests that will be run natively in the desktop and are expected to run FAST. tests that fall into this category fits the name unit test, knowing that it will be fired rapidly during every incremental development. all platform specific components (esp idf, arduino) MUST be mocked!

### Platform.ini breakdown
- common platform environments: add more platforms as needed
    ```
    ; -------- COMMON PLATFORM ENVIRONMENTS --------
    [env:esp32]
    ...

    [env:platform_name]
    ...
    ```

- unit test environments: a list of unit tests, you can tweak this to specify what tests should be run and not
    - to learn more, read [Platform IO Docs > Unit Testing > Test Hierarchy: Pizza Project Tests](https://docs.platformio.org/en/latest/advanced/unit-testing/structure/hierarchy.html)
    ```
    ; -------- UNIT TESTS ENVIRONMENTS --------

    [env:unit_tests__test_native]
    test_filter = unit_tests/* ; whitelist
    test_ignore = integration_tests/* ; blacklist

    [env:unit_tests__test_component_name_or_cluster]
    ...
    ```
- integration tests: a list of integrations tests. you can make different testing scenarios using real hardware with this
    - it is recommended to further integrate the integration testing with a CI system, and utilize a testing dock for semi-permanently connected devices for better a integration testing experience
    ```
    ; -------- INTEGRATION TESTS ENVIRONMENTS --------
    [env:integration_tests__test_esp32]

    ```

### Error Findings
- Solution to `Undefined Reference` Error: Place `.hpp` and `.cpp` of testable components within `{root}/lib`
    - another solution is to use [`test_build_src = yes`](https://docs.platformio.org/en/latest/projectconf/sections/env/options/test/test_build_src.html) inside `platform.ini`, but this is not so recommended especially when running a `platform = native` test, because all dependencies to `ESP-IDF` or `Arduino` will break and cause you a major headache! 🤮
    
        ![undefined reference](docs/undefined-reference.png)

    - source: [forum @ community.platformio.org](https://community.platformio.org/t/unit-tests-platformio-googletest-errors-linking/28529)

---
## Disclaimer
I did not write the code `AppLight`, it is from [PacktPublishing - Developing IoT Projects with ESP32 2nd edition](https://github.com/PacktPublishing/Developing-IoT-Projects-with-ESP32-2nd-edition) :octocat:. I made minor changes mainly to divert from the initial target which is directed for ESP32S3 to a ESP32