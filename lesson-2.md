# Lesson 2: Writing Effective Unit Tests with Vitest

## 1. Anatomy of a unit test

A unit test is a piece of code that checks if a small part of your application works correctly. In Vitest, a unit test usually follows this structure:

```javascript
import { expect, test } from "vitest";
import { functionToTest } from "./yourModule";

test("description of what the test is checking", () => {
  // Arrange: Set up the test data
  const input = "some input";
  const expectedOutput = "expected output";

  // Act: Call the function being tested
  const result = functionToTest(input);

  // Assert: Check if the result matches the expected output
  expect(result).toBe(expectedOutput);
});
```

Let's break this down:

1. **Import statements**: We bring in the tools we need from Vitest and the function we want to test.
2. **Test function**: We use `test()` to create a new test. We could also use `it()` instead of `test()` as they aliases. We give it a name that describes what we're testing.
3. **Arrange**: We set up the data we'll use in our test.
4. **Act**: We run the function we're testing.
5. **Assert**: We check if the result is what we expected.

## 2. Testing pure functions from the sample app

For this part of the lesson, we'll be using examples from this GitHub repository:

https://github.com/cnnrbrn/workflow-repo

The functions we'll be testing are located in the `src/js/utils/` directory of the repository.

### Testing the validateEmail function

You can find this function in the `src/js/utils/validation.js` file:

```javascript
export function validateEmail(email) {
  const emailRegex = /^[^\s@]+@(stud\.noroff\.no|noroff\.no)$/;
  return emailRegex.test(email);
}
```

This function checks if an email address is valid for Noroff students or staff. Here's what it does:

1. It takes an email address as input.
2. It uses a pattern to check if the email ends with either "stud.noroff.no" or "noroff.no".
3. It returns `true` if the email is valid (ends with one of these domains), and `false` otherwise.

This ensures that only people with Noroff email addresses can use the system.

Now, let's write tests for this function in `validation.test.js`:

```javascript
import { expect, describe, it } from "vitest";
import { validateEmail } from "./validation";

describe("validateEmail function", () => {
  it("returns true for valid student Noroff email", () => {
    const email = "student@stud.noroff.no";
    const result = validateEmail(email);
    expect(result).toBe(true);
  });

  it("returns true for valid Noroff staff email", () => {
    const email = "teacher@noroff.no";
    const result = validateEmail(email);
    expect(result).toBe(true);
  });

  it("returns false for non-Noroff email", () => {
    const email = "student@gmail.com";
    const result = validateEmail(email);
    expect(result).toBe(false);
  });

  it("returns false for invalid email format", () => {
    const email = "not-an-email";
    const result = validateEmail(email);
    expect(result).toBe(false);
  });
});
```

In these tests:

1. We group related tests using `describe`.
2. Each `it` function describes a specific behavior we're testing. We could also use `test()`.
3. We follow the Arrange-Act-Assert pattern in each test:
   - Arrange: Set up the email to test
   - Act: Call the validateEmail function
   - Assert: Check if the result is what we expect

This approach helps us thoroughly test our `validateEmail` function for different scenarios.

### Testing the validatePassword function

Now let's test the `validatePassword` function. Here's the function in `validation.js`:

```javascript
export function validatePassword(password) {
  return password.length >= 8;
}
```

And here are the tests in `validation.test.js`:

```javascript
import { validatePassword } from "./validation";

describe("validatePassword function", () => {
  const testCases = [
    { password: "short", expected: false },
    { password: "exactly8", expected: true },
    { password: "longerpassword", expected: true },
  ];

  testCases.forEach(({ password, expected }) => {
    it(`returns ${expected} for password "${password}"`, () => {
      const result = validatePassword(password);
      expect(result).toBe(expected);
    });
  });
});
```

In these tests:

1. We define an array of test cases, each with a password and the expected result.
2. We use `forEach` to run the same test for each case.
3. The test description changes based on the password and expected result.
4. We check if the function returns the expected result for each password.

This approach allows us to test multiple scenarios efficiently.

### Testing the validateForm function

Finally, let's test the `validateForm` function. Here's the function in `validation.js`:

```javascript
export function validateForm(email, password) {
  const errors = {};
  if (!validateEmail(email)) {
    errors.email = "Please enter a valid Noroff email address";
  }
  if (!validatePassword(password)) {
    errors.password = "Password must be at least 8 characters";
  }
  return {
    isValid: Object.keys(errors).length === 0,
    errors,
  };
}
```

And here are the tests in `validation.test.js`:

```javascript
import { validateForm } from "./validation";

describe("validateForm function", () => {
  const testCases = [
    {
      email: "valid@stud.noroff.no",
      password: "validpass",
      expected: { isValid: true, errors: {} },
    },
    {
      email: "invalid@gmail.com",
      password: "short",
      expected: {
        isValid: false,
        errors: {
          email: "Please enter a valid Noroff email address",
          password: "Password must be at least 8 characters",
        },
      },
    },
    {
      email: "valid@noroff.no",
      password: "short",
      expected: {
        isValid: false,
        errors: {
          password: "Password must be at least 8 characters",
        },
      },
    },
  ];

  testCases.forEach(({ email, password, expected }) => {
    it(`validates correctly for email "${email}" and password "${password}"`, () => {
      const result = validateForm(email, password);
      expect(result).toEqual(expected);
    });
  });
});
```

In these tests:

1. We define test cases with different combinations of email and password.
2. Each case includes the expected result (whether the form is valid and any error messages).
3. We use `forEach` to run the same test structure for each case.
4. We use `toEqual` instead of `toBe` because we're comparing objects.

This approach tests the `validateForm` function thoroughly, checking how it handles various combinations of valid and invalid inputs.
