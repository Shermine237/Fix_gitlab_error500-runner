    a. Reset CI/CD variables
    gitlab-rails dbconsole
    SELECT * FROM public."ci_group_variables";
    SELECT * FROM public."ci_variables";
    DELETE FROM ci_group_variables;
    DELETE FROM ci_variables;
    
    b. Reset runner registration tokens
    gitlab-rails dbconsole
    -- Clear project tokens
    UPDATE projects SET runners_token = null, runners_token_encrypted = null;
    -- Clear group tokens
    UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;
    -- Clear instance tokens
    UPDATE application_settings SET runners_registration_token_encrypted = null;
    UPDATE application_settings SET encrypted_ci_jwt_signing_key = null;
    -- Clear runner tokens
    UPDATE ci_runners SET token = null, token_encrypted = null;
    
    c. Reset pending pipeline jobs
    sudo gitlab-rails dbconsole
    -- Clear build tokens
    UPDATE ci_builds SET token = null, token_encrypted = null;
    
    d. Fix project integrations
    gitlab-rails dbconsole
    -- truncate web_hooks table
    TRUNCATE web_hooks CASCADE;


gitlab-rails console
settings = ApplicationSetting.last
settings.update_column(:runners_registration_token_encrypted, nil)
ApplicationSetting.first.delete
ApplicationSetting.first
exit
exit
gitlab-ctl restart
