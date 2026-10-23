# Stage 3: Lightspeed swaps Token A for a Keycloak token

This is a plain-language walk-through of one stage on the [sequence diagram page](https://dkypuros.github.io/multicloud-gitops-design/). It describes the experiment in [luis5tb/multicloud-gitops](https://github.com/luis5tb/multicloud-gitops/tree/rca-praxis-ols-claude-a2a-bm) and is not official documentation. For the whole flow, see [How the flow works](how-it-works.md).

The OpenShift cluster doesn't accept Red Hat SSO tokens. It was set up to trust only the Keycloak server that runs next to it. So before Lightspeed can look at anything on the cluster for this caller, it has to turn Token A into a token the cluster will accept. The standard way to do that is called a token exchange, and it's described in RFC 8693.

Lightspeed has to prove who it is to Keycloak, and it doesn't use a stored password for that. It asks SPIRE, a service on the cluster that issues identities to running software, for a short-lived signed identity. That identity names the exact namespace and service account Lightspeed runs under. It is addressed to Keycloak, so it can't be used anywhere else.

Lightspeed then sends Keycloak one request with two things in it. Its own identity proves Lightspeed is the one asking. Token A says whose request this is. It also tells Keycloak which issuer Token A came from and asks for a new token meant for the cluster's tool service, called openshift-mcp.

Keycloak does several checks. It confirms that Lightspeed's identity matches the identity it was told to expect for Lightspeed. If Lightspeed has moved to a different namespace since then, this check fails, which really happened once. Keycloak then asks Red Hat SSO whether Token A is still active and who it belongs to. That second lookup is the one that needs the openid scope from Stage 1. Keycloak then records the caller as a user it knows about, linked to the Red Hat SSO identity. A rule set up for each SSO login service puts that user in the group acme-agent-rca. That rule was added after a real failure: without it, the new token had no group, and the cluster treated the caller as an individual user with no permissions.

Keycloak then issues Token B. Token B is signed by Keycloak and meant only for the cluster's tool service. It names Lightspeed as the service holding it, still represents the original caller, and carries the group acme-agent-rca.

Token A goes no further. Nothing past Lightspeed ever sees it. The cluster's tool service and the cluster itself only ever see Token B.
