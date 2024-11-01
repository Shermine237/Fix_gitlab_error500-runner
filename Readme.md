# Fix Fix_gitlab_error500-runner
# Method 1
## 1- Enter in pod
```shell
kubectl exec --stdin --tty $PODNAME -- /bin/bash
```

## 2- Scan error
```shell
gitlab-rake gitlab:doctor:secrets VERBOSE=1
```

## 3- Enter rails console
```shell
gitlab-rails dbconsole
```
### a. Reset CI/CD variables
```shell
SELECT * FROM public."ci_group_variables";
SELECT * FROM public."ci_variables";
DELETE FROM ci_group_variables;
DELETE FROM ci_variables;
```
    
### b. Reset runner registration tokens
<p>-- Clear project tokens</p>

```shell
UPDATE projects SET runners_token = null, runners_token_encrypted = null;
```
<p>-- Clear group tokens</p>

```shell
UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;
```
<p>-- Clear instance tokens</p>

```shell
UPDATE application_settings SET runners_registration_token_encrypted = null;
UPDATE application_settings SET encrypted_ci_jwt_signing_key = null;
```
<p>-- Clear runner tokens</p>

```shell
UPDATE ci_runners SET token = null, token_encrypted = null;
```
    
### c. Fix project integrations
<p>-- truncate web_hooks table</p>

```shell
TRUNCATE web_hooks CASCADE;
```

### d. Reset setting
```shell
settings = ApplicationSetting.last
settings.update_column(:runners_registration_token_encrypted, nil)
ApplicationSetting.first.delete
ApplicationSetting.first
```

### e. Quit rails console
```shell
\q
```

## 4- Restart Gitlab
```shell
gitlab-ctl restart
```

# Method 2 if still failed
## Fix
```shell
gitlab-rails console
ApplicationSetting.first.delete
ApplicationSetting.first
exit
```
## 4- Restart Gitlab
```shell
gitlab-ctl restart
```
