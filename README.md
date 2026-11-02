# PHP (Moodle) service for Kubernetes on Wodby

Build and run Moodle LMS on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for PHP (Moodle).

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a boilerplate

Use one of the boilerplates exposed by this service to start with compatible
build configuration and Wodby CI:

- [Vanilla Moodle](https://github.com/moodle/moodle)

## Wodby stacks using this service

- [Moodle application stack](https://github.com/wodby/stack-moodle)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `moodle` |
| Type | Application service |
| Inherits from | [`php`](https://github.com/wodby/service-php) with version constraint `^3.0.0` |
| Versions | `8.4` by default, `8.3` |
| Service links | Moodle data storage, required; Redis |
| Volumes | `moodledata`, shared |
| Derivatives | `moodle-cron` |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; boilerplates: Vanilla Moodle |
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
