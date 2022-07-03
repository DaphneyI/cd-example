this example uses aws federation to grant access to services in an aws account without the use of service account keys.

it works almost the same as on gcp. first you create a provider. for circleci, the provider is `https://oidc.circleci.com/org/<organization-id>`
and audience is `<organization-id>`
where the organization-id is the id of your circleci org and can be found under organization settings in the nav bar of the circleci application

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