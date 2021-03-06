# CloudFormation-Elastic Beanstalk-CodePipeline Example

Gotchas encountered in the creation of this process are covered in 
[this blog post](https://blairnangle.com/2020-05-10/gotchas-elastic-beanstalk-cross-account-cloudformation-codepipeline).

Example project to create the infrastructure and cross-account continuous deployment pipeline for a simple Spring Boot 
application on AWS's Elastic Beanstalk.

## Prerequisites

* [AWS Command Line Interface](https://aws.amazon.com/cli/) (tested against 2.0.11)
* Two AWS accounts:
    * A "tools" account for hosting CD pipeline and associated resources
    * A test account where the app will be deployed
* [AWS account-based credentials configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).
    `~/.aws/credentials` (the IAM user associated with the access keys must have the necessary
    permissions to create all the resources in the templates) should look similar to:
    ```bash
    [tools]
    aws_access_key_id=AKIAIOSFODNN7EXAMPLE
    aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
  
    [test]
    aws_access_key_id=AKIAIOSFODNN7EXAMPLE
    aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    ```
* [GitHub personal access code](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) 

## Configuration

Add your AWS region, tools and test account numbers and profile names to `infra/.config`. Consistent with the [profiles 
above](#prerequisites), for example:

```bash
REGION=eu-west-2
TOOLS_PROFILE=tools
TOOLS_ACCOUNT=301251875798
TEST_PROFILE=test
TEST_ACCOUNT=752510075258
```

Also add your GitHub username and project name. For example:

```bash
GITHUB_USERNAME=blairnangle
GITHUB_REPO=example-elastic-beanstalk-cross-account-codepipeline-cloudformation
```

Add your GitHub personal access token to AWS Secrets Manager in your tools account. I have called named mine 
`gitHubPersonalAccessToken` with a "SecretKey" of `token`. It is referenced in `infra/tools/pipeline.yml`.

## Usage

Run the `go` script from the project root:

```bash
$ ./go
```

It should take ~14 minutes to complete.

### Creating and Associating a Custom Security Group

See the "Security Groups" section of the [blog post](https://blairnangle.com/2020-05-10/gotchas-elastic-beanstalk-cross-account-cloudformation-codepipeline)
associated with this project to understand what is going on here.
Bear in mind that the security group created will leave the app open to all traffic on the internet and is just
intended to provide an automated way of validating the project end-to-end.

Run `create-and-associate-security-group` from the project root to create a new security group for the load balancer
associated with the Elastic Beanstalk environment and add it as an inbound source for the existing instance security
group:

```bash
$ ./create-and-associate-security-group
```

It should take ~1 minute to complete.

### Cleaning up

Run the `destroy` script from the project root to delete all the AWS resources:

```bash
$ ./destroy
```

It should take ~7 minutes to complete.

## Naming Convention

User-defined variables are in `camelCase` to distinguish them from AWS keys which are in `PascalCase`.

CloudFormation restricts parameters and resource names to be alphanumeric (no hyphens or underscores), meaning that, if 
parameter values are to be used in the construction of AWS resource names, resource names all need to be alphanumeric 
if the naming convention is to be consistent.

## Notes

The `dummyPipelinePreReqs` (tools account) and `dummyIam` (test account) stacks are not specific to the deployment of 
this project. They incorporate resources that can be reused for other cross-account CodePipeline pipelines.

## License

Distributed under [MIT License](./LICENSE).
