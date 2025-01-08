# 🗝 Keycloak

## Resolve Local Naming

Keycloak is accessed via two routes: host.docker.internal (from the backend) and localhost (from the frontend). To ensure consistent access, an additional hostname is introduced that works for both.

Add the following entry to your /etc/hosts file to resolve keycloak.internal to your local machine:

```sh
echo -e '127.0.0.1\tkeycloak.internal' | sudo tee -a /etc/hosts
```

Verify the setup by running:

```sh
ping keycloak.internal
```

This resolves keycloak.internal to 127.0.0.1 for both backend and frontend access.
