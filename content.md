# Continuous Integration / Continuous Deployment (CI/CD) With GitHub Actions

## Introduction

When you’re building software with a team, it’s important to make sure that new code doesn’t accidentally break existing features. That’s where **CI/CD** comes in.

- **Continuous Integration (CI)** means: every time you push code, an automated system checks it (for example, by running your tests).

- **Continuous Deployment (CD)** means: once the code has passed those checks, it can automatically be delivered to users.

You can think of CI/CD as having a robot teammate who runs your commands for you in the cloud, every time code changes. Instead of waiting to find out something is broken later, the robot lets you know right away with a ✅ or ❌.

### What is a pipeline?

The series of steps this robot runs is called a pipeline. For example this "robot" could be told to:

1. Check out your code

2. Install Ruby and your gems

3. Run your tests

It does all this so you and your team don't have to!

In this lesson, you’ll create your first pipeline using GitHub Actions. The pipeline will run your Rails test suite whenever changes are pushed to the repository, giving you quick feedback about whether your code is working as expected.

## Setting Up Your GitHub Repository

GitHub Actions can run on any kind of project — Ruby, Python, JavaScript, or otherwise. The only requirement is that your code lives in a repository on GitHub.

For this lesson, we’ll focus on a Rails application because the example workflow is designed to run a Rails test suite. That doesn’t mean GitHub Actions is limited to Rails though, it’s just the context we’ll use here.

It’s important to note: this lesson is not about creating a brand-new empty repository. The workflow we’ll build expects real code and tests to run. If you already have a Rails project pushed to GitHub that has rspec tests, you’re ready. If not, complete at least one Rails assignment and push it to GitHub before continuing. Otherwise, the example workflow will likely fail.

## Creating the GitHub Actions Workflow

Now that you've selected an existing Rails project on GitHub, you can add a workflow to it. A workflow in GitHub Actions is just a set of automated steps (a “pipeline”) that run whenever certain events happen — in our case, whenever code is pushed or a pull request is opened.

In this lesson, we’ll create a workflow that runs your Rails test suite automatically, so you get instant feedback on whether your code is working as expected.

1. Create the Workflow File
In your repository, navigate to the `Actions` tab. Click `New workflow`.

![](assets/actions.png)

GitHub will suggest some starter workflows (including one for Ruby on Rails). For this lesson, don’t worry about those suggestions. Instead, choose “set up a workflow yourself” (blue hyperlink in the screenshot above). This opens an editor where you can paste in your own configuration.

Name the file something like `rails_ci.yml`.

![](assets/new-workflow.png)

2. Define Workflow Configuration
Insert the following YAML configuration into the `rails_ci.yml` file:

```yaml
# This workflow will install a prebuilt Ruby version, install dependencies, and run tests
name: "Ruby on Rails CI"
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:11-alpine
        ports:
          - "5432:5432"
        env:
          POSTGRES_DB: rails_test
          POSTGRES_USER: rails
          POSTGRES_PASSWORD: password
    env:
      RAILS_ENV: test
      DATABASE_URL: "postgres://rails:password@localhost:5432/rails_test"
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Install Ruby and gems
        uses: ruby/setup-ruby@55283cc23133118229fd3f97f9336ee23a179fcf # v1.146.0
        with:
          bundler-cache: true
      - name: Set up database schema
        run: bin/rails db:schema:load
      # This runs the tests in the spec/ folder
      - name: Run rspec tests
        run: bundle exec rspec
```

3. Commit and Push the `rails_ci.yml` Workflow File

4. Verify the Workflow
After pushing, go back to the Actions tab in your GitHub repository.
You should see the workflow running. If everything is set up correctly, the workflow will execute and show a success status if your tests pass. Now anytime you create a pull request into the `main` branch it will trigger this workflow.

![](assets/pull-request.png)

## Demystifying the YAML

If your workflow failed, the first place to start is by understanding exactly what we're asking the "robot" to check for. We have to understand the instructions in order to understand where it might be going wrong.

Let’s break down the workflow file piece by piece. Remember: a workflow is just a recipe of steps the GitHub Actions robot follows whenever it’s triggered.

**Chunk 1:**

```yaml
# This workflow will install a prebuilt Ruby version, install dependencies, and run tests

name: "Ruby on Rails CI"
```

- `name:` is simply a label for the workflow. It shows up in the Actions tab. You could call it “Rails Checks” or “Bob's Handy CI Pipeline” and it would work the same.

**Chunk 2:**

```yaml
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```

- `on:` tells GitHub *when* to run this workflow.

- In this case: whenever code is pushed to main, or when a pull request targets main.

**Chunk 3:**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

- `jobs:` defines the main tasks.
- Here we have one job, called `test` (you could name it anything).
- `runs-on:` specifies the type of machine GitHub should provide. `ubuntu-latest` means “give me a fresh Ubuntu Linux **virtual machine** (essentially a temporary computer that runs in GitHub’s cloud).”

**Chunk 4:**

```yaml
    services:
      postgres:
        image: postgres:11-alpine
        ports:
          - "5432:5432"
        env:
          POSTGRES_DB: rails_test
          POSTGRES_USER: rails
          POSTGRES_PASSWORD: password
```

- `services:` adds extra **containers** your job can use. (A container is like a small prebuilt sidekick computer-in-a-box that runs alongside your main virtual machine)
- Here we’re spinning up a PostgreSQL database in a container, which Rails will connect to during tests.
- The `env` variables define the database name, user, and password.

**Chunk 5:**

```yaml
    env:
      RAILS_ENV: test
      DATABASE_URL: "postgres://rails:password@localhost:5432/rails_test"
```

- `env:` sets environment variables for the whole job.
- These tell Rails it’s running in the test environment, and how to connect to the database.

**Chunk 6:**

```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
```

- `steps:` lists the individual instructions inside the job.
- Anytime you see `uses:`, you’re telling GitHub Actions to pull in and run a prebuilt action.
- Those actions can be official ones GitHub provides (like `actions/checkout`), or community-built ones published on the GitHub Marketplace.
- `actions/checkout@v3` is a prebuilt GitHub Action (made by GitHub) that downloads your repository code into the VM. Without it, the VM would start out empty.
- You can think of them as plugins that save you from writing everything yourself.

**Chunk 7:**

```yaml
      - name: Install Ruby and gems
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
```

- This uses another prebuilt Action,`ruby/setup-ruby@v1`, to install Ruby and your gems.
- By default it reads your `.ruby-version` file to know which Ruby version to install, so make sure this file exists in your repo!
- `bundler-cache: true` speeds things up by caching installed gems between runs.

**Chunk 8:**

```yaml
      - name: Set up database schema
        run: bin/rails db:schema:load
```

- `run:` means “type this command into the virtual machine’s terminal.” (Just like if you maually ran this command in your VS code terminal in your project repo.)
- This Rails terminal command prepares the test database by loading the schema.

**Final Chunk!**

```yaml
      - name: Run rspec tests
        run: bundle exec rspec
```

- Finally, this runs your RSpec tests! Does the command look familar?
- If they pass, you’ll see a ✅. If they fail, you’ll see a ❌.

## Additional Considerations

- **Caching Dependencies**: To speed up the workflow, you can cache your dependencies using the actions/cache step. This reduces the time spent in the installation step on subsequent runs.
- **Notifications**: You can set up notifications (e.g., email, Slack) to alert you when the CI process fails. This helps keep the team informed about the health of the application.
- **Deployment**: Once your CI setup is stable, consider adding deployment steps to automate the deployment of your application to your hosting service.

## Quiz

- What is the primary purpose of Continuous Integration (CI)?
- To deploy the application to production.
  - Not quite. Deployment is part of Continuous Deployment (CD), not CI.
- To automate the process of integrating code changes and running tests to ensure the changes do not break the application.
  - Correct! CI helps catch issues early by automatically testing code changes.
- To manage code versions and branches.
  - Not quite. Version control is handled by tools like Git, not CI.
{: .choose_best #ci_purpose title="Purpose of Continuous Integration" points="1" answer="2" }

- Which event triggers the GitHub Actions workflow defined in the example?
- Issue creation and comments.
  - Not quite. The workflow is triggered by pushes and pull requests, not issue activities.
- Deployment to production.
  - Not quite. The workflow is triggered by code changes, not deployments.
- Push and pull request events to the main branch.
  - Correct! The workflow is triggered by pushes and pull requests to the main branch.
{: .choose_best #workflow_trigger title="Triggering GitHub Actions Workflow" points="1" answer="3" }

- What is the purpose of the runs-on key in a GitHub Actions workflow?
- To specify the type of machine to run the job on.
  - Correct! runs-on defines the environment where the job will execute, such as ubuntu-latest.
- To define the commands to run in the job.
  - Not quite. Commands are defined in the steps section, not runs-on.
- To list the dependencies required for the job.
  - Not quite. Dependencies are handled in the steps section.
{: .choose_best #runs_on_purpose title="Purpose of runs-on Key" points="1" answer="1" }

- What does the services section in the GitHub Actions workflow file do?
- Defines the Ruby version to use.
  - Not quite. Ruby setup is handled in the steps section.
- Sets up a PostgreSQL database for the test environment.
  - Correct! The services section configures the PostgreSQL database required for the Rails test environment.
- Configures notifications for workflow failures.
  - Not quite. Notifications are configured separately, not in the services section.
{: .choose_best #services_section title="Purpose of services Section" points="1" answer="2" }

- Which steps are necessary to run tests in the GitHub Actions workflow? (Select all that apply)
- Checkout code.
  - Correct! Checking out the code is necessary to access the repository's files.
- Install Ruby and gems.
  - Correct! Installing Ruby and the required gems is essential to run the Rails application.
- Set up the database schema.
  - Correct! Setting up the database schema prepares the test database.
- Deploy to production.
  - Not quite. Deployment is not required to run tests.
{: .choose_all #run_tests_steps title="Steps to Run Tests in GitHub Actions" points="3" answer="[1, 2, 3]" }

- What are some benefits of using GitHub Actions for CI/CD? (Select all that apply)
- Automates the testing process.
  - Correct! GitHub Actions automates running tests, ensuring code quality.
- Integrates seamlessly with GitHub repositories.
  - Correct! GitHub Actions is built into GitHub, providing tight integration with repositories.
- Requires no additional configuration for deployments.
  - Not quite. Deployments may require additional configuration depending on the environment and tools used.
- Provides flexibility to customize workflows.
  - Correct! GitHub Actions allows customization to fit various development and operational needs.
{: .choose_all #github_actions_benefits title="Benefits of GitHub Actions" points="3" answer="[1, 2, 4]" }

## Playlist

[Playlist](https://github.com/DPI-WE/pr-playlist) is an index of favorite songs. Follow the contribution guidelines to add your song and practice your pull request skills with github actions.

- Did you add your song in a pull request?
- Yes
  - Great job!
- No
  - Please contribute to the repository before continuing.
{: .choose_best #playlist_pull_request_url title="Playlist Pull Request URL" points="1" answer="1" }

<div class="alert alert-danger mt-2">

Return to Canvas and submit the URL in the assignment "Pull Request URL for Playlist".

An instructor will provide additional feedback on your submission there.
</div>

## Add a GitHub Action CI/CD Flow to an assignment

Submit the repository URL where you've added a GitHub Action CI/CD Flow and practice your DevOps skills. Add a `test` step (not a lint step). Add it to an assignment with a test suite. (eg your assignments with a rake grade step)

- Did you add a GitHub Action CI/CD Flow to an assignment?
- Yes
  - Great job!
- No
  - Please do that before continuing.
{: .choose_best #github_action_url title="GitHub Action CI/CD Flow URL" points="1" answer="1" }

<div class="alert alert-danger mt-2">

Return to Canvas and submit the URL in the assignment "Add a GitHub Action CI/CD Flow to an assignment".

An instructor will provide additional feedback on your submission there.
</div>

## Conclusion

Setting up CI/CD with GitHub Actions empowers your team to catch issues early and deploy confidently. This lesson provided a basic setup, but GitHub Actions is very flexible, allowing you to tailor your workflows to match your development and operational needs perfectly. Always explore further optimizations and tools to integrate into your pipeline for more robust automation.

---

- Approximately how long (in minutes) did this lesson take you to complete?
{: .free_text_number #time_taken title="Time taken" points="1" answer="any"}
