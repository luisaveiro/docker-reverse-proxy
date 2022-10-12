# Changelog
All notable changes to `luisaveiro/docker-reverse-proxy` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v0.2.0] 2022-10-12
### Added
- TLS configuration in `Caddyfile` example file to enable Caddy HTTPS support.
- Developer-friendly comments for environment variables in the DotEnv example file.
- "Trusting Caddy certificate authority" steps in Readme.

### Changed
- Renamed Docker Compose file to follow compose specifications.
- Readme to adopt Compose V2 specifications.

## [v0.1.2] 2022-05-03
### Fixed
- Connection refuse when triggering the `caddy reload` command.

## [v0.1.1] 2021-12-13
### Fixed
- Docker Compose not creating volumes.

## [v0.1.0] - 2021-12-01
### Added
- Docker Compose YAML file.
- Caddyfile example file.
- DotEnv example file.
