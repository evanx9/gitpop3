# GitPop3

[![Tests](https://github.com/AndreMiras/gitpop3/actions/workflows/tests.yml/badge.svg)](https://github.com/AndreMiras/gitpop3/actions/workflows/tests.yml)
[![Coverage Status](https://coveralls.io/repos/github/AndreMiras/gitpop3/badge.svg?branch=develop)](https://coveralls.io/github/AndreMiras/gitpop3?branch=develop)
[![Deploy](https://github.com/AndreMiras/gitpop3/actions/workflows/deploy.yml/badge.svg)](https://github.com/AndreMiras/gitpop3/actions/workflows/deploy.yml)

Find the most popular fork on GitHub.

<https://andremiras.github.io/gitpop3/>

GitPop3 helps you choose a fork when a project goes unmaintained.
It allows you to sort forks by "Stars", "Forks" or "Commits" count.
![Screenshot](https://i.imgur.com/4Ac311o.png)
See [GitPop2](https://github.com/AndreMiras/gitpop2) for the same tool using backend tech.

## Tech Stack

- **Frontend:** [Vite](https://vitejs.dev/) + [React](https://react.dev/) 17 (TypeScript), deployed to GitHub Pages
- **Routing:** `HashRouter` from react-router-dom v6 — supports deep-linkable URLs (e.g. `/#/django/django`)
- **Data fetching:** Apollo Client querying the GitHub GraphQL API v4 via a Cloud Function proxy
- **Error reporting:** Sentry
- **Infrastructure:** Google Cloud Platform, managed with Terraform

## Requirements

- Node.js 20.x or higher
- Yarn 1.22.0 or higher

## Setup

Clone the repo and install dependencies:

```sh
yarn install
```

Copy the example environment file:

```sh
cp .env.example .env
```

The key variable is `VITE_GRAPHQL_ENDPOINT`, which points to the GraphQL proxy (Cloud Function) that
injects the GitHub Personal Access Token and forwards requests to the GitHub API. The default value
in `.env.example` targets the production Cloud Function and works out of the box for local
development.

## Run

Start the Vite development server:

```sh
yarn dev
```

The app will be available at <http://localhost:3000>.

## Test

```sh
yarn lint
yarn test
```

To auto-fix formatting issues:

```sh
yarn format
```

## Deployment

The app is automatically deployed to GitHub Pages on every push to the `develop` branch via the
[Deploy workflow](.github/workflows/deploy.yml).

To deploy manually:

```sh
yarn deploy
```

## Cloud Function

The Cloud Function acts as a proxy between the frontend and the GitHub GraphQL API. It injects a
[Personal Access Token](https://docs.github.com/en/graphql/guides/forming-calls-with-graphql#authenticating-with-graphql)
(stored securely in GCP Secret Manager) into each request, so the PAT is never exposed to the
browser.

See the [serverless](serverless) folder for setup, local development, and testing instructions.

## Infrastructure

All infrastructure (Cloud Function, GCS state bucket, Secret Manager, IAM) is managed with
Terraform. See the [terraform](terraform) folder for details.
