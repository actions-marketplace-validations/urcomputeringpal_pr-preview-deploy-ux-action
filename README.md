# pr-preview-deploy-ux-action

An example of a GitHub Actions workflow that creates and updates transient Deployment objects using the GitHub API to provide a friendly user-experience for interacting with transient preview deployments.

## Features

- Each open PR contains a link to its active deployments.
- No additional notifications or emails are generated.
- PRs and the [Deployments tab](https://github.com/urcomputeringpal/pr-preview-deploy-ux-action/deployments) contains the history of all deployments.
- Sidebar and environment settings only shows active deployments.

## Requirements

- GitHub App with repo Administration permissions

## Example

### Active PR

#### Initial deployment

<img width="855" alt="image" src="https://user-images.githubusercontent.com/47/232165989-94d3b897-c03d-4057-a352-e667e11a1b93.png">

#### Subsequent synchronize events

<img width="866" alt="image" src="https://user-images.githubusercontent.com/47/232166116-9e861121-c3b6-437e-92fc-134e8b649bc4.png">

### Merged PR

[Example](https://github.com/urcomputeringpal/pr-preview-deploy-ux-action/pull/20)

## Usage

### Create Preview Deploy when PRs are created or updated

```yaml
name: PR Preview
on:
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        name: Start preview deploy
        id: start
        with:
          step: start
          token: ${{ secrets.GITHUB_TOKEN }}
          head_ref: ${{ github.head_ref }}
          env: preview-${{ github.event.pull_request.number }}
          env_url: https://pr-preview-${{ github.event.pull_request.number }}.example.com
          
      - run: echo perform your preview deploy logic here

      - uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        name: Finish preview deploy
        if: always()
        with:
          step: finish
          status: ${{ job.status }}
          token: ${{ secrets.GITHUB_TOKEN }}
          head_ref: ${{ github.head_ref }}
          env: preview-${{ github.event.pull_request.number }}

          deployment_id: ${{ steps.start.outputs.deployment_id }}
          env_url: https://pr-preview-${{ github.event.pull_request.number }}.example.com
 ```
 
 ### Delete Preview environments when PRs are merged or closed

- [Create a new app](https://docs.github.com/en/developers/apps/creating-a-github-app) with the following permissions:
  - Repository Deployments: Read and write
  - Repository Administration: Read and write
- Create secrets for the App ID and the App Private key. 

 ```yaml
name: PR Preview Cleanup
on:
  pull_request:
    types: [ closed ]
  
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - run: echo perform your cleanup logic here

      - name: Cleanup Preview deploy
        uses: urcomputeringpal/pr-preview-deploy-ux-action@v0
        with:
          step: cleanup
          env: preview-${{ github.event.pull_request.number }}
          app_id: ${{ secrets.APP_ID }}
          private_key: ${{ secrets.APP_PEM }}            
```

See [.github/workflows](./.github/workflows) for more examples.
