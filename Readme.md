# Fix Fix_gitlab_error500-runner
## 1- Enter in pod
<code>kubectl exec --stdin --tty $PODNAME -- /bin/bash</code>

## 2- Scan error
<code>gitlab-rake gitlab:doctor:secrets VERBOSE=1</code>

## 3- Enter rails console
<code>gitlab-rails dbconsole</code>
### a. Reset CI/CD variables
<code>SELECT * FROM public."ci_group_variables";</code><br/>
<code>SELECT * FROM public."ci_variables";</code><br/>
<code>DELETE FROM ci_group_variables;</code><br/>
<code>DELETE FROM ci_variables;</code>
    
### b. Reset runner registration tokens
<p>-- Clear project tokens</p>
<code>UPDATE projects SET runners_token = null, runners_token_encrypted = null;</code>
<p>-- Clear group tokens</p>
<code>UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;</code>
<p>-- Clear instance tokens</p>
<code>UPDATE application_settings SET runners_registration_token_encrypted = null;</code><br/>
<code>UPDATE application_settings SET encrypted_ci_jwt_signing_key = null;</code>
<p>-- Clear runner tokens</p>
<code>UPDATE ci_runners SET token = null, token_encrypted = null;</code>
    
### c. Fix project integrations
<p>-- truncate web_hooks table</p>
<code>TRUNCATE web_hooks CASCADE;</code>

### d. Reset setting
<code>settings = ApplicationSetting.last</code><br/>
<code>settings.update_column(:runners_registration_token_encrypted, nil)</code><br/>
<code>ApplicationSetting.first.delete</code><br/>
<code>ApplicationSetting.first</code>

### e. Quit rails console
<code>\q</code>

## 4- Restart Gitlab
<code>gitlab-ctl restart</code>
