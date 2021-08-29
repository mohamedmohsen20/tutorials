# 

[YouTube Tutorial]()

- Create `data.devopsbyexample.io` S3 bucket

- Create IAM Policy `FooS3ListAccess`
Grants programmatic read-write access to the `data.devopsbyexample.io` bucket:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

- Create IAM role `foo-s3-access` and attach `FooS3ListAccess` IAM Policy

- Update Trust Relationship to allow only `foo` service account to assume it
```
aud -> sub
sts.amazonaws.com -> system:serviceaccount:production:foo
```

- Create pod with default account and run `aws sts get-caller-identity`

```bash
kubectl exec -it foo -- bash
```

/usr/local/bin/aws sts get-caller-identity

/usr/local/bin/aws s3 ls