* Section about toggling feedback can stay but its important the only thing changed are the booleans, not the paths
  * For feedback and transcripts
* Prompt section can stay
* No longer using llama safety guard, there is a developer focused validation you can turn on/off with environment variables that will restrict questions to developer tools, languages, RHDH, things like that. It will restrict things like asking for pizza recipes.
  * No longer have 2 different configs, all in 1 config and toggled via env vars
* 