# OpenHands PR Review GitHub Action

[![CI](https://github.com/All-Hands-AI/openhands-pr-review-action/actions/workflows/ci.yml/badge.svg)](https://github.com/All-Hands-AI/openhands-pr-review-action/actions/workflows/ci.yml)

This GitHub Action uses OpenHands AI to automatically review Pull Requests in your repository. It acts as a specialized wrapper around the more generic [`xinbenlv/openhands-action`](https://github.com/All-Hands-AI/openhands-action), providing a focused interface and prompt for PR reviews.

## How it Works

This action triggers the `xinbenlv/openhands-action` with a predefined prompt optimized for PR review tasks. It passes necessary context like the repository name and PR number, along with required secrets like the GitHub token and LLM API key, to the underlying action. The OpenHands agent then runs in a Docker container, analyzes the PR based on the prompt, and interacts with the GitHub API (using the provided token) to post review comments, line-level suggestions, and potentially approve the PR.

## Usage

To use this action, add the following step to your GitHub Actions workflow file (e.g., `.github/workflows/pr_review.yml`). Ensure the workflow triggers on `pull_request`.

```yaml
name: AI PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened] # Trigger on PR events

permissions:
  pull-requests: write  # Required to post comments/reviews
  contents: read      # Required to check out code

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
           fetch-depth: 0 # Fetch all history for better diff context if needed

      - name: Run OpenHands PR Review
        uses: All-Hands-AI/openhands-pr-review-action@v1 # Replace with the desired version
        with:
          # Required: A GitHub token with pull-request:write permissions.
          # Use a Personal Access Token (PAT) or a GitHub App token for cross-repo scenarios
          # or if the default GITHUB_TOKEN doesn't have sufficient permissions.
          review_bot_github_token: ${{ secrets.REVIEW_BOT_GITHUB_TOKEN }}

          # Required: Your LLM API Key (e.g., Anthropic, OpenAI)
          llm_api_key: ${{ secrets.LLM_API_KEY }}

          # Optional: Specify the LLM model (defaults to Claude 3 Haiku)
          # llm_model: 'anthropic/claude-3-opus-20240229'

          # Optional: Override the default review prompt
          # review_prompt: |
          #   Please review PR ${{ github.event.pull_request.number }} in ${{ github.repository }}.
          #   Focus on potential security issues and adherence to our style guide.
          #   Use the GITHUB_TOKEN to add comments.

          # Optional: Specify OpenHands or Runtime image versions
          # openhands_image: 'docker.all-hands.dev/all-hands-ai/openhands:latest'
          # runtime_image: 'docker.all-hands.dev/all-hands-ai/runtime:latest'

          # Optional: Pass additional CLI args to OpenHands
          # cli_args: '--max-iterations 15'
```

**Important:**

*   You **must** create repository secrets named `REVIEW_BOT_GITHUB_TOKEN` and `LLM_API_KEY`.
*   The `REVIEW_BOT_GITHUB_TOKEN` needs `pull-requests: write` permission to post reviews and comments. Using the default `${{ secrets.GITHUB_TOKEN }}` might not be sufficient, especially for comments/reviews made by the action itself. A PAT or a token from a dedicated GitHub App is recommended.
*   The runner environment must have Docker installed and running, and the action needs permission to access the Docker socket (`/var/run/docker.sock`). This is standard for GitHub-hosted runners.

## Inputs

| Input                     | Description                                                                                                                               | Required | Default                                                                                                                                                                                                                                                                                                                                                                                                                                |
| :------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `review_bot_github_token` | GitHub token with `pull-requests: write` permissions. Typically passed via `${{ secrets.REVIEW_BOT_GITHUB_TOKEN }}`.                         | `true`   | `N/A`                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `llm_api_key`             | API key for the LLM service (e.g., Anthropic, OpenAI). Typically passed via `${{ secrets.LLM_API_KEY }}`.                                    | `true`   | `N/A`                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `llm_model`               | The LLM model to use for the review.                                                                                                      | `false`  | `anthropic/claude-3-haiku-20240307`                                                                                                                                                                                                                                                                                                                                                                                                    |
| `openhands_image`         | Docker image for the OpenHands AI agent (passed to the underlying `xinbenlv/openhands-action`).                                               | `false`  | `docker.all-hands.dev/all-hands-ai/openhands:0.32`                                                                                                                                                                                                                                                                                                                                                                                     |
| `runtime_image`           | Docker image for the sandbox runtime environment (passed to the underlying `xinbenlv/openhands-action`).                                       | `false`  | `docker.all-hands.dev/all-hands-ai/runtime:0.32-nikolaik`                                                                                                                                                                                                                                                                                                                                                                              |
| `log_all_events`          | Enable detailed logging of all events during task execution by OpenHands.                                                                   | `false`  | `true`                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `review_prompt`           | The specific prompt instructing OpenHands how to review the PR. GitHub context (repo, PR number) is passed via `additional_env`.           | `false`  | (Default prompt instructs the AI to act as a coding expert, analyze changes, identify issues, post comments/approval using the provided `GITHUB_TOKEN`. It expects `GITHUB_REPOSITORY` and `GITHUB_PR_NUMBER` env vars set by the action.) See `action.yml` for the full default prompt. |
| `cli_args`                | Additional CLI arguments to pass directly to the OpenHands agent (e.g., `--max-iterations 10`).                                             | `false`  | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## Permissions

The workflow using this action requires the following permissions:

*   `pull-requests: write`: To allow the action (via the provided token) to post review comments and approve PRs.
*   `contents: read`: To allow the `actions/checkout` step to read repository content.

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
