# Crossplane Demo

## Install Crossplane 
```
helm upgrade --install     crossplane crossplane-stable/crossplane     --namespace crossplane-system     --create-namespace     --wait
```

## Install AWS Provider
```
k apply -f provider-jet-aws.yaml

k get providers
NAME               INSTALLED   HEALTHY   PACKAGE                              AGE
provider-jet-aws   True        True      crossplane/provider-jet-aws:v0.5.0   4m25s
```

It will take some time for provider to be healthy, as it will download and install the provider.
You can confirm that the aws provider pod is running in crossplane-system namespace.
```
k get po -n crossplane-system
NAME                                           READY   STATUS    RESTARTS   AGE
crossplane-rbac-manager-8466dfb7fc-wskhq       1/1     Running   0          54m
crossplane-7c88c45998-jd2s2                    1/1     Running   0          54m
provider-jet-aws-599c4ea494ca-7d8d575c-fl2g4   1/1     Running   0          5m27s
```

## Create the ProviderConfig 
Before creating the ProviderConfig, we need to create a kubernetes secret in crossplane-system with our AWS account credentails.
For that, you can create the aws conf file with aws cli format , like below. 

```
cat /Users/junedm/.aws/enbuild_dev.conf
[default]
aws_access_key_id = AWS_SECRET_ACCESS_KEY
aws_secret_access_key = AWS_SECRET_ACCESS_KEY
region = us-east-1
```

And then create a secret
```
kubectl create secret generic aws-enbuild-dev -n crossplane-system --from-file=creds=/Users/junedm/.aws/enbuild_dev.conf
```

Now, you can create the providerconfig
```
k apply -f provider-config.yaml

k get providerconfig
NAME              AGE
aws-enbuild-dev   17s
```

# Create the AWS Resources

You can create the AWS resources e.g. VPC is the easy example 
```
❯ k apply -f vpc.yaml
vpc.ec2.aws.jet.crossplane.io/sample-vpc unchanged

❯ k get vpc
NAME         READY   SYNCED   EXTERNAL-NAME           AGE
sample-vpc   True    True     vpc-0a3f0db085f584bc5   3m30s
```