# Lesson 1: Introduction to Unit Testing and Vitest

## 1. What is Unit Testing?

Unit testing is a way to check if small parts of your code work correctly. Imagine you're building a big puzzle. Unit testing is like making sure each puzzle piece is the right shape before you try to put the whole puzzle together.

In programming, a "unit" is usually a single function or a small piece of code that does one specific job. When we write unit tests, we:

1. Run the code
2. Check if it gives us the result we expect

For example, if you have a function that adds two numbers, a unit test would check if `add(2, 3)` correctly returns `5`.

## 2. Why is Unit Testing Important?

Unit testing helps us in many ways:

1. **Finds bugs early:** We can catch mistakes in our code before they cause bigger problems.
2. **Makes changes easier:** When we want to improve our code, tests help us make sure we didn't break anything.
3. **Improves code quality:** Writing tests often leads to better, cleaner code.
4. **Serves as documentation:** Tests show how our code should work, which helps other developers understand it.
5. **Saves time:** While writing tests takes time at first, it saves a lot of time finding and fixing bugs later.

## 3. Introduction to Vitest

Vitest is a tool that helps us write and run unit tests. It's made to work well with Vite, a popular tool for building web applications, but it can also be used in other JavaScript projects.

Vitest is:

- Fast: It runs tests quickly, which is great when you have many tests.
- Easy to use: It has a simple way of writing tests that's easy to learn.
- Feature-rich: It includes many helpful features for different testing needs.

## 3.5 Vitest vs Jest

Now that we know about Vitest, let's compare it to another popular testing tool called Jest.

### What is Jest?

Jest is a well-known testing tool created by Facebook in 2011. Many developers have been using Jest for years to test their JavaScript code, especially in React projects.

### How are Vitest and Jest similar?

1. **Purpose:** Both Vitest and Jest are used for testing JavaScript code.
2. **Syntax:** They use very similar ways of writing tests. If you know how to write Jest tests, you'll find Vitest familiar.
3. **Features:** Both offer features such as:
   - Mocking: This lets you replace parts of your code with fake versions for testing.
   - Code coverage: This shows you how much of your code is being tested.
   - Ability to run specific tests: You can choose to run only certain tests instead of all of them.

### How are they different?

1. **Speed:** Vitest is usually faster, especially for big projects.
2. **Configuration:** Vitest often needs less setup, especially in Vite projects.
3. **ESM Support:** Vitest works better with ECMAScript Modules (ESM), which is the modern way of organizing JavaScript code.

   ESM is a way to organize and share code between different JavaScript files uing`import` and `export` statements.

   Vitest supports this modern syntax out of the box. This means you can write your tests using `import` statements without any extra setup:

   ```javascript
   // math.test.js
   import { expect, test } from "vitest";
   import { add } from "./math.js";

   test("adds 2 + 3 to equal 5", () => {
     expect(add(2, 3)).toBe(5);
   });
   ```

   With Vitest, you can use ESM in your tests just like you do in your regular code. This makes it easier to test modern JavaScript projects without any extra steps or configuration.

### Can Jest tests be used with Vitest?

Yes, in most cases. Vitest was designed to be compatible with Jest. This means:

1. You can often run your Jest tests with Vitest without changing them.
2. If you're moving from Jest to Vitest, you usually don't need to rewrite all your tests.

However, some advanced Jest features might need small changes to work with Vitest.

### Which one should you choose?

- If you're starting a new project, especially with Vite, Vitest is a great choice.
- If you're working on an existing project that uses Jest, it's usually fine to stick with Jest.
- If you want faster tests and easier configuration, consider switching to Vitest.

Remember, both Jest and Vitest are good tools. The best choice depends on your project's needs.

## 4. Setting up Vitest

We can set up Vitest in two main ways:

### a. In a Vite project

1. Create a new Vite project:

   ```bash
   npm create vite@latest my-vite-project -- --template vanilla
   cd my-vite-project
   ```

2. Install Vitest as a dev dependency (-D):

   ```bash
   npm install -D vitest
   ```

3. Add a test script to `package.json`:
   ```json
   "scripts": {
     "test": "vitest"
   }
   ```

### b. In a regular JavaScript project (no bundler)

1. Create a new project folder:

   ```bash
   mkdir my-js-project
   cd my-js-project
   ```

2. Start a new npm project:

   ```bash
   npm init -y
   ```

3. Install Vitest:

   ```bash
   npm install -D vitest
   ```

4. Add a test script to `package.json`:
   ```json
   "scripts": {
     "test": "vitest"
   }
   ```

## 5. Writing Your First Test

Let's write a simple test for a function that adds two numbers:

1. Create a file called `math.js`:

   ```javascript
   export function add(a, b) {
     return a + b;
   }
   ```

2. Create a test file called `math.test.js`:

   ```javascript
   import { expect, test } from "vitest";
   import { add } from "./math.js";

   test("adds 1 + 2 to equal 3", () => {
     expect(add(1, 2)).toBe(3);
   });
   ```

3. Run the test:
   ```bash
   npm test
   ```

If everything is set up correctly, you should see a message saying the test passed!

## Conclusion

In this lesson, we learned what unit testing is, why it's important, and how to set up Vitest for our projects. We also wrote our first simple test. In the next lesson, we'll dive deeper into writing different kinds of tests for a web application.
