# Local Agent

A VSCode extension that runs local AI agents powered by Docker.

## Features

- Start and stop Docker containers from the VSCode sidebar
- Stream container logs in real time
- Manage container lifecycle without leaving the editor

## Requirements

- Docker Desktop (macOS/Windows) or Docker Engine (Linux) must be running
- VSCode 1.85 or later

## Usage

1. Open the **Local Agent** panel from the Activity Bar
2. Enter a Docker image name (default: `alpine:latest`)
3. Click **Start container** to pull and run the image
4. Click **Stop** to terminate the container

## Release Notes

### 0.0.1

Initial release of local-agent.
