# Moodle service for Kubernetes on Wodby

Run Moodle LMS on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Moodle.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Moodle application stack](https://github.com/wodby/stack-moodle)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `moodle` |
| Type | Application service |
| Image | [wodby/moodle](https://github.com/wodby/moodle), prebuilt; no application build |
| Versions | `5.3` by default |
| Service links | Database, required; Moodle data storage, required; Mail Transfer Agent; Redis |
| Settings | Admin username, Admin password (required), Admin email |
| Volumes | `moodledata`, shared |
| Derivatives | `moodle-cron` |
| Operations | 2 actions |

## Use this service

Use this service through [Moodle application stack](https://github.com/wodby/stack-moodle), or reference `moodle` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
