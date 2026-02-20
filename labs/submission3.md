## Lab 3: GitHub Actions and Workflow Automation

### Objective
The objective of this lab was to gain hands-on experience with GitHub Actions by creating and analyzing both automatically and manually triggered CI/CD workflows. The lab focused on understanding the core concepts of GitHub Actions and comparing different execution methods.

---

### Task 1: Implementing a Basic CI Workflow

For the first task, I implemented a basic continuous integration workflow using GitHub Actions. The configuration file was created at `.github/workflows/github-actions-demo.yml`.

#### Key Concepts Learned
This exercise provided a practical understanding of the fundamental building blocks of GitHub Actions:

-   **Jobs**: A workflow is composed of one or more jobs. A job is a set of steps that execute on the same runner. In my configuration, I defined a single job named `build` which contains all the steps for checking out code and running a simple script.
-   **Steps**: Steps are individual tasks that run commands or actions in a job. My workflow includes steps like `actions/checkout@v4` (a reusable action to fetch the repository code) and a `- run:` step to execute a shell command (`echo "Hello, world from Task 1!"`).
-   **Runners**: A runner is a server that executes the workflows. GitHub provides hosted runners (like `ubuntu-latest` which I used) that come with pre-installed tools and software, offering a clean and consistent environment for each job execution.
-   **Triggers**: Also known as events, these are specific activities that cause a workflow to run. My workflow was configured with an `on: push` trigger, meaning it executes automatically whenever code is pushed to the repository.

#### Trigger Analysis
The workflow run was triggered automatically by a `push` event. Specifically, when I pushed the commit containing the `.yml` workflow file and this report to the `main` branch, GitHub detected the change and initiated a new workflow run based on the rules defined in the configuration file.

#### Workflow Execution Process
Upon the `push` event, the following execution process was observed:
1.  **Event Detection**: GitHub detected the push to the `main` branch.
2.  **Runner Provisioning**: GitHub Actions provisioned a virtual machine running `ubuntu-latest` to act as the runner for the job.
3.  **Job Execution**:
    -   The runner downloaded and executed the `actions/checkout@v4` step, which pulled my repository code onto the runner.
    -   Next, it executed the `run` step, which ran the `echo` command in the runner's shell.
4.  **Completion & Logging**: The job completed successfully, and all console output (including the "Hello, world!" message) was captured and made available in the workflow run's logs on GitHub.

![Working workflow](screenshots/workflow.png)

---

### Task 2: Implementing and Analyzing a Manually Triggered Workflow

For the second task, I modified the workflow to include a manual trigger (`workflow_dispatch`), allowing for on-demand execution with configurable inputs.

#### Implementation: Workflow Dispatch with Inputs
I added a `workflow_dispatch` block to the `on:` section of the YAML file, incorporating configurable parameters. This configuration was adapted from the GitHub documentation.

```yaml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
          - info
          - warning
          - debug
      print_tags:
        description: 'True to print to STDOUT'
        required: true
        type: boolean
```

#### Analysis of Runner Environment and Capabilities
When the workflow was triggered manually, the runner logs provided deep insight into the execution environment. A segment of the debug log output is shown below:
```
##[debug]Evaluating condition for step: 'Post Check out repository code'
##[debug]Evaluating: always()
##[debug]Evaluating always:
##[debug]=> true
##[debug]Result: true
##[debug]Starting: Post Check out repository code
##[debug]Loading inputs
##[debug]Evaluating: github.repository
##[debug]Evaluating Index:
##[debug]..Evaluating github:
##[debug]..=> Object
##[debug]..Evaluating String:
##[debug]..=> 'repository'
##[debug]=> 'DanisSharafiev/DevOps-Intro'
##[debug]Result: 'DanisSharafiev/DevOps-Intro'
##[debug]Evaluating: github.token
##[debug]Evaluating Index:
##[debug]..Evaluating github:
##[debug]..=> Object
##[debug]..Evaluating String:
##[debug]..=> 'token'
##[debug]=> '***'
##[debug]Result: '***'
##[debug]Loading env
```

This output demonstrates key capabilities of the runner environment:

- Contextual Information: The runner has access to rich context objects like github, which contains information about the repository (github.repository), the event that triggered the run, and more.

- Security: Sensitive information, such as the github.token (used for authentication), is automatically masked and redacted (***) in the logs to prevent accidental exposure.

- Expression Evaluation: The runner dynamically evaluates expressions (like always()) to control the flow of steps.

- Step Isolation: The "Post" step for checking out code shows that the runner manages the lifecycle of actions, including cleanup.

#### Comparison: Manual vs. Automatic Workflow Triggers

-   **Automatic (`push`)** — triggered by code changes, runs the same way every time.
-   **Manual (`workflow_dispatch`)** — triggered by a person, allows choosing options before running.