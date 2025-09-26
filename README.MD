# Nifi Local Cluster Examples

This repository provides Docker Compose examples for setting up local Apache Nifi clusters. It addresses common challenges in configuring Nifi with Docker, especially regarding certificate generation.

## Available Examples

- **1.28.1**: Contains a Docker Compose setup for Nifi version 1.28.1.
- **2.6.0**: Contains a Docker Compose setup for Nifi version 2.6.0. This version includes an `init-certs.sh` script for certificate generation and a `library_docs` folder with relevant documentation.

## Key Considerations for Nifi 2.6.0

Certificate generation for Nifi 2.6.0 using `nifi-toolkit` version 2.6.0 was problematic due to the absence of `tls-toolkit.sh`. To resolve this, `nifi-toolkit` version 1.28.1 was used for certificate generation for Nifi 2.6.0.

## Getting Started

### Prerequisites

- Docker
- Docker Compose

### Usage

To run a Nifi cluster:

1. Navigate to the desired Nifi version directory (e.g., `cd 2.6.0`).
2. Follow the instructions in the respective `README.md` file within that directory. Typically, this involves running `docker-compose up -d`.