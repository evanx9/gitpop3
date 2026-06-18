# About GitPop3

## What Is It?

**GitPop3** is a client-side React web application that helps you find the most popular fork of any GitHub repository. It is especially useful when an upstream project goes unmaintained and you need to identify which fork has the most activity or community traction.

Live app: <https://andremiras.github.io/gitpop3/>

> See also [GitPop2](https://github.com/AndreMiras/gitpop2) — the same concept implemented with a traditional backend.

---

## Core Functionality

1. **Enter a GitHub repository URL** (e.g. `https://github.com/owner/repo`).
2. GitPop3 queries the **GitHub GraphQL API** for the repository and its top 100 forks (ordered by stars).
3. Results are displayed in a **sortable, paginated table** with the following columns:
   - **Repo** — `owner/name` linked to GitHub
   - **Stars** — stargazer count
   - **Forks** — fork count
   - **Commits** — total commit count on the default branch
   - **Modified** — date of the last commit on the default branch
4. The original repository is included in the list alongside its forks.
5. The current search is reflected in the **URL** (e.g. `/#/owner/repo`), so results are shareable and bookmarkable.

---

## Architecture

### Frontend (`src/`)

| File / Directory | Purpose |
|---|---|
| `src/App.tsx` | Root component; sets up React Router, FontAwesome icon library, and Sentry |
| `src/components/Container.tsx` | Orchestrates search state, URL routing, and renders child components |
| `src/components/PopForm.tsx` | Search form — accepts a GitHub URL and triggers the fork lookup |
| `src/components/ResultTable.tsx` | Sortable, paginated table of forks |
| `src/components/ForkLine.tsx` | Single row in the results table |
| `src/components/Navigation.tsx` | Top navigation bar |
| `src/components/Footer.tsx` | Page footer |
| `src/components/ErrorDialog.tsx` | Displays API or validation errors |
| `src/utils/graphql.ts` | Apollo Client setup + `GET_FORKS_QUERY` GraphQL query |
| `src/utils/search.ts` | `searchPopularForks()` — calls the GraphQL API and merges origin repo with forks |
| `src/utils/validators.ts` | `splitUrl()` — parses and validates a GitHub URL into `[owner, repo]` |
| `src/utils/time.ts` | Date formatting helpers |
| `src/utils/sentry.ts` | Sentry error-reporting initialisation |
| `src/utils/types.ts` | Shared TypeScript types (`Node`, `Repository`, `Result`) |

**Tech stack:** React 17, TypeScript, Vite, Apollo Client (GraphQL), React Bootstrap, React Router v6, FontAwesome, Sentry, Vitest.

### Serverless Proxy (`serverless/`)

The GitHub GraphQL API requires authentication. Rather than exposing a Personal Access Token (PAT) in the browser bundle, GitPop3 routes all GraphQL requests through a **Google Cloud Function** that:

- Validates the `Origin` header against an `ALLOWED_ORIGINS` allowlist (CORS).
- Injects the `GITHUB_PAT` bearer token before forwarding the request to `https://api.github.com/graphql`.
- Returns the GitHub response to the browser.

The frontend points to this proxy via the `VITE_GRAPHQL_ENDPOINT` environment variable (defaults to the production Cloud Function URL in `.env.example`).

### Infrastructure (`terraform/`)

The Cloud Function and its supporting resources (GCS bucket for Terraform state, Secret Manager secret for the PAT) are managed with **Terraform** targeting **Google Cloud Platform**.

Key Terraform resources:
- `google_cloudfunctions2_function` — the serverless proxy
- `google_storage_bucket` — Terraform remote state storage
- `google_secret_manager_secret` — stores the GitHub PAT securely

Deployment is driven by `make devops/terraform/plan` / `make devops/terraform/apply`.

---

## Data Flow

```
Browser
  │
  │  HTTP POST (GraphQL query, no auth header)
  ▼
Google Cloud Function  (serverless/src/)
  │  adds Authorization: bearer <GITHUB_PAT>
  ▼
GitHub GraphQL API  (api.github.com/graphql)
  │  returns repository + top-100 forks
  ▼
Google Cloud Function  (strips nothing, proxies response)
  │
  ▼
Browser  →  ResultTable renders sortable fork list
```

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **GraphQL over REST** | A single query fetches the repo and its top-100 forks with all required fields in one round-trip. |
| **Serverless auth proxy** | Keeps the GitHub PAT off the client without requiring a persistent backend server. |
| **Client-side sorting** | All 100 forks are fetched at once; sorting/pagination is done in-browser with Lodash for instant feedback. |
| **URL-based routing** | React Router encodes the searched repo in the hash URL, making results shareable. |
| **Vite + Vitest** | Replaced Create React App for faster builds and a modern test runner with native ESM support. |

---

## Project Layout

```
gitpop3/
├── src/                  # React frontend (TypeScript)
│   ├── components/       # UI components
│   └── utils/            # GraphQL client, validators, helpers
├── serverless/           # Google Cloud Function (auth proxy)
├── terraform/            # GCP infrastructure as code
├── public/               # Static assets
├── .env.example          # Environment variable template
├── vite.config.mts       # Vite build configuration
├── tsconfig.json         # TypeScript configuration
└── Makefile              # Developer & DevOps convenience targets
```

---

## Environment Variables

| Variable | Where used | Description |
|---|---|---|
| `VITE_GRAPHQL_ENDPOINT` | Frontend (Vite) | URL of the GraphQL proxy (Cloud Function) |
| `ALLOWED_ORIGINS` | Cloud Function | Comma-separated list of allowed CORS origins |
| `GITHUB_GRAPHQL_API_URL` | Cloud Function | GitHub GraphQL endpoint to proxy to |
| `GITHUB_PAT` | Cloud Function | GitHub Personal Access Token (`public_repo` scope) |

---

## Running Locally

```sh
# 1. Install dependencies
yarn install

# 2. Configure environment
cp .env.example .env   # VITE_GRAPHQL_ENDPOINT already points to production proxy

# 3. Start the dev server
yarn dev               # http://localhost:5173

# 4. Run tests & linting
yarn test
yarn lint
```

---

## Deployment

| Target | Command |
|---|---|
| GitHub Pages (frontend) | `yarn deploy` |
| Cloud Function + infra | `make devops/terraform/apply` |

---

## Versioning & History

The project uses **calendar versioning** (`YYYY.MM.DD`). Notable milestones:

- **2020-12-10** — Initial UI release
- **2020-12-19** — Sortable columns; last-commit-date bug fix
- **2025-10-25** — Migrated to Vite + TypeScript; added serverless auth proxy; React Router URL navigation; Prettier formatting; Sentry error reporting; Node 20

---

## License

MIT — see [LICENSE](LICENSE).
