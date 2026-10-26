# py-graders-ci

The reusable GitHub Actions workflow that grades BatSchool Python course
G-projects. It holds no tests: it runs the public grader image
`ghcr.io/bondiano/py-grader-<craft>:<stage>` against your checkout next to the
services the stage needs, and reports the result to the school with a GitHub
Actions OIDC token. The school accepts reports only from this workflow at a
published tag and only from a repository you linked in your school settings.

## Use it

1. Sign in to the school with GitHub, then link your public repository on the
   settings page (the repository owner must be that same GitHub account).
2. Add `.github/workflows/grade.yml` to your repository:

```yaml
name: grade
on:
  push:
  workflow_dispatch:

# The called workflow needs these; a caller cannot grant less.
permissions:
  id-token: write
  contents: read

jobs:
  grade:
    uses: bondiano/py-graders-ci/.github/workflows/grade.yml@v1
    with:
      craft: ormka
      stage: '12'
      services: none   # none | postgres | redis | postgres,redis
```

3. Push. The job log lists every test, the job summary shows the score, and a
   green run is credited as a passed stage on the school side.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `craft` | required | craft id, e.g. `ormka` |
| `stage` | required | stage number |
| `services` | `none` | service containers: `postgres` (`localhost:5432`, user and password `postgres`), `redis` (`localhost:6379`); the grader gets `DATABASE_URL` and `REDIS_URL` |
| `platform-url` | `https://school.bondiano.com/api/code/ci-report` | report endpoint |
| `audience` | `https://school.bondiano.com` | OIDC audience |

## What runs

The workflow copies your checkout to a temporary directory, runs the grader
with it mounted at `/work` (no network unless services are requested,
read-only root, 2 CPU, 2 GiB), parses the one-line-per-test JSONL report and
posts `{task, sha, tests}` with the OIDC token. The task slug comes from the
grader image label, not from the inputs. Third-party actions are pinned by
commit SHA.
