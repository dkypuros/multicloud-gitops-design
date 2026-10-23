# multicloud-gitops-design

An experiment in diagramming the end-to-end authentication and routing flow built by
Luis Tomas Bolivar and Mario in [luis5tb/multicloud-gitops](https://github.com/luis5tb/multicloud-gitops/tree/rca-praxis-ols-claude-a2a-bm)
(branch `rca-praxis-ols-claude-a2a-bm`).

**Live page:** https://dkypuros.github.io/multicloud-gitops-design/

The flow, in short:

1. An external agent gets a token from Red Hat SSO.
2. MAS validates it and routes the question to Ask Red Hat (on Red Hat infrastructure) or to
   OpenShift Lightspeed (on an OpenShift cluster), or to both.
3. On the Lightspeed path, Keycloak exchanges the SSO token (RFC 8693) for one that maps to an
   OpenShift group, so the cluster applies proper RBAC.

This is not official documentation. The authoritative write-up is
[AUTHENTICATION.md](https://github.com/luis5tb/multicloud-gitops/blob/rca-praxis-ols-claude-a2a-bm/agents/AUTHENTICATION.md)
in Luis's repo. Diagrams reflect commit `c6d8ffd4`.

The page is a single static `index.html` using Mermaid for the sequence diagrams.
