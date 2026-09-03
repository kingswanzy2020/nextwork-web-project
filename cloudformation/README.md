# CloudFormation

The infrastructure behind the pipeline, captured as templates so the whole stack is
reproducible instead of clicked together.

| Template | Contents |
|---|---|
| `pipeline-stack.yaml` | The delivery stack — CodeArtifact domain and repositories (including the Maven Central upstream store), the S3 artifact bucket, the CodeStar Connections link to GitHub, CodeDeploy application, and the IAM roles, instance profile, and managed policies each service needs |
| `deployment-environment.yaml` | The deployment environment the app is released onto |

## Before deploying

**AWS account IDs are redacted to `111122223333`.** They appear inside generated IAM
managed-policy resource names such as
`...CodeBuildBasePolicynextworkdevopscicdapnortheast1111122223333`. Substitute your own
account ID before deploying, or the policy names will not match what the console-generated
roles expect.

`pipeline-stack.yaml` takes `GitHubRepoOwner` and `GitHubRepo` as parameters:

```bash
aws cloudformation deploy \
  --template-file cloudformation/pipeline-stack.yaml \
  --stack-name nextwork-devops-cicd \
  --parameter-overrides GitHubRepoOwner=<your-github-user> GitHubRepo=nextwork-web-project \
  --capabilities CAPABILITY_NAMED_IAM
```

`CAPABILITY_NAMED_IAM` is required because the stack creates named IAM roles and managed
policies.

## Note on the CodeStar connection

`AWS::CodeStarConnections::Connection` is created in `PENDING` status. A connection to
GitHub cannot be completed by CloudFormation — someone has to authorize the GitHub App in
the console once. Until that happens the pipeline's source stage will fail, which looks
like a broken template but is the expected handshake.
