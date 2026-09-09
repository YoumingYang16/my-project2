# HoldLight: Climbing Assistance Demo System

HoldLight is a full-stack coursework prototype that explores how browser interaction, computer vision, route planning, and explainable safety states can be combined into a climbing-assistance workflow.

The system connects a React and TypeScript client, a Node.js and MongoDB API, persistent Python vision workers, and browser-side MediaPipe pose tracking. A typical flow moves from wall capture and hold detection to route review, camera alignment, live pose tracking, and visual or spoken guidance.

> **Scope and safety:** HoldLight is an academic demonstration. It is not a medical device, safety-certified system, professional coaching product, or substitute for a trained climbing partner and appropriate protective equipment.

## System Capabilities

- Account registration and login with hashed passwords and expiring JWT authentication.
- Wall-image capture and upload from the browser.
- Hold, colour, and route inference through a Python vision service.
- Manual scan review before guidance begins.
- Browser-side pose tracking with subject locking and interference detection.
- Semantic route scoring based on hold confidence, size, position, role, and support relationships.
- Live visual overlays and browser speech cues.
- Safety-state explanations that can request a retake, companion check, or pause in guidance.
- Persistence for users, wall scans, climb sessions, guidance logs, and notifications.
- Container and cloud deployment configuration.

## Architecture

```text
Camera or uploaded wall image
           │
           ▼
React + TypeScript client
  ├── scan and review workflow
  ├── route recommendation
  ├── MediaPipe pose tracking
  ├── live overlay and speech cues
  └── safety-state presentation
           │ authenticated HTTP
           ▼
Express API + MongoDB
  ├── auth, users, scans, sessions, logs, notifications
  ├── rate limiting, CORS, health checks
  └── vision request orchestration
           │ JSON over persistent child processes
           ▼
Python vision runtime
  ├── hold and route inference
  ├── mask and contour processing
  ├── HSV, LAB and grayscale colour features
  ├── neutral-colour classifier
  └── planar calibration
```

The Node service keeps Python workers alive instead of starting a new interpreter for every request. Inference and calibration use separate worker pools, with request timeouts, JSON response handling, warm-up support, and health snapshots.

## Repository Structure

```text
.
├── .github/workflows/                 # automated workflow configuration
├── climb-app-frontend/
│   └── src/
│       ├── app/                       # application wiring and routing
│       ├── features/
│       │   ├── auth/
│       │   ├── climb-assist/
│       │   ├── dashboard/
│       │   ├── notifications/
│       │   ├── onboarding/
│       │   └── profile/
│       ├── pages/
│       ├── shared/
│       └── tests/
├── climb-app-backend/
│   ├── middleware/                    # JWT authentication and rate limiting
│   ├── models/                        # Mongoose persistence models
│   ├── routes/                        # REST and vision endpoints
│   ├── services/                      # Python runtime and provider orchestration
│   ├── vision_service/                # OpenCV, PyTorch and calibration code
│   ├── .env.example
│   └── server.js
└── deploy/                            # Docker, Nginx and hosting configuration
```

## Frontend Design

The climbing-assistance feature is split into components, hooks, pages, services, state, and utilities. Key workflow pages include wall scanning, route recommendation, live guidance, and session summary.

### Pose tracking

The client loads MediaPipe Pose from a local asset path with CDN fallbacks. Raw landmarks are converted into semantic joints and anchors, then evaluated for:

- visible limbs and landmark quality
- active-subject confidence
- subject lock and loss
- possible bystander interference
- live runtime profile and pose error state

`useLivePoseTracker` integrates these outputs into the React lifecycle rather than exposing raw landmarks directly to the interface.

### Route planning

The browser-side semantic planner scores and links detected holds using confidence, size, location, role, and support context. It includes memoised depth-first search and a greedy fallback. Start and finish semantics, foot support, and route quality are explicit parts of the decision process.

### Safety states

The safety-state service returns explainable states rather than a single Boolean:

- `ready`
- `companion`
- `retake`
- `pause-live-guidance`

It considers scan review, wall alignment, pose failures, pose quality, subject lock, and interference risk. Current thresholds include a pose-quality check below 48 and an interference-risk check at or above 62. These values are engineering heuristics in the prototype, not validated safety guarantees.

## Backend and API

The Express server exposes route groups for:

- authentication
- users
- climb scans
- climb sessions
- guidance logs
- notifications
- vision operations

Protected vision operations include:

- `POST /infer/holds`
- `POST /infer/routes`
- `POST /infer/full`
- `POST /calibrate/planar`

The exact mounted prefix is defined in `server.js`; inspect the server route registration when integrating an external client.

Authentication uses `bcrypt` with a generated salt and signed JWTs with an expiry. Vision inference and calibration have separate rate limiters. The server also reports MongoDB and vision-runtime status through health checks. In production, an empty allowed-origin configuration causes cross-origin browser requests to be denied rather than silently allowed.

## Vision Pipeline

The Python runtime uses OpenCV and PyTorch-based components, with optional Detectron2-backed inference where configured.

1. Decode and validate the input image.
2. Verify required dependencies and model assets.
3. Run hold or route inference.
4. Process masks, contours, regions, and confidence values.
5. Extract colour features using HSV, LAB, and grayscale information.
6. Apply the neutral-colour classifier for `other`, `white`, and `black` categories where applicable.
7. Return structured JSON to the Node worker manager.

Model weights are not assumed to be present in every checkout. The deployment configuration supports externally supplied model assets, including Git LFS or configured download URLs.

## Requirements

- Node.js and npm
- MongoDB, either local or hosted
- Python supported by the vision dependencies
- A browser with camera support for live capture and pose tracking
- Model weights required by the selected inference configuration

GPU acceleration is optional and depends on the installed PyTorch and vision stack. Verify compatibility among Python, PyTorch, torchvision, Detectron2, and the model weights before deployment.

## Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/YoumingYang16/my-project2.git
cd my-project2
```

If model assets are stored with Git LFS:

```bash
git lfs install
git lfs pull
```

### 2. Configure the backend

```bash
cd climb-app-backend
npm install
```

Copy `.env.example` to `.env` and replace every placeholder. At minimum, configure the MongoDB connection and a strong JWT secret. Do not commit `.env`, model-download credentials, or runtime logs.

```bash
npm start
```

Use the scripts declared in `climb-app-backend/package.json` if the command differs in the current branch.

### 3. Configure the frontend

In a second terminal:

```bash
cd climb-app-frontend
npm install
npm run dev
```

Set the frontend API base URL according to the environment configuration included in the project.

### 4. Check service health

Confirm that the Node server can reach MongoDB and that the vision health status reports the required runtime and model resources. A web page loading successfully does not by itself prove that vision inference is ready.

## Testing and Verification

Use the scripts defined in each `package.json` as the source of truth. A complete verification pass should cover:

- frontend unit or component tests
- frontend production build and TypeScript checks
- backend route and authentication tests where configured
- vision health and warm-up checks
- a hold-only inference request
- a route or full-pipeline inference request
- camera-denied and model-unavailable failure states
- subject-loss, wall-misalignment, and interference transitions

Do not report model accuracy or safety performance without a documented evaluation dataset, ground truth, metric definition, and reproducible result.

## Deployment

The repository includes Docker, Nginx, Render Blueprint, GitHub Actions, and Git LFS configuration. Deployment requires environment-specific origins, MongoDB access, JWT configuration, model assets, and sufficient memory for the selected vision runtime.

Before exposing the service publicly:

- use a strong, rotated JWT secret
- restrict CORS to intended origins
- keep inference and calibration rate limits enabled
- keep model credentials outside the repository
- confirm upload-size and timeout limits
- verify health endpoints without leaking sensitive configuration

## Failure Modes and Limitations

- Camera permissions, browser support, lighting, occlusion, and wall angle affect input quality.
- Missing or incompatible model assets prevent vision readiness.
- CDN fallback availability can affect MediaPipe initialisation if local assets are absent.
- Pose and interference thresholds are prototype heuristics.
- Route recommendations depend on detected holds and should be reviewed by the user.
- The system has not been clinically validated or certified for climbing safety.

## What the Project Demonstrates

- Full-stack decomposition across browser, API, database, and ML runtime boundaries
- Long-running worker management for reducing interpreter cold-start cost
- Combining server-side vision with real-time browser pose tracking
- Turning uncertain model outputs into reviewable and explainable application states
- Authentication, rate limiting, CORS, health checks, and deployment-aware configuration
- Explicit communication of model, environment, and safety limitations

## Academic Context

HoldLight was developed as an academic demonstration and portfolio project. It should be evaluated through its architecture, implementation, and documented constraints rather than treated as a production safety system.
