# Stage 2b: The question reaches OpenShift Lightspeed

This is a plain-language walk-through of one stage on the [sequence diagram page](https://dkypuros.github.io/multicloud-gitops-design/). It describes the experiment in [luis5tb/multicloud-gitops](https://github.com/luis5tb/multicloud-gitops/tree/rca-praxis-ols-claude-a2a-bm) and is not official documentation. For the whole flow, see [How the flow works](how-it-works.md).

When the orchestrator decides a question needs the live cluster, MAS sends it toward OpenShift Lightspeed. It still carries Token A unchanged. It also carries a header called X-OLS-Cluster, which names the cluster to ask. The name is a fixed form of the cluster's address, written with hyphens. The list of known clusters is kept in one place in the repo, and every component reads it from there.

The request first reaches Praxis, a gateway that sits in front of Lightspeed. Praxis can check tokens, but that check is turned off in this setup, so here it only routes. It reads the cluster header and sends the request to that cluster's Lightspeed. If the header is missing or names a cluster it doesn't know, Praxis stops the request. There is no default cluster to fall back to.

The check was turned off for two reasons. Praxis's caller list was written for tokens from the local Keycloak, so it would turn away Red Hat SSO tokens. Also, the Praxis version in use tries to reach Red Hat SSO's key server over a kind of network connection this cluster doesn't have, and it fails. Lightspeed and Keycloak don't have that problem, so the checking moved to them.

One real problem showed up at this point. The orchestrator's language model writes the header name itself, and sometimes it wrote X_OLS_Cluster, with underscores instead of hyphens. Praxis matches the name exactly, so those requests were turned away and the question was silently lost. It happened only some of the time, because the model doesn't write the same thing every time. Stricter instructions and making the model less random reduced it. The lasting fix would be for MAS to set the header itself instead of leaving it to the model.

Lightspeed is where Token A is really checked on this path. It confirms the token is signed by Red Hat SSO, is meant for api.ask_red_hat, and hasn't expired. It also confirms that the cluster named in the header is its own cluster. A bad token is refused as not authenticated. A request meant for another cluster is refused as not allowed.
