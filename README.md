# Crossplane Demo

## Install Crossplane 
```
helm upgrade --install     crossplane crossplane-stable/crossplane     --namespace crossplane-system     --create-namespace     --wait
```

## Install AWS Provider
We will install both [AWS](https://github.com/crossplane-contrib/provider-aws) and [AWS Jet](https://github.com/crossplane-contrib/provider-jet-aws) provider 

```
❯ k apply -f provider-aws.yaml
provider.pkg.crossplane.io/provider-aws created
provider.pkg.crossplane.io/provider-jet-aws created

❯ k get providers
NAME               INSTALLED   HEALTHY   PACKAGE                              AGE
provider-aws       True        Unknown   crossplane/provider-aws:v0.30.1      13s
provider-jet-aws   True        Unknown   crossplane/provider-jet-aws:v0.5.0   13s
```

It will take some time for provider to be healthy, as it will download and install the provider.
You can confirm that the aws provider pod is running in crossplane-system namespace.

```
❯ k get po -n crossplane-system
NAME                                           READY   STATUS    RESTARTS   AGE
crossplane-rbac-manager-8466dfb7fc-h4g7t       1/1     Running   0          7m15s
crossplane-7c88c45998-xfx82                    1/1     Running   0          7m15s
provider-jet-aws-599c4ea494ca-7d8d575c-n8jq4   1/1     Running   0          3m33s
provider-aws-f72884c6b119-5bf5454658-jd9wb     1/1     Running   0          3m17s
```

Since we installed both the providers there will be some overlapping CRD's and resources e.g. VPC 
```
❯ k api-resources | grep -w vpcs
vpcs                                                  ec2.aws.crossplane.io/v1beta1                        false        VPC
vpcs                                                  ec2.aws.jet.crossplane.io/v1alpha2                   false        VPC

```

In that case the v1bet1 takes precedence. 

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
❯ kubectl create secret generic aws-enbuild-dev -n crossplane-system --from-file=creds=/Users/junedm/.aws/enbuild_dev.conf
secret/aws-enbuild-dev created
```

Now, you can create the providerconfig, we will create the provider config with both the providers, but as discussed the v1beta1 will take precedence.
```
❯ k apply -f provider-config.yaml
providerconfig.aws.jet.crossplane.io/aws-enbuild-dev created
providerconfig.aws.crossplane.io/aws-enbuild-dev created

❯ k get providerconfig
NAME              AGE
aws-enbuild-dev   65s
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