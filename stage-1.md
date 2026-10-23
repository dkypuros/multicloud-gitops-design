# Stage 1: The agent gets a Red Hat SSO token

This is a plain-language walk-through of one stage on the [sequence diagram page](https://dkypuros.github.io/multicloud-gitops-design/). It describes the experiment in [luis5tb/multicloud-gitops](https://github.com/luis5tb/multicloud-gitops/tree/rca-praxis-ols-claude-a2a-bm) and is not official documentation. For the whole flow, see [How the flow works](how-it-works.md).

In this demo, the outside agent is played by ACME, a chat agent with its own web page. A person types a question there. Along with the question, the page tells ACME which login service to use for this request: the local Keycloak, or Red Hat SSO in production or stage. In the demo it picks Red Hat SSO stage. That choice applies to this one request only, so two conversations open at the same time can use different login services without getting mixed up.

ACME first has to decide where the question should go. It asks its own language model, which it reaches through a service called LiteLLM. That connection uses a fixed key that belongs to ACME's deployment. The key says nothing about who is asking and has nothing to do with the tokens described below. The model's answer is always the same here: send Red Hat questions to MAS.

Before ACME can call MAS, it needs a token. It sends Red Hat SSO the client ID and secret that were registered for it. Those come from a secrets store and are never shown to the browser. It also asks for two scopes. One, called api.ask_red_hat, says what the token is for, and MAS, Lightspeed and Ask Red Hat all look for it. The other, called openid, matters later. When Keycloak checks this token during the exchange, it calls Red Hat SSO to look up the caller, and that lookup is refused unless the token was issued with openid.

Red Hat SSO checks the ID and secret and returns Token A. Token A says it was issued by Red Hat SSO, that it may be used for api.ask_red_hat, and which service account it belongs to. That service account value is what MAS checks against its allowed list in the next stage. It is not the same as the client ID ACME logged in with, which caused a real failure while this was being built. ACME keeps Token A until it expires and reuses it in the meantime.

If Red Hat SSO refuses, ACME now shows the reason SSO gave. Earlier, the only visible symptom was a vague error somewhere else. That is how the team found that the SSO client had to be allowed to act as a service account before any of this could work.
