# Supabase Postgres Overview

## Introduction

Supabase Postgres is a specialized version of PostgreSQL, designed to offer a batteries-included PostgreSQL distribution with a curated set of useful extensions pre-installed. It is tailored for modern applications, providing enhanced functionality and ease of use.

## Key Features and Differences

### Curated Extensions

Supabase Postgres includes a wide range of extensions that enhance PostgreSQL's functionality. These include:

- **Full-text search and indexing**: Tools like `pgroonga` and `rum`.
- **Geospatial data processing**: `PostGIS` and `pgrouting`.
- **Time-series data management**: `TimescaleDB`.
- **JSON validation and GraphQL support**: `pg_graphql` and `pg_jsonschema`.
- **Cryptography and security**: `pgsodium` and `pgaudit`.
- **Message queuing**: `pgmq`.
- **And many more**.

### Multi-version Support

The project supports multiple PostgreSQL versions simultaneously:

- **PostgreSQL 15**: Stable, battle-tested version.
- **PostgreSQL 17**: Latest features and improvements.
- **OrioleDB-17**: Experimental storage engine for PostgreSQL 17.

### Build System (Nix)

The project uses Nix as its build system, which provides:

- **Reproducible Builds**: Ensures that the same input always produces the same output.
- **Declarative Configuration**: Allows defining what you want, not how to build it.
- **Dependency Management**: Automatically handles complex dependency trees.
- **Cross-platform Support**: Enables building for Linux, macOS, and other platforms.

### Infrastructure as Code (Ansible)

The repository includes Ansible playbooks and configurations for server setup and deployment, ensuring consistent and automated infrastructure management.

### Docker Support

Dockerfiles are provided for building and running PostgreSQL with extensions in Docker containers, making it easy to deploy and manage in containerized environments.

### Additional Goodies

Supabase Postgres includes additional tools and services that enhance the overall functionality:

- **PgBouncer**: Connection pooling for PostgreSQL.
- **PostgREST**: Instantly transforms your database into a RESTful API.
- **WAL-G**: A tool for physical database backup and recovery.

### Philosophy

Supabase Postgres follows these core principles:

- **Unmodified PostgreSQL**: Uses standard PostgreSQL without modifications.
- **Curated Extensions**: Includes well-maintained, production-tested extensions.
- **Multi-version Support**: Supports multiple PostgreSQL versions.
- **Ready for Production**: Configured with sensible defaults for replication, security, and performance.
- **Open Source**: Everything is open source and can be self-hosted.

## Directory Structure

- **nix/**: Core build system directory containing all Nix expressions for building PostgreSQL and extensions.
- **ansible/**: Infrastructure as Code for server configuration and deployment.
- **migrations/**: Database migration management and upgrade tools.
- **docker/**: Container definitions and Docker-related files.
- **tests/**: Integration and system tests.
- **scripts/**: Utility scripts for development and deployment.
- **docs/**: Additional documentation, images, and resources.
- **ebssurrogate/**: AWS EBS surrogate building for AMI creation.
- **http/**: HTTP-related configurations and files.
- **rfcs/**: Request for Comments - design documents and proposals.
- **db/**: Database-related utilities and configurations.
- **.github/**: GitHub-specific configurations (Actions, templates, etc.).
- **Root Config Files**: Configuration files like `.gitignore`, `ansible.cfg`, and others.

## Installation

The repository provides detailed installation instructions in the [repo wiki](https://github.com/supabase/postgres/wiki), including Docker and AWS EC2 deployment options.

## Motivation

The primary goal is to make it fast and simple to get started with PostgreSQL, showcasing some of its most exciting features. It is the same build used at [Supabase](https://supabase.io), ensuring consistency and reliability.

## License

The repository is licensed under the [PostgreSQL License](https://opensource.org/licenses/postgresql), with a focus on ensuring that all bundled plugins comply with their respective licenses.

## Sponsors

Supabase is supported by various sponsors, contributing to the development and maintenance of open-source products.

## Conclusion

Supabase Postgres is a powerful and flexible PostgreSQL distribution tailored for modern applications, offering a wide range of extensions and tools to enhance its functionality and ease of use.