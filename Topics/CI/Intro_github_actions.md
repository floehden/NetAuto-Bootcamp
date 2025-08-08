# What is a GitHub Actions Pipeline?

A GitHub Actions pipeline is a continuous integration and continuous delivery (CI/CD) platform built directly into GitHub. It allows you to automate tasks in response to events in your repository, such as a `push` or `pull_request`. These automated tasks are defined in a **workflow**, which is a YAML file stored in your repository's `.github/workflows` directory.

A workflow is composed of one or more **jobs**, which are sets of steps that run in a virtual environment called a **runner**. Each job can run on its own virtual machine or within a container. The steps in a job can be simple shell commands or pre-built, reusable actions from the GitHub Marketplace.

-----

## Core Concepts and Terminology

  * **Workflow**: The YAML file that defines your automation process.
  * **Event**: The activity that triggers a workflow run, like a `push` to a branch or a `pull_request` being opened.
  * **Job**: A set of steps that execute on the same runner. Jobs can run in parallel or be configured to run sequentially based on dependencies.
  * **Runner**: A virtual machine that executes your jobs. GitHub provides hosted runners for various operating systems (Ubuntu, macOS, Windows). You can also use self-hosted runners.
  * **Step**: An individual task within a job. A step can be a shell script (`run`) or a reusable action (`uses`).
  * **Action**: A pre-built, reusable component that can be used in a step. The GitHub Marketplace has thousands of actions for common tasks like checking out code, setting up environments, and deploying applications.

-----
## Simple Node.js application

Create a GitHub repository for the Node.js application with the following files
I will create a simple Node.js application to demonstrate the use of a GitHub Actions pipeline. The example will include a basic Express.js server, a test file, and scripts for linting and building the project.

-----

### 1\. The Example Node.js Application

First, let's create a minimal Node.js project. You'll need a `package.json` file, a main application file, a test file, and a linting configuration.

#### **Project Structure**

Your project directory should look like this:

```
/my-node-app
|-- .github
|   |-- workflows
|       |-- main.yml
|-- .eslintrc.js
|-- index.js
|-- package.json
|-- package-lock.json
|-- test.js
```

-----

#### **`package.json`**

This file defines the project's metadata and scripts. It includes dependencies for `express`, `jest`, and `eslint`.

```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "description": "A simple Node.js app for GitHub Actions demo",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "jest",
    "lint": "eslint .",
    "build": "echo 'Build successful' && exit 0"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "eslint": "^8.49.0",
    "jest": "^29.7.0"
  }
}
```

  * **`start`**: Runs the application.
  * **`test`**: Executes the test suite using Jest.
  * **`lint`**: Runs `eslint` to check for code quality issues.
  * **`build`**: A placeholder script. In a real-world scenario, this might compile TypeScript, bundle assets, or minify code.

-----

#### **`index.js`**

This is a simple Express.js server that exposes a single endpoint.

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello, GitHub Actions!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});
```

-----

#### **`test.js`**

This file contains a basic test using Jest to ensure our application behaves as expected.

```javascript
describe('Placeholder Test', () => {
  it('should always pass', () => {
    expect(true).toBe(true);
  });
});
```

-----

#### **`.eslintrc.js`**

A minimal ESLint configuration to demonstrate the linting step.

```javascript
module.exports = {
  env: {
    browser: true,
    commonjs: true,
    es2021: true,
    node: true,
    jest: true
  },
  extends: 'eslint:recommended',
  parserOptions: {
    ecmaVersion: 12
  },
  rules: {}
};
```

Run npm install to generated the package-lock.json

## Simple Example Use Case: A Node.js CI Pipeline

Let's imagine you have a simple Node.js application and you want to automate the process of linting, testing, and building the code every time a developer pushes new code or opens a pull request.

The following YAML code, saved as `main.yml` in the `.github/workflows/` directory of your repository, sets up this pipeline.

```yaml
name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    
    - name: Use Node.js 20
      uses: actions/setup-node@v4
      with:
        node-version: 20
        cache: 'npm'

    - name: Install dependencies
      run: npm install

    - name: Run linting
      run: npm run lint

    - name: Run tests
      run: npm test

    - name: Build project
      run: npm run build
```

This workflow is triggered on every `push` or `pull_request` to the `main` branch. The **`build`** job runs on an `ubuntu-latest` runner and performs the following steps:

1.  **`actions/checkout@v4`**: This is a marketplace action that checks out your repository code into the runner's workspace.
2.  **`actions/setup-node@v4`**: This action installs the specified Node.js version (in this case, version 20) and configures `npm` caching to speed up subsequent runs.
3.  **`npm install`**: This step installs your project's dependencies.
4.  **`npm run lint`**: This runs your project's linting script. The workflow will fail if any linting errors are found.
5.  **`npm test`**: This runs your project's test suite. The workflow will fail if any tests fail.
6.  **`npm run build`**: This builds your project. This step only runs if the previous steps (linting and testing) pass successfully.

-----

### Common Challenges with GitHub Actions

While powerful, GitHub Actions can present a few challenges, especially as pipelines become more complex:

  * **Debugging**: When a workflow fails, it can be difficult to pinpoint the exact cause, especially with complex actions that have limited logging. Sometimes, you may need to add extra `run` steps to print out environment variables or debug logs.
  * **YAML Syntax and Expression Evaluation**: The YAML syntax can be sensitive to indentation and formatting. Additionally, understanding the different contexts and expressions (e.g., `github.event`, `steps.outputs`) can be tricky and lead to unexpected behavior.
  * **Passing Data Between Jobs**: Each job runs on a separate runner, which means there's no shared file system or memory. To pass data (like a build artifact) from one job to another, you must use the `actions/upload-artifact` and `actions/download-artifact` actions.
  * **Security and Secrets Management**: Managing secrets across different environments (e.g., staging, production) requires careful configuration using GitHub's built-in secrets and environment protection rules. Exposing secrets in logs or insecurely in your workflow can be a major security risk.
  * **Concurrency**: If not managed carefully, multiple concurrent workflow runs can cause resource contention or race conditions when interacting with external services. The `concurrency` keyword can be used to group workflow runs and ensure only one runs at a time.

This video provides a great overview of how to get started with GitHub Actions and build your first CI/CD pipeline.

[Writing Your First CI/CD Pipeline on Github Actions](https://www.youtube.com/watch?v=Lj59I0NO7J8)
