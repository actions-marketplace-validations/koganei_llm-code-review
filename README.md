# llm-code-review-action
A container GitHub Action to review a pull request by OpenAI's LLM Model.

If the size of a pull request is over the maximum chunk size of the OpenAI API, the Action will split the pull request into multiple chunks and generate review comments for each chunk.
And then the Action summarizes the review comments and posts a review comment to the pull request.

## Pre-requisites
We have to set a GitHub Actions secret `OPENAI_API_KEY` to use the OpenAI API so that we securely pass it to the Action.

## Inputs

- `apiKey`: The OpenAI API key to access the API.
- `githubToken`: The GitHub token to access the GitHub API.
- `githubRepository`: The GitHub repository to post a review comment.
- `githubPullRequestNumber`: The GitHub pull request number to post a review comment.
- `gitCommitHash`: The git commit hash to post a review comment.
- `pullRequestDiff`: The diff of the pull request to generate a review comment.
- `pullRequestDiffChunkSize`: The chunk size of the diff of the pull request to generate a review comment.
- `temperature`: The temperature to generate a review comment.
- `topP`: The top_p to generate a review comment.
- `topK`: The top_k to generate a review comment.
- `maxNewTokens`: The max_tokens to generate a review comment.
- `logLevel`: The log level to print logs.

As you might know, a model of OpenAI has limitation of the maximum number of input tokens.
So we have to split the diff of a pull request into multiple chunks, if the size of the diff is over the limitation.
We can tune the chunk size based on the model we use.

## Example usage
Here is an example to use the Action to review a pull request of the repository.
The actual file is located at [`.github/workflows/test-action.yml`](.github/workflows/test-action.yml).


```yaml
name: "Test Code Review"

on:
  pull_request:
    paths-ignore:
      - "*.md"
      - "LICENSE"

jobs:
  build:
    permissions:
      contents: read
      pull-requests: write
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # needed to checkout all branches for the diff action to work

      # Check the PR diff using the current branch and the base branch of the PR
      - uses: GrantBirki/git-diff-action@v2.4.1
        id: git-diff-action
        with:
          json_diff_file_output: diff.json
          raw_diff_file_output: diff.txt
          file_output_only: "false"

      - uses: koganei/llm-code-review@main
        name: "Code Review"
        id: review
        env:
          DIFF: ${{ steps.git-diff-action.outputs.raw-diff-path }}
        with:
          apiKey: ${{ secrets.OPENAI_API_KEY }}
          githubToken: ${{ secrets.GITHUB_TOKEN }}
          githubRepository: ${{ github.repository }}
          githubPullRequestNumber: ${{ github.event.pull_request.number }}
          gitCommitHash: ${{ github.event.pull_request.head.sha }}
          temperature: "0.2"
          maxNewTokens: "250"
          topK: "50"
          topP: "0.95"
          pullRequestDiff: |-
            $(cat $DIFF)
          pullRequestChunkSize: "3500"
          logLevel: "DEBUG"
```
