![Travis-CI build status](https://travis-ci.org/kamax-io/mxisd.svg?branch=master)

# Introduction
mxisd is an implementation of the Matrix Identity Server which aims to provide an alternative
to [sydent](https://github.com/matrix-org/sydent) and an external validation implementation of the
[Identity Service API](http://matrix.org/docs/spec/identity_service/unstable.html).

## Contact
If you need help, want to report a bug or just say hi, you can reach us at [#mxisd:kamax.io](https://matrix.to/#/#mxisd:kamax.io)

For more high-level discussion about the Identity Server architecture/API, go to [#matrix-identity:matrix.org](https://matrix.to/#/#matrix-identity:matrix.org)

## How does it work
Given the 3PID `john.doe@example.org`, the following could be performed until a mapping is found:
- LDAP: lookup the Matrix ID (partial or complete) from a configurable attribute.
- DNS: lookup another Identity Server using the domain part of an e-mail and:
  - Look for a SRV record under `_matrix-identity._tcp.example.org`
  - Lookup using the base domain name `example.org`
- Forwarder: Proxy the request to other configurable identity servers.

The lookup strategy will use a priority order and a configurable recursive/local type of request.

# Quick start
## Requirements
- JDK 1.8

## Build
```
git clone https://github.com/kamax-io/mxisd.git
cd mxisd
./gradlew build
```

## Configure
1. Create a new local config: `cp application.example.yaml application.yaml`
- Set the `server.name` value to the domain value used in your Home Server configuration
- Provide the LDAP attributes you want to use for lookup
- Edit an entity in your LDAP database and set the configure attribute with a Matrix ID (e.g. `@john.doe:example.org`)

## Run
Start the server in foreground:
```
./gradlew bootRun
```

Ensure the signing key is available:
```
curl http://localhost:8090/_matrix/identity/api/v1/pubkey/ed25519:0
```

Validate your LDAP config and binding info (replace the e-mail):
```
curl "http://localhost:8090/_matrix/identity/api/v1/lookup?medium=email&address=john.doe@example.org"
```

If you plan on testing the integration with a homeserver, you will need to run an HTTPS reverse proxy in front of it
as the reference Home Server implementation [synapse](https://github.com/matrix-org/synapse) requires a HTTPS connection
to an ID server.

# Install
Run all the following commands as `root` or using `sudo`

1. Create a dedicated user: `useradd -r mxisd`
- Create config directory: `mkdir /etc/mxis`
- Change user ownership of `/etc/mxis` to dedicated user: `chown mxisd /etc/mxis`
- Copy `<repo root>/build/libs/mxisd` to `/usr/bin/mxisd`: `cp ./build/libs/mxisd /usr/bin/mxisd`
- Make it executable: `chmod a+x /usr/bin/mxisd`
- Copy (or create a new) `./application.yaml` to `/etc/mxis/mxisd.yaml`
- Configure `/etc/mxis/mxisd.yaml` with production value, `key.path` being the most important - `/etc/mxis/mxisd-signing.key` is recommended
- Copy `<repo root>/main/systemd/mxisd.service` to `/etc/systemd/system/` and edit if needed
- Enable service: `systemctl enable mxisd`
- Start service: `systemctl start mxisd`

# TODO
- Deb package
- Docker container
