## General Description

this example uses aws federation to grant access to services in an aws account without the use of service account keys.

it works almost the same as on gcp. first you create a provider. for circleci, the provider is `https://oidc.circleci.com/org/<organization-id>` and audience is `<organization-id>`, where the organization-id is the id of your circleci org and can be found under organization settings in the nav bar of the circleci application

next you create a role. this role must be of type web-identity and you select the provider you created above. once this is done, you specify the permissions that the role should have. you do this either by choosing from one of the existing policies or creating a new policy. remeber to use the principle of least privilege.

once the provider and role have been created, you can authenticate to aws usig the `aws sts` command.
```
aws sts assume-role-with-web-identity \
--role-arn <arn of the role you created> \
--role-session-name <any string you choose to name the session>
--duration-seconds <how many seconds the session should be>
--web-identity-token $CIRCLE_OIDC_TOKEN
```

please note that the variable `$CIRCLE_OIDC_TOKEN` which holds the OIDC token that authenticates circleci to aws only exists for workflows that use a `context`.

calling the aws sts command without specifying a region and an endpoint by default calls the global sts endpoint. it is best practice to use a regional sts endpoint close to the resources you want to reach as this reduces latency and improves response times.</br>
for example, if you want to aacces resources in us-west-2, to use sts for us-west-2 we would have to specify a region and the regions sts endpoint
```
aws sts assume-role-with-web-identity \
--role-arn <arn of the role you created> \
--role-session-name <any string you choose to name the session>
--duration-seconds <how many seconds the session should be>
--web-identity-token $CIRCLE_OIDC_TOKEN
-- region <region> \
--endpoint-url "https://sts.<region>.amazonaws.com" \
```

## CircleCI Config
this config specifies a `assume-role-with-web-identity` command
this command contains steps to execute a script which  call `aws sts assume-role-with-web-identity` which generates an access id, secret key and session token. to use these credentials in other steps within the same job, we have to configure aws on the runner machine. to do this we use the command `aws configure set` to set the session token, access id, secret key and region. so subsequen aws commands in subsequent jobs would authenticate successfully as long as the session token is valid.