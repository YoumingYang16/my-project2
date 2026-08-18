# HoldLight: Climbing Assistance Demo System

HoldLight is a full-stack web application developed for CPT208. It explores how browser-based computer vision, voice interaction, and guided training workflows can support climbing practice.

## Main Capabilities

- User registration and authentication with JSON Web Tokens.
- Video upload and browser-based pose analysis.
- MediaPipe-powered pose detection and feedback features.
- Voice-command and audio-feedback interfaces using browser APIs.
- Training, profile, practice-record, and community-oriented pages.
- REST API and MongoDB-backed data layer.
- Docker, Nginx, Render, and GitHub Actions configuration for deployment workflows.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Front end | React 18, TypeScript, Vite, React Router, Axios, Tailwind/PostCSS tooling |
| Browser features | MediaDevices, Web Speech API, Web Audio API, MediaPipe Pose |
| Back end | Node.js, Express, Mongoose, JWT, bcrypt |
| Data | MongoDB |
| Computer vision | Python utilities and machine-learning dependencies included with the project |
| Deployment | Docker, Nginx, Render Blueprint, GitHub Actions |

## Repository Structure

```text
.
|-- climb-app-frontend/          # React + TypeScript client application
|-- climb-app-backend/           # Express API and MongoDB integration
|-- docker-compose.yml           # Local multi-service configuration
|-- render.yaml                  # Render deployment blueprint
|-- .github/workflows/           # Continuous-integration workflows
`-- package.json                 # Convenience scripts for both applications
```

## Prerequisites

- Node.js 18 or later (recommended)
- npm
- MongoDB instance for the back end
- Git LFS, if model or large binary assets need to be retrieved

Install Git LFS and download LFS-tracked assets when required:

```bash
git lfs install
git lfs pull
```

## Local Setup

Clone this repository and install both JavaScript applications:

```bash
git clone https://github.com/YoumingYang16/my-project2.git
cd my-project2
npm run install:all
```

Create the required environment files from the provided examples, then set the relevant MongoDB and authentication values. Do not commit private environment files or credentials.

Start the back end in one terminal:

```bash
npm run dev:backend
```

Start the front end in another terminal:

```bash
npm run dev:frontend
```

To create a production front-end build:

```bash
npm run build:frontend
```

## Configuration Notes

- The back-end service must be able to reach the configured MongoDB instance.
- Browser camera, microphone, speech, and audio features may require HTTPS or `localhost` permissions.
- Large model files are managed with Git LFS according to `.gitattributes`.
- Deployment-related files are included for reference and may need environment-specific values before production use.

## Academic Use

This repository is a coursework demonstration system. It is not a medical, safety-critical, or professional climbing-coaching product.
