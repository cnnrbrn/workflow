# Lesson 4: Testing Asynchronous Code and API Calls with Vitest

## 1. Introduction to Asynchronous Testing

When building web applications, we often work with operations that don't complete immediately, such as making API calls or reading files. These are called asynchronous operations. Testing async code is crucial because:

1. It ensures our app handles real-world scenarios correctly.
2. It helps catch timing-related bugs.
3. It verifies our error handling for async operations.

However, testing async code can be challenging because:

1. Tests might finish before the async operation completes.
2. Async error handling can be complex.
3. We need to simulate time-consuming operations without actually waiting.

## 2. Understanding Async Functions

Let's start by looking at two simple async functions:

```javascript
// helpers.js
export async function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

export async function fetchDataWithDelay(data, ms) {
  await delay(ms);
  return data;
}
```

Let's break these down:

1. `delay(ms)`:

   - This function creates a Promise that resolves after a specified number of milliseconds.
   - It's async because it doesn't complete immediately; it waits for the specified time.
   - We use `setTimeout` to create the delay, wrapping it in a Promise for easier async/await usage.

2. `fetchDataWithDelay(data, ms)`:
   - This function simulates fetching data with a delay.
   - It's async because it uses the `await` keyword to wait for the `delay` function.
   - After the delay, it returns the provided data, simulating a delayed API response.

## 3. Basic Async Testing with Vitest

Now, let's test these functions:

```javascript
// helpers.test.js
import { expect, describe, it } from "vitest";
import { delay, fetchDataWithDelay } from "./helpers";

describe("Async Helper Functions", () => {
  it("delay function waits for the specified time", async () => {
    const start = Date.now();
    await delay(100);
    const end = Date.now();
    expect(end - start).toBeGreaterThanOrEqual(100);
  });

  it("fetchDataWithDelay returns data after delay", async () => {
    const data = { message: "Hello, World!" };
    const result = await fetchDataWithDelay(data, 100);
    expect(result).toEqual(data);
  });
});
```

Key points:

1. We use `describe` to group related tests together.
2. We mark our test functions as `async` to use `await` inside them.
3. For `delay`, we check if the elapsed time is at least the specified delay.
4. For `fetchDataWithDelay`, we verify that it returns the correct data after the delay.

## 4. Testing API Calls

Now, let's focus on testing a more realistic async function: the `register` function from the repo, which makes an API call:

```javascript
// auth.js
import { URL } from "../../constants/api.js";

export async function register(user) {
  const url = `${URL}auth/register`;

  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(user),
  };

  const response = await fetch(url, options);
  const json = await response.json();

  if (!response.ok) {
    throw new Error("Registration failed");
  }

  return json;
}
```

Let's break down this `register` function:

1. It's async because it uses `fetch`, which returns a Promise.
2. It sends a POST request to the registration endpoint with the user data.
3. It waits for the response with `await`.
4. It then parses the JSON response body with another `await`.
5. If the response is not ok (status is not in the 200-299 range), it throws an error.
6. If the response is ok, it returns the parsed JSON data.

When testing the `register` function, we need to check if it works correctly when the registration is successful and when it fails. However, we can't use the real `fetch` function in our tests. Instead, we use a fake version of `fetch` that we can control, a "mocked" version.

Mocking `fetch` is important for testing API calls because:

1. We can test our code without needing a working internet connection.
2. Our tests run faster because they don't wait for real server responses.
3. We can easily test what happens when the API returns errors.
4. We avoid accidentally creating real user accounts on the server during testing.

Here's how we use a mock `fetch` in our tests:

```javascript
// auth.test.js
import { expect, describe, it, vi } from "vitest";
import { register } from "./auth";
import { URL } from "../../constants/api.js";

describe("Authentication Functions", () => {
  // Create a fake fetch function
  global.fetch = vi.fn();

  it("register works correctly when API call is successful", async () => {
    const mockUser = {
      name: "Test User",
      email: "test@example.com",
      password: "password123",
    };
    const mockResponse = {
      id: 1,
      name: "Test User",
      email: "test@example.com",
    };

    // Make our fake fetch return a successful response
    fetch.mockResolvedValue({
      ok: true,
      json: () => Promise.resolve(mockResponse),
    });

    const result = await register(mockUser);

    // Check if fetch was called correctly
    expect(fetch).toHaveBeenCalledWith(
      `${URL}auth/register`,
      expect.objectContaining({
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(mockUser),
      })
    );

    // Check if the function returns the correct data
    expect(result).toEqual(mockResponse);
  });

  it("register throws an error when API call fails", async () => {
    // Make our fake fetch return a failed response
    fetch.mockResolvedValue({
      ok: false,
      json: () => Promise.resolve({ message: "Registration failed" }),
    });

    const mockUser = {
      name: "Test User",
      email: "test@example.com",
      password: "password123",
    };

    await expect(register(mockUser)).rejects.toThrow("Registration failed");
  });
});
```

What's happening in these tests:

- We create a fake `fetch` function using `vi.fn()`.
- We set up two test scenarios:
  1. Successful registration
  2. Failed registration
- For each scenario:
  - We configure our fake `fetch` to return a specific response.
  - We call the `register` function with test data.
  - We check if `fetch` was called correctly (for successful case).
  - We verify if the `register` function returns the expected result or throws an error.

Remember:

- Always test what happens when things work correctly and when they don't.
- Use the fake `fetch` to check if your function is calling the API correctly.
- When testing functions that use `await`, make sure to use `await` in your tests too.

By using these techniques, you can write good tests for your code that talks to APIs, making sure it works correctly in different situations.
