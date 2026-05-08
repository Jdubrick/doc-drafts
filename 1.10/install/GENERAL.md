---
Section:
```
https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.9/html/interacting_with_red_hat_developer_lightspeed_for_red_hat_developer_hub/supported-architecture-for_interacting-with-developer-lightspeed-for-rhdh
```
Changes:
  * No more Llama Stack server --> Only Lightspeed Core now 
  * We don't really want to talk about Llama Stack anymore, we just want to interface with Lightspeed Core

---
Section:
```
https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.9/html/interacting_with_red_hat_developer_lightspeed_for_red_hat_developer_hub/install-and-configure_interacting-with-developer-lightspeed-for-rhdh
```
Changes:
* Don't need a custom plugins configmap to add lightspeed
  * Included as part of the Helm chart or Operator
* There is significant changes with the new setup, would recommend upgrading from a fresh install (i.e. no 1.9 setup present for Lightspeed)
* Dont need to create any configmaps unless you want to override anything, that will be explained in the Helm/Operator sections individually
  * Only one you might have to update if the app-config for RHDH to specify lightspeed prompts etc
* Secrets handling will be specific to Helm/Operator --> should now see those sections for creating them 
* dont need to manually add things the Backstage Deployment anymore, it is handled via Operator or Helm
* We need to add an air-gapped section, I believe it will be the same process as RHDH (might be able to reuse/reference those docs)


---

* Should coordinate with UI team for any UI related items / screenshots they may have to add