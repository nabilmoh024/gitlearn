# GitHub Actions: Beginner-to-Practical Hands-On Workbook

**Audience:** No coding background required. **Platform:** GitHub website first; VS Code and Git Bash optional later. **Cost:** First 12 labs need no AWS resources; GitHub Actions usage and quotas depend on your repository and plan. **Source:** User-supplied transcript of DevOps Directive's complete GitHub Actions course (3h42m), video https://www.youtube.com/watch?v=Xwpi0ITkL3U . Video timestamps are provided for cross-reference. This workbook adds original beginner exercises; they are not verbatim from the video.

## How to study

Spend 20–35 minutes per lab. Read the analogy, type or paste the YAML, predict what will happen, run it, inspect logs, and answer the checkpoint. Use a **new personal practice repository**, not an employer's production repository. Do not add real AWS credentials, passwords, or proprietary code. GitHub's UI labels can change.

### Learning map

| Lab | Topic | Video location | Deliverable |
|---|---|---|---|
| 0 | Git/GitHub/YAML vocabulary | 19:04, 24:00 | Practice repository |
| 1 | First manual workflow | 24:00 | Green successful run |
| 2 | Multiple steps and failure | 27:32 | Explain logs and exit codes |
| 3 | Parallel jobs and `needs` | 29:33 | Sequential dependency graph |
| 4 | Push and pull request triggers | 30:55 | Automatic run on change |
| 5 | Environment variables and contexts | 36:22–44:20 | Scoped configuration |
| 6 | Step and job outputs | 37:54–40:23 | Data between jobs |
| 7 | Secrets, variables, permissions | 40:23–44:20, 57:44–1:04:20 | Safe configuration demo |
| 8 | Artifacts vs caches | 49:21–54:21 | Downloadable report |
| 9 | Conditions and matrix | 1:04:20 | Parallel versions |
| 10 | Marketplace actions | 1:07:17 | Checkout action |
| 11 | Reusable workflows | 1:13:34–1:18:57 | One called workflow |
| 12 | Debugging and best practices | 1:40:45–2:06:30 | Troubleshooting playbook |
| 13 | Build/test/push architecture | 1:31:37, 2:06:35–3:08:43 | Paper CI/CD design |
| 14 | GitOps, releases, observability | 3:08:43–3:33:39 | Architecture sketch |
| 15 | AWS deployment architecture | 1:00:00–1:04:20, 1:34:37 | Deployment design, no AWS spend |

## Mental model: a restaurant kitchen

GitHub **repository** = recipe book; **event** = customer order; **workflow** = kitchen's overall preparation plan; **job** = cooking station; **step** = one instruction; **runner** = the actual kitchen where instructions run; **action** = a reusable appliance. A **YAML file** is the written plan, not the kitchen itself. GitHub Actions executes instructions in jobs when a configured event occurs.

```
GitHub event (push / PR / manual)
       |
       v
Workflow (.github/workflows/example.yml)
       |
       +---- Job A (runner A): step 1 -> step 2
       |
       +---- Job B (runner B): step 1 -> step 2
                    |
                    v
               Job C (needs: [A, B])
```

**CI** means continuously checking that changes are healthy (e.g. automated tests). **CD** can mean continuously preparing releases or automatically deploying them. **Commit** = saved version of changes; **branch** = separate line of work; **pull request (PR)** = proposal to merge changes; **merge** = integrate changes; **artifact** = produced file; **cache** = reused temporary dependencies. An **action** is a reusable step, whereas **GitHub Actions** is the automation platform.

### YAML survival guide

```yaml
name: My Example       # key: value; # starts a comment
on:                   # an event configuration
  workflow_dispatch:  # two-space indentation nests configuration
jobs:                 # a group of named jobs
  demo:               # job identifier
    runs-on: ubuntu-latest
    steps:             # list of steps
      - name: Say hello # dash introduces a list item
        run: echo "Hello"
```

Indent with spaces, **not tabs**. `run:` executes a shell command. `uses:` invokes a reusable action. `echo` prints a message. `ubuntu-latest` selects a GitHub-hosted Linux runner; it does not mean Ubuntu is installed on your laptop. GitHub may change which Ubuntu release `ubuntu-latest` selects. For reproducibility, use a specific supported runner label such as `ubuntu-24.04`.

## Lab 0 — Create your safe practice repository

1. Sign into github.com. Choose **New repository**. Name it `github-actions-learning-lab`.
2. Make it **private** if you prefer; check your plan's Actions quotas/billing. Initialize with a README and create it.
3. Open the repository's **Code** tab and click **Add file → Create new file**.
4. Enter `.github/workflows/hello.yml` as the file path. GitHub creates the folders automatically.
5. Paste the Lab 1 YAML and choose **Commit changes** to the default branch (`main` in these examples). If your default branch has a different name, adjust later examples.

**Checkpoint:** A repository stores files and history. A workflow is recognized from a YAML file under `.github/workflows/`.

## Lab 1 — Your first manual workflow

**Concept:** `workflow_dispatch` adds a manual Run workflow control. No programming language is required to print a message.

**File:** `.github/workflows/hello.yml`

```yaml
name: 01 - My First Workflow
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  greet:
    runs-on: ubuntu-24.04
    steps:
      - name: Print a welcome message
        run: echo "Hello! I ran my first GitHub Actions job."
```

**Run:** Commit file on default branch → **Actions** tab → select **01 - My First Workflow** → **Run workflow** → select branch → Run. Open the newest run → click **greet** → expand **Print a welcome message**. **Expected:** the sentence appears and the job is green. A short delay while GitHub assigns a runner is normal.

**Explain every field:** `name` is the display name; `on` is the trigger; `permissions` limits GitHub API access; `jobs` contains work; `greet` is your own job ID; `runs-on` picks the machine; `steps` lists commands; `run` executes a command.

**Try:** Change the greeting to your own words and rerun. **Checkpoint:** Does saving a new message automatically start this workflow? No—only manual dispatch is configured.

## Lab 2 — Steps, logs, and deliberate failure

**File:** `.github/workflows/steps.yml`

```yaml
name: 02 - Steps and Errors
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  practice:
    runs-on: ubuntu-24.04
    steps:
      - name: Step one
        run: echo "Starting"
      - name: Step two
        run: echo "Checking"
      - name: Step three
        run: echo "Finished"
```

Run and observe three ordered steps in **one job**. Then temporarily change step two to `run: exit 1`. Rerun: step two fails, and step three is skipped by default. Restore the original file. **Why:** a nonzero exit code means the shell command failed. **Checkpoint:** Jobs contain steps; steps within one job execute in order by default.

## Lab 3 — Parallel work versus dependent work

**File:** `.github/workflows/dependencies.yml`

```yaml
name: 03 - Job Dependencies
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  check:
    runs-on: ubuntu-24.04
    steps:
      - run: echo "Checking project"
  package:
    runs-on: ubuntu-24.04
    steps:
      - run: echo "Packaging project"
  publish:
    needs: [check, package]
    runs-on: ubuntu-24.04
    steps:
      - run: echo "Both prerequisites completed"
```

Run and inspect the job graph. `check` and `package` can run in parallel; `publish` waits for both to succeed. **Analogy:** you can prepare sauce and pasta simultaneously but cannot serve until both are ready. **Try:** remove `needs` and see how the graph changes. **Checkpoint:** Separate jobs normally use separate runners; do not assume files created in one job exist in another.

## Lab 4 — Automatically run on a Git change

**File:** `.github/workflows/push.yml`

```yaml
name: 04 - Push Trigger
on:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  inspect:
    runs-on: ubuntu-24.04
    steps:
      - name: Describe event
        run: echo "Triggered by ${{ github.event_name }} on ${{ github.ref_name }}"
```

Commit the workflow on `main`; the commit itself should trigger it. Modify `README.md` and commit again. Open Actions: a new run should appear without clicking Run workflow. **Try:** create a feature branch and commit there; the `main`-only trigger will not run. For a PR exercise, create a separate workflow with `on: pull_request:` and open a PR against main. Avoid merging merely for experimentation. **Checkpoint:** `push` and `pull_request` are distinct events. The source video's 30:55–36:22 section also covers branch and path filters.

## Lab 5 — Configuration and contexts

**File:** `.github/workflows/variables.yml`

```yaml
name: 05 - Variables
on:
  workflow_dispatch:
permissions:
  contents: read
env:
  APP_NAME: LearningLab
jobs:
  show:
    runs-on: ubuntu-24.04
    env:
      ENVIRONMENT: practice
    steps:
      - name: Print configuration
        env:
          MESSAGE: Welcome
        run: |
          echo "$MESSAGE to $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Repository: ${{ github.repository }}"
```

Run and observe the output. `env` at workflow scope is available across jobs, job scope within that job, and step scope within that step. `${{ github.repository }}` is a GitHub expression; `$APP_NAME` is a shell variable. **Try:** add a second job that prints `$APP_NAME` and `$ENVIRONMENT`; the first is set, the second is not. **Checkpoint:** Scope determines where a value is available.

## Lab 6 — Pass information between jobs

**File:** `.github/workflows/outputs.yml`

```yaml
name: 06 - Outputs
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  produce:
    runs-on: ubuntu-24.04
    outputs:
      greeting: ${{ steps.create.outputs.message }}
    steps:
      - id: create
        run: echo "message=Hello from job one" >> "$GITHUB_OUTPUT"
  consume:
    needs: produce
    runs-on: ubuntu-24.04
    steps:
      - env:
          RECEIVED: ${{ needs.produce.outputs.greeting }}
        run: echo "$RECEIVED"
```

Run and expand the `consume` step. **Expected:** `Hello from job one`. `GITHUB_OUTPUT` is a special runner-provided file; the producing step creates an output, the job exposes it, and `needs` lets the next job read it. **Checkpoint:** Ordinary job-local environment variables are not automatically shared between jobs.

## Lab 7 — Secrets, variables, and least privilege

**Concept:** Put non-sensitive configuration in **Variables**, sensitive values in **Secrets**, both accessible from repository Settings → Secrets and variables → Actions (subject to UI/permissions). **Never use real credentials in this exercise.**

Create repository variable `TRAINING_COLOR` with value `blue`. Use:

```yaml
name: 07 - Repository Variables
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  read-config:
    runs-on: ubuntu-24.04
    steps:
      - name: Show harmless variable
        env:
          COLOR: ${{ vars.TRAINING_COLOR }}
        run: echo "Favorite training color is $COLOR"
```

Run and check output. Next, create a **dummy** secret `TRAINING_DEMO_SECRET` with an invented value. You may reference it via `${{ secrets.TRAINING_DEMO_SECRET }}`, but **do not print it**. GitHub's log masking is helpful but not a guarantee against all secret leaks. `permissions: contents: read` follows least privilege. Never grant `contents: write`, `id-token: write`, or cloud permissions unless a specific job needs them. **Checkpoint:** `vars` is for ordinary values; `secrets` is for sensitive ones; neither is the same as a shell variable.

## Lab 8 — Artifacts versus caches

**Concept:** An artifact is a saved output you want to retrieve. A cache avoids repeating work; it is not a durable release artifact.

**File:** `.github/workflows/artifact.yml`

```yaml
name: 08 - Save an Artifact
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  report:
    runs-on: ubuntu-24.04
    steps:
      - name: Create report
        run: echo "Training run completed" > report.txt
      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: training-report
          path: report.txt
```

Run → open run summary → **Artifacts** → download `training-report` → open the text file. **Expected:** your saved sentence. `>` writes a file; `uses` calls a GitHub-provided action. For production, pin third-party actions (and preferably all actions) to a reviewed full commit SHA; the version tag here is used for readability in a learning lab. **Checkpoint:** A cache is disposable optimization; an artifact is a retained output subject to retention settings.

## Lab 9 — Matrix jobs and conditions

**File:** `.github/workflows/matrix.yml`

```yaml
name: 09 - Matrix
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  greet:
    strategy:
      matrix:
        environment: [dev, test, prod]
    runs-on: ubuntu-24.04
    steps:
      - name: Identify environment
        run: echo "This is ${{ matrix.environment }}"
      - name: Special message for test
        if: matrix.environment == 'test'
        run: echo "Only the test copy runs this step"
```

Run and count the jobs: three. Only the `test` job runs the second step. **Important:** these are just printed environment names; they do not deploy anywhere. **Checkpoint:** Matrix expands one job definition into multiple job runs, while `if` controls conditional execution.

## Lab 10 — Using a Marketplace action

**File:** `.github/workflows/checkout.yml`

```yaml
name: 10 - Checkout Source
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  inspect:
    runs-on: ubuntu-24.04
    steps:
      - name: Download repository files
        uses: actions/checkout@v4
      - name: Show repository files
        run: ls -la
```

Run and look for `README.md` in the listing. `checkout` downloads the repository into the job's workspace; `ls -la` lists files. **Checkpoint:** The runner doesn't automatically have your repository content; `checkout` normally provides it. Source video: 1:07:17–1:13:34. Production improvement: choose a current, reviewed action release and pin to its full SHA rather than copying old video versions.

## Lab 11 — Reusable workflow, not copy/paste

Create **`.github/workflows/reusable.yml`**:

```yaml
name: Reusable Greeting
on:
  workflow_call:
    inputs:
      person:
        required: true
        type: string
permissions:
  contents: read
jobs:
  greet:
    runs-on: ubuntu-24.04
    steps:
      - env:
          PERSON: ${{ inputs.person }}
        run: echo "Hello, $PERSON"
```

Create **`.github/workflows/caller.yml`**:

```yaml
name: 11 - Call Reusable Workflow
on:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  call-greeting:
    uses: ./.github/workflows/reusable.yml
    with:
      person: GitHub Learner
```

Run the **caller** from Actions and inspect the nested job. `workflow_call` makes the first file callable; `uses` at the **job** level calls a reusable workflow. A composite action, by contrast, is used as a **step** within a job. **Checkpoint:** Reusable workflows standardize whole jobs; composite actions package reusable steps.

## Lab 12 — Troubleshooting workshop

| Symptom | Likely reason | First check |
|---|---|---|
| No Run workflow button | No `workflow_dispatch`, file not on default branch, or permission/UI issue | `on:` block and branch |
| Workflow not visible | Wrong file location or invalid YAML | `.github/workflows/*.yml` |
| Red job | Shell command failed | Expand first failing step |
| `Unexpected value` | Invalid key/indentation | YAML structure and GitHub schema |
| File not found | No checkout or wrong working directory | `actions/checkout` and paths |
| Permission denied | Token lacks required permission | `permissions:` and repository policy |
| Secret empty | Secret name/scope unavailable | Secret settings, environment, fork restrictions |
| Job skipped | `needs` failed or `if` false | Job dependency graph |
| `act` differs from GitHub | Local container doesn't match hosted runner | Test on actual GitHub runner |

**Practice:** Change `run: echo "Hi"` to `run: command_that_does_not_exist`, run, locate error, then fix. Keep logs free of tokens. Optional local tools mentioned in the transcript: VS Code with YAML/GitHub Actions extensions, `actionlint`, and `act` with Docker. These are optional after browser-based labs; `act` is not a perfect replica of hosted runners.

## Lab 13 — Understand a real CI/CD pipeline without coding

The instructor's capstone (2:06:35–3:33:39) uses a React front end, Node and Go APIs, Python load generator, PostgreSQL, and Kubernetes. It demonstrates six workflow families: test, release, build/push images, stale issue/PR cleanup, GitOps manifest updates, and telemetry export. As a beginner, start with this architecture rather than recreating the multi-language application immediately:

```
Developer edits code -> opens PR
           |
           v
    Test workflow (CI) -- failure --> fix code
           |
        success
           v
      Review + merge
           |
           v
   Build container image -> publish to registry
           |
           v
  Update deployment manifest / deployment request
           |
           v
      Approval (production)
           |
           v
      Deploy + verify
```

**Paper exercise:** Label which arrows are events, which boxes are jobs/workflows, which outputs are artifacts, and where permissions or approval are required. **Key distinction:** a Docker image is a deployable artifact stored in a registry; the Kubernetes manifest says which image to run. The transcript's GitOps workflow updates a Git-tracked manifest; a cluster-side GitOps agent is responsible for applying it. Updating Git alone does not prove a cluster deployment happened.

## Lab 14 — Releases and observability

**Release automation:** Conventional commit messages such as `fix: correct login` and `feat: add search` can drive version bumps and release pull requests (video 3:20:27). **Stale automation:** Scheduled workflow marks untouched issues/PRs and may close them after a grace period (3:28:59). **Observability:** Export job/step timing via an OpenTelemetry-compatible action to an external observability service (3:31:06). These integrations can modify repository state or send data externally; study their permissions and destination before enabling.

**Exercise:** Draft a release policy on paper: PR tests must pass; only merged main changes build; production releases require a reviewer; deployment records include commit SHA, image digest, target environment, and outcome.

## Lab 15 — AWS architect's view (design only; no AWS charges)

```
GitHub PR -> tests -> build -> container registry (ECR/JFrog)
                                  |
                                  v
                       deployment promotion
                                  |
                         approval + controls
                                  |
                                  v
                        Amazon EKS / ECS
                                  |
                           health checks
```

**AWS security model:** GitHub-hosted runner obtains a short-lived GitHub OIDC identity token; AWS IAM validates the federated identity against a tightly scoped role trust policy; the workflow receives temporary AWS credentials via `aws-actions/configure-aws-credentials`. Restrict the trust policy to the intended repository, branch/environment and audience; grant only needed IAM actions. Do not copy a broad wildcard trust policy from a tutorial. The source video contrasts static keys and OIDC at 1:00:00–1:04:20. For an existing proof of concept that already uses GitHub secrets with static AWS keys, do not change authentication mid-lab solely to learn the basics; treat OIDC as a separate production-hardening exercise.

**Cost and deployment safety:** All hands-on labs above run on GitHub only. No EKS cluster, NAT gateway, ECR, or EC2 runner is required. When you later deploy, use an existing authorized test environment, confirm budget, scope IAM access, keep credentials out of logs, and require explicit approval before production changes. EKS and ECS are deployment targets, not GitHub Actions runners unless you intentionally host runners there.

## Quick reference: read a workflow from top to bottom

1. `name`: what is it called?
2. `on`: what event starts it?
3. `permissions`: what GitHub API rights does it get?
4. `env` / `vars` / `secrets`: what configuration does it use?
5. `jobs`: what pieces of work exist?
6. `runs-on`: where does each job execute?
7. `needs`: what must finish first?
8. `steps`: what exactly runs, and in what order?
9. `if` / `matrix`: which variants execute?
10. `outputs` / artifacts: what information leaves a step or job?

## Final self-assessment

Answer without looking: (1) Workflow versus action? (2) Event versus job? (3) `run` versus `uses`? (4) Why is `needs` important? (5) Why aren't files automatically shared across jobs? (6) Why use a secret instead of a variable? (7) Artifact versus cache? (8) How do you identify a failing step? (9) Why pin action dependencies? (10) How would you stop unreviewed code from deploying to production?

**Suggested answers:** (1) Workflow orchestrates jobs; action is reusable functionality. (2) Event triggers; job executes. (3) Shell command vs reusable action. (4) Controls dependencies. (5) Jobs have isolated runner environments. (6) Sensitive storage and access controls. (7) Retained output vs performance reuse. (8) Expand red job/step logs. (9) Reduce supply-chain drift. (10) Review, branch protection, scoped credentials, environment approvals.

## References and accuracy notes

- **Primary learning source:** attached transcript and video: https://www.youtube.com/watch?v=Xwpi0ITkL3U
- **GitHub workflow syntax:** https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- **GitHub expressions and contexts:** https://docs.github.com/en/actions/reference/workflows-and-actions/contexts
- **GitHub security hardening:** https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- **AWS OIDC integration:** https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
- **AWS configure credentials action:** https://github.com/aws-actions/configure-aws-credentials

**Source-versus-workbook distinction:** Course organization, concept progression, and capstone design come from the supplied transcript. All 12 runnable beginner labs and the restaurant analogy are original instructional adaptations. Exact current GitHub/AWS product behavior, billing, action major versions, and permissions should be checked against the linked live documentation before production use; these links are references, not a claim that all pages were freshly validated during authoring.
