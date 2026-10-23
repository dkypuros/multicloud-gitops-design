# Stage 2: MAS validates the token and picks a specialist

This is a plain-language walk-through of one stage on the [sequence diagram page](https://dkypuros.github.io/multicloud-gitops-design/). It describes the experiment in [luis5tb/multicloud-gitops](https://github.com/luis5tb/multicloud-gitops/tree/rca-praxis-ols-claude-a2a-bm) and is not official documentation. For the whole flow, see [How the flow works](how-it-works.md).

ACME sends the question to MAS with Token A attached. MAS is set up to accept tokens from three issuers: Red Hat SSO production, Red Hat SSO stage, and the local Keycloak. It fetches each issuer's public signing keys and keeps them, so it doesn't have to ask on every request.

MAS checks Token A in a fixed order. The signature must match the issuer's keys. The issuer must be one of the three it trusts. The token must not have expired. It must say it is meant for api.ask_red_hat. Last, the service account inside it must appear on MAS's list of allowed callers. That list holds exact values, and the token has to match one of them. A broken or missing token gets a refusal saying the caller isn't authenticated. A valid token from a caller who isn't on the list gets a refusal saying the caller isn't allowed.

Once the token passes, MAS keeps it for the rest of this request and starts a saved workflow called a blueprint. The blueprint begins with an orchestrator, which is a language model with instructions. Those instructions tell it to send questions about the live cluster to Lightspeed, send documentation and product questions to Ask Red Hat, and use both when the answer needs both.

When the orchestrator picks Ask Red Hat, MAS calls the Ask Red Hat service on Red Hat's infrastructure. The step for Ask Red Hat has no token of its own written into it. It is left empty on purpose, which tells MAS to forward the token it already holds, so Ask Red Hat receives Token A unchanged. Ask Red Hat checks the token on its side and returns guidance from documentation and product knowledge. That check happens outside the repo this page describes.

After each answer comes back, the orchestrator looks again and decides whether it still needs the other agent or can finish. When it finishes, MAS combines the results into one answer. The instructions ask it to keep what was actually seen on the cluster separate from what the documentation says, so advice is never presented as a fact about the cluster.
