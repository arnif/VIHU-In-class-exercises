# Setting Up a Basic GitHub Actions Workflow: Hello World Example

Setting up a basic GitHub Actions workflow to run a simple command, like printing "Hello World," is a great way to start experimenting with CI/CD processes. Here's a step-by-step guide:

### 1. Create a GitHub Repository

If you don't already have a repository, create one on GitHub. This is where your workflow will be configured.

### 2. Add a Workflow File

GitHub Actions are configured through YAML files within the `.github/workflows` directory of your repository.

1. In your GitHub repository, create a new directory structure: `.github/workflows/`.
2. Inside the `workflows` directory, create a new file, for example, `hello_world.yml`.

### 3. Define the Workflow

Edit `hello_world.yml` to define the workflow. Here's a basic template to get you started:

```yaml
name: Hello World Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Run a one-line script
        run: echo Hello, world!
```

This workflow does the following:

- **Triggered on Push**: It runs every time you push code to any branch of your repository.
- **Job**: It has a single job called `build`.
- **Runner**: The job runs on the latest Ubuntu runner provided by GitHub.
- **Steps**:
  - **Checkout Repository**: Checks out your repository so the workflow can access it.
  - **Run Script**: Executes the `echo Hello, world!` command.

### 4. Commit and Push

Commit the `hello_world.yml` file to your repository and push it to GitHub.

```bash
git add .github/workflows/hello_world.yml
git commit -m "Add hello world GitHub action"
git push
```

### 5. Verify the Workflow

After pushing the workflow file to GitHub:

1. Go to your GitHub repository.
2. Click on the "Actions" tab.
3. You should see your "Hello World Workflow" listed there.
4. Click on it to see the details and logs, ensuring that it prints "Hello, world!"

### 6. Experiment and Learn

Since you're a tinkerer and like to experiment:

- Try modifying the command in the script.
- Experiment with different triggers (like on pull requests, specific branches, etc.).
- Explore more complex workflows with multiple jobs, dependencies, and conditional execution.

### Tips

- Make sure you have the correct indentation in your YAML file, as it is crucial for it to be parsed correctly.
- Check out the [GitHub Actions documentation](https://docs.github.com/en/actions) for more advanced configurations and options.

This should give you a basic setup to start with. From here, you can expand and customize your workflow to suit your specific needs.
