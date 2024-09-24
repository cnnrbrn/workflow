# Lesson 3: Mocking in Vitest

## 3.1 Mocking Browser APIs

Browser APIs are built-in tools in web browsers that allow web pages to perform various tasks. Common examples include:

- DOM (Document Object Model) API
- Web Storage API (localStorage, sessionStorage)
- Fetch API
- Geolocation API
- Canvas and WebGL for graphics

### Why Mock Browser APIs?

When testing JavaScript code that uses browser APIs in a Node.js environment, we need to create mock implementations because these APIs are not available in Node.js. We mock browser APIs for several reasons:

1. To test browser-specific code in a Node.js environment
2. To control API behavior for different test scenarios
3. To ensure consistent test results across environments

### Example: Mocking localStorage

Let's look at an example of how we might mock `localStorage` for our tests. We'll use these functions from the storage utility file in the repo:

```javascript
// js/utils/storage.js
const tokenKey = "token";

export function saveToken(token) {
  saveToStorage(tokenKey, token);
}

export function getToken() {
  return getFromStorage(tokenKey);
}

export function clearStorage() {
  localStorage.clear();
}

function saveToStorage(key, value) {
  localStorage.setItem(key, JSON.stringify(value));
}

function getFromStorage(key) {
  const value = localStorage.getItem(key);
  return value ? JSON.parse(value) : null;
}
```

Now, let's write tests for these functions, creating a mock `localStorage`:

```javascript
// js/utils/storage.test.js
import { expect, describe, it, beforeEach, vi } from "vitest";
import { saveToken, getToken, clearStorage } from "./storage";

describe("Storage functions", () => {
  beforeEach(() => {
    // Create a mock localStorage before each test
    const localStorageMock = {
      getItem: vi.fn(),
      setItem: vi.fn(),
      clear: vi.fn(),
    };
    global.localStorage = localStorageMock;
  });

  it("saves a token", () => {
    const testToken = "test-token";
    saveToken(testToken);
    expect(localStorage.setItem).toHaveBeenCalledWith(
      "token",
      JSON.stringify(testToken)
    );
  });

  it("retrieves a token", () => {
    const testToken = "test-token";
    localStorage.getItem.mockReturnValue(JSON.stringify(testToken));
    const retrievedToken = getToken();
    expect(retrievedToken).toBe(testToken);
  });

  it("clears storage", () => {
    clearStorage();
    expect(localStorage.clear).toHaveBeenCalled();
  });
});
```

In these tests:

1. We create a mock `localStorage` object with mock functions for `getItem`, `setItem`, and `clear`.
2. We assign this mock to `global.localStorage`, making it available globally in our tests.
3. We use `vi.fn()` to create mock functions that we can track and control.
4. We set up the mock before each test in the `beforeEach` hook.
5. In our tests, we can set return values for our mock functions and verify that they were called correctly.

This approach allows us to test code that uses `localStorage` in a Node.js environment where `localStorage` doesn't exist.

### What Happens If We Don't Mock localStorage?

If we don't mock `localStorage` when running our tests in a Node.js environment, we'll encounter errors:

1. **ReferenceError**: Node.js will throw a `ReferenceError` because `localStorage` is not defined in the global scope.
2. **Test Failures**: All tests involving `localStorage` will fail due to the missing API.
3. **Inability to Test**: We won't be able to verify if our code is correctly interacting with `localStorage`.

For example, without mocking, a simple test like this would fail:

```javascript
it("saves a token", () => {
  saveToken("test-token");
  // This line would throw an error before we even get to the assertion
});
```

By mocking `localStorage`, we avoid these errors and can focus on testing our code's logic and its interaction with `localStorage`.

## 3.2 Using jsdom with Vitest

While manually mocking `localStorage` works, jsdom provides a more complete simulation of a browser environment in Node.js. This includes `localStorage` and many other browser APIs, making our tests more realistic and reducing the need for manual mocking.

### What is jsdom?

jsdom is a pure JavaScript implementation of many web standards for use with Node.js. It simulates a browser-like environment, allowing you to run browser-specific code in Node.js.

### Setting up jsdom with Vitest

1. First, install the necessary packages:

   ```bash
   npm install -D jsdom @vitest/browser
   ```

2. Update your Vitest configuration. If you're using a `vite.config.js` file, add the following:

   ```javascript
   import { defineConfig } from "vite";

   export default defineConfig({
     test: {
       environment: "jsdom",
     },
   });
   ```

   If you're using a separate `vitest.config.js`, the configuration would be similar.

3. Now, let's rewrite our tests using jsdom:

   ```javascript
   // js/utils/storage.test.js
   import { expect, describe, it, beforeEach, afterEach } from "vitest";
   import { saveToken, getToken, clearStorage } from "./storage";

   describe("Storage functions", () => {
     beforeEach(() => {
       // Clear localStorage before each test
       localStorage.clear();
     });

     afterEach(() => {
       // Clear localStorage after each test
       localStorage.clear();
     });

     it("saves and retrieves a token", () => {
       const testToken = "test-token";
       saveToken(testToken);
       expect(localStorage.getItem("token")).toBe(JSON.stringify(testToken));

       const retrievedToken = getToken();
       expect(retrievedToken).toBe(testToken);
     });

     it("returns null for non-existent token", () => {
       const result = getToken();
       expect(result).toBeNull();
     });

     it("clears storage", () => {
       saveToken("test-token");
       clearStorage();
       expect(localStorage.getItem("token")).toBeNull();
     });
   });
   ```

### Comparing Manual Mocking and jsdom Approaches

You may have noticed that our tests look different when using jsdom compared to manual mocking. Let's briefly explain why:

1. **Setup and Teardown**: With manual mocking, we create a mock `localStorage` object. With jsdom, we clear the real `localStorage` before and after each test.

2. **Testing Strategy**: Manual mocking tests function calls, while jsdom tests actual state changes in `localStorage`.

3. **Assertions**: Manual mocking uses `expect().toHaveBeenCalledWith()` to check function calls. jsdom uses `expect().toBe()` or `expect().toBeNull()` to check actual values.

4. **Test Granularity**: Manual mocking often has separate tests for each operation. jsdom allows testing multiple operations in a single test more easily.

5. **Realism**: jsdom tests are closer to how the code would behave in a real browser environment.

For example, in manual mocking, we might test saving a token like this:

```javascript
it("saves a token", () => {
  const testToken = "test-token";
  saveToken(testToken);
  expect(localStorage.setItem).toHaveBeenCalledWith(
    "token",
    JSON.stringify(testToken)
  );
});
```

With jsdom, the same test might look like this:

```javascript
it("saves and retrieves a token", () => {
  const testToken = "test-token";
  saveToken(testToken);
  expect(localStorage.getItem("token")).toBe(JSON.stringify(testToken));

  const retrievedToken = getToken();
  expect(retrievedToken).toBe(testToken);
});
```

The jsdom approach allows us to test the full cycle of operations more easily and realistically.

### Benefits and Considerations of Using jsdom

Benefits:

1. Realistic browser-like environment (includes `window`, `document`, etc.)
2. Reduces need for manual mocking of browser APIs
3. Tests are closer to real browser behavior
4. Simpler interaction with DOM and browser APIs in tests

Considerations:

1. May slow down test execution for large test suites
2. Doesn't implement all browser features
3. Behavior may differ slightly from real browsers

Using jsdom simplifies our tests and makes them more robust, closely mimicking a real browser environment without the need for extensive manual mocking.

# 3.3 Testing DOM Manipulation Functions

Now that we've covered mocking browser APIs and using jsdom, let's apply these concepts to test functions that directly manipulate the DOM. We'll use the `updateMainHeading` function from the repo as an example.

Here's the function we'll be testing:

```javascript
export function updateMainHeading(newHeading) {
  const heading = document.querySelector("h1");
  if (heading) {
    heading.textContent = newHeading;
  }
}
```

This function updates the text content of the first `<h1>` element on the page. If no `<h1>` element exists, it does nothing.

## Setting Up the Test Environment

First, make sure you have jsdom set up in your Vitest configuration as discussed earlier in this lesson. Then, create a new test file and write the tests:

## Writing Tests for updateMainHeading

Now, let's write tests for our `updateMainHeading` function:

```javascript
// File: js/ui/common/updateMainHeading.test.js
import { expect, describe, it, beforeEach, afterEach } from "vitest";
import { updateMainHeading } from "./common";

describe("updateMainHeading function", () => {
  beforeEach(() => {
    // Set up a mock DOM environment before each test
    document.body.innerHTML = "<h1>Original Heading</h1>";
  });

  afterEach(() => {
    // Clean up the mock DOM after each test
    document.body.innerHTML = "";
  });

  it("updates the content of the h1 element", () => {
    updateMainHeading("New Heading");
    const heading = document.querySelector("h1");
    expect(heading.textContent).toBe("New Heading");
  });

  it("does nothing if no h1 element exists", () => {
    document.body.innerHTML = ""; // Remove the h1 element
    expect(() => updateMainHeading("New Heading")).not.toThrow();
  });
});
```

Let's break down these tests:

1. **Test Setup**: We use `beforeEach` to set up our mock DOM before each test. This ensures each test starts with a clean, consistent DOM state.

2. **Test Teardown**: We use `afterEach` to clean up the DOM after each test. This prevents tests from interfering with each other.

3. **Testing the Happy Path**: Our first test checks if the function correctly updates the heading when an `<h1>` element exists.

4. **Testing Edge Cases**: Our second test verifies that the function doesn't throw an error when there's no `<h1>` element to update.

## Key Points for DOM Testing

1. **Clean Tests**: Set up a fresh DOM before each test and clean up after. This keeps tests independent.

2. **Test Everything**: Check both normal cases (updating a heading) and unusual cases (no heading present).

3. **Keep It Simple**: Write clear, easy-to-understand tests. This helps everyone, especially beginners.

4. **Close to Real**: jsdom gives us a browser-like environment, but it's not exactly the same as a real browser.

5. **Speed Considerations**: DOM tests might run slower than regular JavaScript tests.

Remember these points when testing DOM functions. They'll help you write better tests and catch potential issues in your web applications.
