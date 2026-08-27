# MyWebsite IaC
Infraestructura as Code as CloudFormation templates in AWS to deploy and managed all the resources used by MyWebsite.

## Deployment order

The two templates depend on each other: `MyWebsiteContactForm.yaml` imports the API ID exported by `MyWebsiteApi.yaml`, so the API stack must exist first.

1. Deploy `MyWebsiteApi.yaml` as its own stack (no parameters required).
2. Zip and upload the Lambda code from [`contactForm`](https://github.com/jaimeelso/contactForm) (`lambda.mjs` + its `node_modules`) to an S3 bucket you control.
3. Deploy `MyWebsiteContactForm.yaml` as a separate stack, passing:
   - `SubscriptionEndpoint`: the email address that should receive contact form notifications (AWS will send a confirmation email to it after the first deploy — it must be confirmed before notifications start arriving).
   - `TurnstileSecretKey`: the secret key from your Cloudflare Turnstile widget.
   - `LambdaS3Bucket` / `LambdaS3Key`: where the zipped Lambda code from step 2 lives.
   - `ApiStackName`: the stack name you used in step 1.

Both stacks can be deployed via the AWS Console (CloudFormation > Create stack > Upload a template file) or the AWS CLI (`aws cloudformation deploy --template-file <file> --stack-name <name> --parameter-overrides ...`).

Updating the Lambda code alone (without changing the template) requires re-uploading the zip to `LambdaS3Bucket`/`LambdaS3Key` and then updating the `MyWebsiteContactForm` stack (or running `aws lambda update-function-code` directly) so it picks up the new S3 object version.
