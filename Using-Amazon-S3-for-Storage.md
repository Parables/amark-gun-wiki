You need to use this Amazon S3 policy (replace both "YOURBUCKETHERE" with the S3 bucket you make/have for this app):

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::YOURBUCKETHERE",
                "arn:aws:s3:::YOURBUCKETHERE/*"
            ]
        }
    ]
}
```

For your Amazon IAM User (in Security Credential's admin dashboard, create a custom policy first then create a new user, select it and attach that policy to it) that is associated with that user's Access Key and Secret Access Key that you will be using for your NodeJS peer.

Now add the following environment variables: (for Heroku, this can be done by going to the `Settings` tab in their app dashboard, and going to the `Config Vars` section)

| VARIABLE  | VALUE  |
|---|---|
| `AWS_ACCESS_KEY_ID`  | `paste your key here `  |
| `AWS_S3_BUCKET`  | `paste bucket key here`  |
| `AWS_SECRET_ACCESS_KEY`  | `paste secret here`  |
| `NPM_CONFIG_PRODUCTION` | `false` |

> Note: `AWS-SDK` devDependencies gets removed by Heroku after build, which may cause your app to crash. You will need to set `NPM_CONFIG_PRODUCTION` to `false` to to fix this.

If you are running your own server, you probably have an upstart script or something similar that monitors your app and starts or restarts it if it crashes. You can set ENVIRONMENT VARIABLES there, exactly how depends upon your setup so please google around for that.

Or if you are using command line directly, try something like:

$`VARIABLE1=value1 VARIABLE2=value2 node yourapp.js`

They will show up in NodeJS as `process.env.AWS_ACCESS_KEY_ID` and so on. They could also be set in NodeJS with `process.env.AWS_ACCESS_KEY_ID = 'value';` but that **is not recommended** because it could leak your access keys to GitHub or other places.

