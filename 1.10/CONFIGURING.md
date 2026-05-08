Section:
```
https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.9/html/interacting_with_red_hat_developer_lightspeed_for_red_hat_developer_hub/customize_interacting-with-developer-lightspeed-for-rhdh
```

* Section about toggling feedback can stay but its important the only thing changed are the booleans, not the paths
  * For feedback and transcripts
* Prompt section can stay

Section:
```
https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.9/html/interacting_with_red_hat_developer_lightspeed_for_red_hat_developer_hub/get-ai-assisted-help-for-your-development-tasks_interacting-with-developer-lightspeed-for-rhdh
```
* No longer using llama safety guard, there is a developer focused validation you can turn on/off with environment variables that will restrict questions to developer tools, languages, RHDH, things like that. It will restrict things like asking for pizza recipes.
  * No longer have 2 different configs, all in 1 config and toggled via env vars that are all in ENABLING_LLMS.md in this repo.
* 