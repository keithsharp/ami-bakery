# Creating an AMI Bakery
I want to create my EC2 instances as [Immutable Infrastructure](https://www.oreilly.com/ideas/an-introduction-to-immutable-infrastructure) and to speed up boot time by doing as little configuration at boot time as possible.  The way to do this is to pre-build, or bake, your AMIs with your application already installed.

The demo in this repository shows how to use [Packer](https://packer.io) in conjunction with [AWS CodeCommit](https://aws.amazon.com/codecommit/), [AWS CodePipeline](https://aws.amazon.com/codepipeline/), and [AWS CodeBuild](https://aws.amazon.com/codebuild/), to automatically create a new Amazon Linux AMI with all of the current updates applied based on the contents of a Git repository.

## Steps to Build the Demo
1. Create the VPC using the `vpc.yaml` CloudFormation template.  Note the VPC and Subnet IDs and update `packer-files/packer.json` with the values.  While you are editing packer.json make sure that the AMI ID for the `source_ami` is set for your environment (it's the Amazon Linux AMI in EU-West-1).
2. Create the Git repository using `codecommit.yaml`, checkout the empty repository, copy the files in `packer-files` into the checkout repository, then commit and push.
3. Create the ECR repository using `ecr-repository.yaml`.
4. Use the following commands to build the Docker container and push it to the repository, substitute the variables in block capitals as appropriate:

    `cd container`

    `docker build --rm -t <USERNAME>/packer:latest .`

    `aws ecr get-login --region <AWS REGION>`

    Run the `docker login` command that's output by `aws ecr ...`.

    `docker tag <USERNAME>/packer:latest <AWS ACCOUNT>.dkr.ecr.<AWS REGION>.amazonaws.com/packer:latest`

    `docker push <AWS ACCOUNT>.dkr.ecr.<AWS REGION>.amazonaws.com/packer:latest`

5. Create the roles for CodeBuild and CodePipeline using the files `codebuild-role.yaml` and `codepipeline-role.yaml`.
6. Edit `codebuild-project.yaml` to reference the container you just created on line 15 - I should really make this automagic using `!Sub`.
7. Create the CodeBuild project using the `codebuild-project.yaml` file.
8. Create the CodePipeline using the `codepipeline.yaml` file.

Once step 8 completes the CodePipeline should automatically detected the Git repository and start executing the pipeline.  After 10 minutes of so you should be able to go to the AMI section of the EC2 page on the AWS console and see your new image.
