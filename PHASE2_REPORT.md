# Project 4 Phase 2: Secure Serverless Website with GitHub CI/CD

## Student Information

- Name: `[Your Name]`
- Course: `CSCE 412 - Cloud Computing`
- Project: `Project 4 Phase 2`
- Date: `April 2026`

## Website Link

- Primary website URL: `https://[your-domain-or-subdomain]`
- GitHub repository URL: `[paste your GitHub repo URL here]`

## Overview

This project extends the secure static website from Phase 1 into a dynamically updateable serverless website. The final design uses GitHub as the source repository and AWS CodePipeline as the deployment pipeline. After the pipeline is configured, the website is updated by making a code change locally, committing the change, and pushing it to GitHub.

The final architecture should ensure that users can access the website only through the custom domain and CloudFront. Direct S3 bucket access should be blocked.

## Final Architecture

1. `GitHub repository` stores the website files.
2. `CodePipeline` watches the GitHub repository for changes.
3. `CodeBuild` is optional for a plain static site. It can be skipped unless you want a build stage.
4. `S3 bucket` stores the website files as the deployment target.
5. `CloudFront` serves the website over HTTPS.
6. `ACM` provides the SSL/TLS certificate in `us-east-1`.
7. `Route 53` routes the custom domain to CloudFront.

## Important Security Requirement

For Phase 2, all endpoints other than the domain and subdomain should be inaccessible. That means:

1. Keep `Block all public access` enabled on the S3 bucket.
2. Do not rely on the public S3 static website endpoint for the final submission.
3. Use CloudFront with an `Origin Access Control (OAC)` or `Origin Access Identity (OAI)` so only CloudFront can read from the bucket.
4. Route traffic through Route 53 and CloudFront only.

If your current site still works directly from the S3 website endpoint, update that before submission because it does not satisfy the stated requirement.

## Steps Taken from Beginning to End

### 1. Start with the secure website from Phase 1

I first completed the secure website setup from Phase 1:

1. Registered a personal domain.
2. Created the S3 bucket named after the domain or subdomain.
3. Uploaded the static website files.
4. Created a hosted zone in Route 53.
5. Requested an ACM certificate in `us-east-1`.
6. Created a CloudFront distribution and attached the certificate.
7. Pointed Route 53 DNS records to CloudFront.

### 2. Create the GitHub repository

I then created a GitHub repository for the website source code.

1. Logged in to GitHub.
2. Created a new repository, for example: `[repo-name]`.
3. Added the website files to the repository.
4. Committed the initial version of the website.
5. Pushed the project to GitHub.

Example commands:

```bash
git init
git add .
git commit -m "Initial website commit"
git branch -M main
git remote add origin https://github.com/[username]/[repo-name].git
git push -u origin main
```

### 3. Prepare the S3 bucket for private access

To satisfy the requirement that only the domain/subdomain should be accessible:

1. Opened the S3 bucket used by the website.
2. Enabled `Block all public access`.
3. Removed any bucket policy or ACLs that made the bucket public.
4. Configured CloudFront to access the bucket privately through OAC or OAI.
5. Updated the bucket policy to allow reads only from the CloudFront distribution.

Example bucket policy pattern:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::[your-bucket-name]/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::[account-id]:distribution/[distribution-id]"
        }
      }
    }
  ]
}
```

### 4. Create the CodePipeline pipeline

I created a deployment pipeline so GitHub pushes automatically update the website.

1. Opened the AWS `CodePipeline` console.
2. Clicked `Create pipeline`.
3. Entered a pipeline name, for example: `[your-pipeline-name]`.
4. Let AWS create a service role automatically.
5. Chose `GitHub` as the source provider.
6. Connected AWS to GitHub using a CodeStar connection if prompted.
7. Selected the repository and branch, usually `main`.
8. Chose one of the following deployment flows:

- Simpler flow for a static site:
  Use `Source -> Deploy to S3`
- Slightly more complete CI/CD flow:
  Use `Source -> Build -> Deploy`

For a plain HTML/CSS/JS site, the simplest valid approach is usually:

1. Source stage: `GitHub`
2. Build stage: `Skip`
3. Deploy stage: `Amazon S3`

### 5. Configure the deploy stage

For deployment:

1. Selected the target S3 bucket.
2. Enabled `Extract file before deploy` if the pipeline artifact was zipped.
3. Created the pipeline and waited for the first run to succeed.

After the deploy completed, the new files were placed into the S3 bucket automatically.

### 6. Connect CloudFront to the updated bucket

Because CloudFront is the public entry point:

1. Confirmed the CloudFront origin points to the S3 bucket, not the public website endpoint.
2. Confirmed the custom domain is listed in `Alternate domain names (CNAMEs)`.
3. Confirmed the ACM certificate is attached.
4. Confirmed Route 53 `A` or `AAAA` alias records point to CloudFront.

### 7. Verify that only the secure domain works

I then verified:

1. `https://[your-domain-or-subdomain]` loads correctly.
2. The S3 bucket URL is not publicly accessible.
3. The S3 website endpoint is not used or is inaccessible.
4. Content updates appear after a GitHub push and pipeline execution.

## How to Update the Website

Once the pipeline is configured, the update process is:

1. Edit the website locally.
2. Save the changed files.
3. Commit the changes with Git.
4. Push the commit to GitHub.
5. Wait for CodePipeline to detect the push and run automatically.
6. Wait for S3 deployment to complete.
7. If needed, wait briefly for CloudFront cache refresh or create an invalidation.
8. Reload the website and verify the new content is live.

Example commands:

```bash
git add .
git commit -m "Update homepage text for Phase 2 demo"
git push origin main
```

## Demonstration of Pre-Change and Post-Change

This section should be completed with your own screenshots.

### Pre-change example

Before pushing the new commit, the website showed:

- Header text: `[old text here]`
- Page version or note: `[old version here]`

Screenshot placeholder:

- `Screenshot 1: website before the GitHub push`
- `Screenshot 2: GitHub commit page showing old state or pending change`

### Change made in the repository

I changed the website by editing one or more files, such as:

- `index.html`
- `css/creative.css`

Example change:

```html
<h1>Version 1 - Before CodePipeline Update</h1>
```

changed to:

```html
<h1>Version 2 - Updated Through GitHub and AWS CodePipeline</h1>
```

### Commit and push

Example:

```bash
git add .
git commit -m "Demonstrate automatic website update"
git push origin main
```

Screenshot placeholder:

- `Screenshot 3: GitHub commit successfully pushed`
- `Screenshot 4: CodePipeline execution in progress`
- `Screenshot 5: CodePipeline execution succeeded`

### Post-change example

After the push completed and the pipeline finished, the website showed the updated content.

Screenshot placeholder:

- `Screenshot 6: website after deployment showing the new text`

## Evidence to Include in Submission

Include screenshots of the following:

1. The GitHub repository main page.
2. The CodePipeline stages and successful execution.
3. The S3 bucket configuration showing it is not public.
4. The CloudFront distribution and custom domain.
5. Route 53 DNS record pointing to CloudFront.
6. The live website loaded through your HTTPS domain.
7. The before-and-after website change.

## Deliverable Summary

1. Steps taken from beginning to end to create a dynamically updateable AWS website using GitHub:
   Completed in the sections above.
2. Link to the website:
   `https://[your-domain-or-subdomain]`
3. Demonstration of pushing a change to GitHub and seeing it on the website:
   Documented in the pre-change, commit/push, pipeline run, and post-change sections.

## Short Conclusion

This project demonstrates a serverless CI/CD workflow for a secure static website. GitHub stores the website source, CodePipeline automates deployment, S3 stores the files privately, CloudFront serves the content securely over HTTPS, and Route 53 provides public DNS for the custom domain. After setup, the website can be updated by pushing a commit to GitHub.
