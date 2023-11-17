1- Enter in pod
kubectl exec --stdin --tty <pod name> -- /bin/bash

2- Scan error
gitlab-rake gitlab:doctor:secrets VERBOSE=1

3- Enter rails console
gitlab-rails dbconsole
a. Reset CI/CD variables
SELECT * FROM public."ci_group_variables";
SELECT * FROM public."ci_variables";
DELETE FROM ci_group_variables;
DELETE FROM ci_variables;
    
b. Reset runner registration tokens
-- Clear project tokens
UPDATE projects SET runners_token = null, runners_token_encrypted = null;
-- Clear group tokens
UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;
-- Clear instance tokens
UPDATE application_settings SET runners_registration_token_encrypted = null;
UPDATE application_settings SET encrypted_ci_jwt_signing_key = null;
-- Clear runner tokens
UPDATE ci_runners SET token = null, token_encrypted = null;
    
c. Fix project integrations
-- truncate web_hooks table
TRUNCATE web_hooks CASCADE;

d. Reset setting
settings = ApplicationSetting.last
settings.update_column(:runners_registration_token_encrypted, nil)
ApplicationSetting.first.delete
ApplicationSetting.first

e. Quit rails console
\q

4- Restart Gitlab
gitlab-ctl restart
