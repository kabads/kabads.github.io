---
layout: post
title: AWS CI/CD Pipelines
date: 2022-07-06
categories:
- devops
---

This document will outline how to use an Amazon Web Services (AWS) CodePipeline to push a static html website from a code repository to an S3 bucket. 
<!--more-->
Prerequisites: 
 - AWS account
 - S3 bucket configured to host static html files
 - Git installed locally on your machine

### Contents
 - Create the S3 bucket
 - Create Files for Git Repo
 - Create Git Repo
 - Create the pipeline 
   - Choose the source 
   - Skip the build stage 
   - Choose the deploy provider
   - Review the pipeline
   - Create the pipeline
 - Create a Change and push to Git Repo

### Create the S3 Bucket to host static HTML files

A detailed webpage on how to carry this out is hosted here https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html. 

You will need to enable the static site hosting by allowing public access, by unchecking the block _all_ public access button:

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mYm-AOjoNVzNv0I0A2vOo9d9Yuh2rOqhjudVJoBJHzohKN6qHMXAYNeQcbJ3DAN-wZPTZHBsAWHCFQG50iYyGEkVDp6RrJen6bwuDXQsKG3FyGTKrw5gbHe46IeLHmFFYWsUzvQ9CqPXbPLv9bZNQ1Y9J2Z7BfYDBTGmR84bDa7hykR4kZTwVodLiWrOMyLN4?width=865&height=674&cropmode=none" width="865" height="674" />
{{< /rawhtml >}}

(a dialogue will pop up to confirm that this is what you really want), 

... and by enabling the static website hosting within the properties tab: 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mnvdZCyI4lLwZCd0hJlnT1K6CpPJpcCT_f0uDeViIhvPpfIEKjGA7mklWb3W55n26UlYhxSTof3wiSSx4bIxo3JkgfbZraRMT_s62TK2yFt-k5WvSSlNS2PwTYptAouuNaoa_s-M8f508nGmjX5AdU4AGxrzA0ylU0P4g3u4XgONti7wW6S3pd9Ka0NjFC5Hk?width=256&height=196&cropmode=none" width="256" height="196" />
{{< /rawhtml >}} 


Edit the public policy and paste in the following code, replacing `<bucket-name>` for the name of your S3 bucket:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::<bucket-name>/*"
            }
        ]
    }

When you enable the static hosting, you will have the chance to identify the index document, and the error document. The error document is optional, but in this case, we should enter both. If a page is requested that does not exist, the requester will be sent to the error document (just like a 404 html page). 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mHrv1Zg6AggA83_QUc8cIJq2Olz8yfKyV31Tqz5bfZ_zylh7pAR41yYZ1TCWooz5ykNfQSbHKESylQjPLE-bdHFfrhjOrUH9UYe4v5Knep5TGywNctxKKmBVTFJ8qQ9b-n35wUHyAEbEM7B9s8bykuWJU2E6Xe9ZWbAUSYc2GMyO70o3CNJzSWcYIYVqDntIX?width=660&height=531&cropmode=none" width="660" height="531" />
{{< /rawhtml >}}

### Create files for the Git Repository

Create a file named `index.html` with the following code: 
    
    <html>
      <head>
        <title>Sample Website</title>
      </head>
      <body>
        <h1>Your Sample Site</h1>
        <p>Website version 1.0</p>
      </body>
    </html>
 
 and a file named `error.html` with the following contents:

    <html>
      <head>
        <title>Page not found</title
      </head>
      <body>
        <h1>Page not found</h1>
      </body>
    </html>

### Create a git repository

Navigate to CodeCommit and click create a repository. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mt8gMzNFIfZ7wy8ZyMfz24UEvefV2oUq9j_glhNNU7wDCQqrOnUpfdMGq7cgjvgb1fQqGvOi6JVfaQNom90VEOPKMyI7O_Agn5Bj11JonNf8CgPl0Hvwpa0mEQsI_CZMdMvipEtokAHv5ZHukpEA2f9aUx8w-52Tnrkt2eakUBG10Gb3FfXjQrPSC73meI2LU?width=1024&height=150&cropmode=none" width="1024" height="150" />
{{< /rawhtml >}}

Name your repository, and give a description if necessary. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mf5yqdOv0EGFCsit_Go6IYKpUgE6JzoR11x6bFxsNfY-Ltf55hMy9kBOxJFEHSuf_YztVRkEfV_qGADIY_O47CDrJbFMY0ZOXYrUoBrTLAKwIlRPJ00W-tB2oeBK6ivz3B-_tLXHxzTQ_1z51H0WxsU07AiDYjxe1zcsnSj7v-PQqSPUrn9EGIYakm473gg5W?width=660&height=473&cropmode=none" width="660" height="473" />
{{< /rawhtml >}}

If you are using a root user, you will not be able to commit to the website via ssh. You will only be able to use HTTPS. It is not recommeneded to use HTTPS for a root user, so if necessary, create a new user, via IAM service, and give that user access to AWS CodeCommit. This new user should have a public and private part of an ssh key. You can find more information on this at https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html. 

Then, navigate to CodeCommit and click the clone URL for SSH. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mNOM5i03momyOEL-q3G6LNPzBS1bX7TgUEGJlhgMsxbohjEm6hqyILy-FafxnCpFUf0cbSxjSMrqYZuqKrZkO64XOMA5ryiSuhKOtG_EcKGv_Kp_sanTfVSk-0OlerRuvFCeKvy5c5ugoD8xw0eSL5YUWkvgqpy-ZpHhk7mlbHu828T6VKqlDuV4IKtK8L2ow?width=1024&height=35&cropmode=none" width="1024" height="35" />
{{< /rawhtml >}}

Then, open a terminal on your local machine, and type `git clone` and paste the URL that you copied in the previous step. Then press enter. Git will download the repository in to a new directory. You will need to change in to that directory with `cd <your-directory>`. 

This is where your two files, `index.html` and `error.html` need to be. Place them here, (`cp <path-to-file>/index.html .` and `cp <path-to-file>/error.html .`. 

Add these files to the repository and push them with these commands:

    git add index.html
    git add error.html 
    git commit -m"Initial commit"
    git push

You will be able to browse to the CodeCommit repository and see the files there. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mnmtpxsz2Fla4yZQbyxRMHokVWsmBkRgldypbZnnyHoQAdH6W7esvpsP-JyeFncRZ65YyT530jMtP1YkZ-c_Pt-Jxl2MiPu6J-gkW-DBN8KdLzTKNbgQ8EdZVzCC8-2qHdadrW70L_LeS-IRs1KVbYZN-Bh8x7Ced9LaNHNbM0nqjHHREteRRtHCikBCjqZ-f?width=565&height=340&cropmode=none" width="565" height="340" />
{{< /rawhtml >}}

### Create the pipeline

Navigate to the CodePipeline page within the AWS console. 

Click Create Pipeline. Name the pipeline. The pipeline needs an IAM role - this step will create one for you. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4ml2YrJtR-eljrKGYrNf2POnLNgKYE2yjP-MXTAiseAV5boXhdZI6O_Jx6du4nQWZdkNerqvmEKft4DkcfWoKVXVq-ASXxAw5Ita1vCtPWoY1HK-XYYHqJIzbaJb02zctKny8fl_H39NpLKCxlXn-1R5Pk6JlQc-Ju151zZTc-VvBfR3Ee9aFLg7raNdk8IrHe?width=660&height=334&cropmode=none" width="660" height="334" />
{{< /rawhtml >}}

#### Choose the Source Setting

Choose artifact store location. You can choose the aws code repository. 

Add source stage - you choose the source - here it will be your code repository. You will also need to choose the branch of the repo. Choose `master` or `main`, depending on your repo. 

When choosing the Output Artifact format, you will be able to choose from a CodePipeline default, or Full clone. The full clone checks out the code, and needs an extra permission. For the sake of this tutorial, we will choose the CodePipeline default, which provides a zip package, which will then be unzipped in the S3 bucket. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4m-JfS6M23PAKo9g_kgHgizWsYIGvyxMtuUsOXuqjswDKP35SPTBbVv6eBUst7-gUFP7U_DId6lrXYXP_gja8jvTh1tTe-dYH2lT6N-mbBk0tdJ8xWNUWJw34ijjUzYcl-ysion3H0Sb66ga156ngT3nBGb9j6gHQMWJmvF1lwhaj4tsQK3xsmoQ3sQ4Ajd18P?width=792&height=631&cropmode=none" width="792" height="631" />
{{< /rawhtml >}}

You should choose the change detection options. You can choose AWSCodePipeline here for the changes (webhook). 

#### Skip the Build stage

Skip Build stage (you do not need to build the static html files). 
You need a deploy stage. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mKAQg2Z_QfhRIkqpopFnZkxnIgUQUKBnZgS5rVfEhlRbJpwLW3Mytie-KuiLCh7cjpPW49UB3aoaVAWYc8jbgV0evWEpYHmSaNXojBm3tkQsQPsasTkhyTA-6MSBIHlQ5q4xnGdp5-FwYVruV9Wjn-AuiFCU9uzohQqStiVaNuOLoWa1lXNG9CfXiS85WMCbU?width=862&height=308&cropmode=none" width="862" height="308" />
{{< /rawhtml >}}


#### Choose Deploy Provider
Choose a Deploy Provider. You will deploy here to S3, so choose AWS S3. Choose the bucket. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mXgc_pUDwzmTkqrIuWyWE4bjO4tGdeT67EsS9aeDtSO_C87Mp4fUQF4Ydqu6oN4l66y2XwhRA_6g6MAQUqE_LN92LHtM9MF8-HZZIedPHtFINCXbJ0HXcLyUp2jechTj5dk8lYcD2lvxuxtglKcppvRqfNv4wdgkZpw2J_arQ4tLA6pyql0ZTmZYZcoybqM6g?width=889&height=739&cropmode=none" width="889" height="739" />
{{< /rawhtml >}}

Ensure that unzip artifact is ticked before publishing.

Under Additional Configuration, you will __not__ need to select `public-read` under the `Canned ACL - optional` setting. In this case, nothing should need changing. 

#### Review the pipeline

Then you need to review your pipeline. Create the pipeline. 

#### Create Pipeline

At the bottom of the review pipeline, you will see create pipeline. You can click this, and it will run the pipeline for the first time. 

Then you should grab the URL endpoint for your S3 static website. 

{{< rawhtml >}}
<img src="https://by3302files.storage.live.com/y4mTKWMI31641M_mLygD2FJ9jaxQSAvCE40ZI9pJxAbKz5_2yOWrnrYT0UkQMcnbpEfdijNXzMOPhoV9U6EqW8IwlGXyk2noGIJ5PPBNm8rVLrQtj1VCKdITi3Jo3W8qT7cD5m2Hq1FwW0gTWNfIm3ilafJYkprieysuiaOvqqoPhi4VNxMFdwyfblaePeDv1By?width=1024&height=307&cropmode=none" width="1024" height="307" />
{{< /rawhtml >}}

#### Create a change and push to the Git Repo

If you edit `index.html` or `error.html` and then push to the git repo, and then deploy to AWS S3 bucket. 

    git add index.html
    git commit -m"Updating version"
    git push

Once you push, you will notice that the pipeline runs, and that your changes are pushed to the S3 bucket. 
