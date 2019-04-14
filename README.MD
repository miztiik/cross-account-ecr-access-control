# Grant access to another AWS Account B to pull or push images to Account A ECR Repo
To allow AWS Account B to be able to connect to Account A ECR image repository to push or pull images, you must create a policy that allows the secondary account to perform those API calls against the repository.


![Fig : Tag-Based ECR Access Control](https://raw.githubusercontent.com/miztiik/cross-account-ecr-access-control/master/images/miztiik-cross-account-ecr-access-control.png)

#### Follow this article in [Youtube](https://youtu.be/UYaV-VfnbT8)

0. ## Prerequisites
    - Ensure you have tagged the repositories in `Account A`
        - Minimum of one repository with atleast the following tag,
        - TagKey: `Team`
        - TagValue: `Payments`
    - `Account B` user(ex: `devusr`) with AWS CLI access
    - _Note down `Account A & B` IDs_

1. ## Set Repo Permissions Policy in `Account A`
    The following policy allows user `devusr2` to push and pull images to a repo which has the tagkey `Team` and its value as `Payments`. 
    As this policy is assigned to only to the particular repo, the user will _not_ get access to any other repo
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "ecr:GetAuthorizationToken",
              "ecr:BatchCheckLayerAvailability",
              "ecr:GetDownloadUrlForLayer",
              "ecr:GetRepositoryPolicy",
              "ecr:DescribeRepositories",
              "ecr:ListImages",
              "ecr:DescribeImages",
              "ecr:BatchGetImage",
              "ecr:InitiateLayerUpload",
              "ecr:UploadLayerPart",
              "ecr:CompleteLayerUpload",
              "ecr:PutImage"
            ],
            "Condition": {
              "StringLike": {
                "aws:ResourceTag/Team": "Payments"
              }
            },
            "Principal": {
              "AWS": [
                "arn:aws:iam::YOUR-ACCOUNT-B-ID:user/devusr2"
              ]
            },
            "Sid": "AllowCrossAccountPushAndPull"
          }
        ]
      }
      ```


1. ## Verifying Access from `devusr2@Account B`
    From the terminal with AWS CLI configured with `devusr2` credentials as profile `dev-acc` run the following,
    1. Get ECR Credentials
        Change the `YOUR-ACCOUNT-A-ID` & `dev-acc`
        ```sh
        $(aws ecr get-login --registry-ids YOUR-ACCOUNT-A-ID --region eu-central-1 --profile dev-acc)
        ```
    1. Prepare Docker Image
        Change the repo name `my-first-repo` & image `my-first-repo:latest` to suit your environment
        ```sh
        docker tag my-first-repo:latest YOUR-ACCOUNT-A-ID.dkr.ecr.eu-central-1.amazonaws.com/my-first-repo:v1
        ```
    1. Push Image to Repo
        Change the repo name `my-first-repo` to suit your environment
        ```sh
        docker push YOUR-ACCOUNT-A-ID.dkr.ecr.eu-central-1.amazonaws.com/my-first-repo:v1
        ```

## Support
Please open a [GitHub issue](https://github.com/miztiik/cross-account-ecr-access-control/issues/new).

## Feedback
Please open a [GitHub issue](https://github.com/miztiik/cross-account-ecr-access-control/issues/new). I do encourage you to contribute your changes and send me pull request.

##### References
1. [AWS Answers - Grant Cross Account Access](https://aws.amazon.com/premiumsupport/knowledge-center/secondary-account-access-ecr/)

1. [AWS Blogs - Tagging Repositories on ECR](https://aws.amazon.com/blogs/compute/tagging-container-image-repositories-on-amazon-ecr/)
    
1. [AWS Blogs - Tag-Based Access Control](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr-supported-iam-actions-tagging.html)