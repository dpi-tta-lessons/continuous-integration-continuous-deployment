# Continuous Integration / Continuous Deployment (CI/CD) With GitHub Actions

## Introduction
Continuous Integration (CI) and Continuous Deployment (CD) are crucial practices in modern software development, particularly for agile teams. In this lesson, we will guide you through setting up a simple CI/CD pipeline using GitHub Actions for a Ruby on Rails application. This pipeline will primarily focus on running the test suite whenever changes are pushed to the repository.

## Setting Up Your GitHub Repository
Before we begin, ensure your Rails application is hosted on GitHub. If not, you can create a new repository and push your Rails application code to it.

## Creating the GitHub Actions Workflow
GitHub Actions allows you to automate, customize, and execute your software development workflows right in your repository. Here, we'll create a workflow to run the Rails test suite.

1. Create the Workflow File
In your repository, navigate to the `Actions` tab. Click `New workflow`.

![](assets/actions.png)


You can choose a suggested workflow for Ruby on Rails or skip the suggested workflows and click set up a workflow yourself. You'll be presented with an editor to write your workflow file. Name the file something like `rails_ci.yml`.

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

## Explanation of Key Sections
- **on**: Specifies the events that trigger the workflow. Here, it triggers on pushes and pull requests to the main branch.
- **jobs**: Defines the jobs that will be run. Here, we have one job called build.
- **runs-on**: Indicates the type of machine to run the job on. We use the latest Ubuntu.
- **services**: Sets up a PostgreSQL database, which is required for the Rails test environment.
- **steps**: Lists the steps of the job. This includes checking out the code, setting up Ruby, installing dependencies, setting up the database, and running the tests.

3. Commit and Push the `rails_ci.yml` Workflow File

4. Verify the Workflow
After pushing, go back to the Actions tab in your GitHub repository.
You should see the workflow running. If everything is set up correctly, the workflow will execute and show a success status if your tests pass. Now anytime you create a pull request into the `main` branch it will trigger this workflow.

![](assets/pull-request.png)

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
