# Contributing to GitPop3

Thank you for your interest in contributing! This guide covers everything you need to get started.

## Prerequisites

- [Node.js](https://nodejs.org/) 20.x or higher
- [Yarn](https://yarnpkg.com/) 1.22.0 or higher

## Clone & Set Up

```sh
git clone https://github.com/AndreMiras/gitpop3.git
cd gitpop3
yarn install
```

Copy the example environment file and fill in any required values:

```sh
cp .env.example .env
```

The key variable is `VITE_GRAPHQL_ENDPOINT`, which points to the GraphQL proxy that forwards requests to the GitHub API. The default value in `.env.example` targets the production Cloud Function and works out of the box for local development.

## Run the App

Start the development server:

```sh
yarn dev
```

The app will be available at <http://localhost:5173>.

## Run Tests & Linting

```sh
# Run the full test suite with coverage
yarn test

# Check code formatting
yarn lint

# Auto-fix formatting issues
yarn format
```

All checks must pass before a pull request can be merged.

## Opening a Pull Request

1. Fork the repository and create a feature branch off `develop`:
   ```sh
   git checkout develop
   git checkout -b my-feature
   ```
2. Make your changes, keeping commits focused and descriptive.
3. Ensure `yarn lint` and `yarn test` both pass locally.
4. Push your branch and open a pull request against the `develop` branch.
5. Describe **what** changed and **why** in the PR description.

## Code Style

This project uses [Prettier](https://prettier.io/) for formatting. Run `yarn format` to auto-format your code before committing. The configuration lives in `.prettierrc.json`.
