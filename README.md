# Project Overview 🚀

This project delves into the fundamentals of systems design, exploring essential architectural and design patterns within the context of a distributed application. My aim is to demonstrate a solid grasp of core concepts while experimenting with widely used technologies. Key areas of focus include:

- **Distributed Architecture**:  Building a system composed of interacting services that communicate over a network.

- **Back-end Technologies**:  Leveraging Python (Django and Flask) and Java (Spring) to construct robust server-side components.

- **Databases**:  Employing PostgreSQL for relational data and MongoDB for flexible log storage.

- **Security**:  Implementing authentication (JWTs) and secure communication practices (TLS, HMAC).

- **CI/CD**:  Automating the build, test, and deployment process using Jenkins.

- **Logging**:  Centralized log aggregation and analysis.

## Motivation

This endeavor serves as a learning playground for understanding how to design and implement scalable, maintainable systems. The emphasis is on practical application and solidifying my knowledge through hands-on development.

# Design Document

## Tech Stack

| Technology          | Elaboration                                                                                                                                                                                                                                                                                                                |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Python**          | The whole project will be built using Python and its technologies. The goal is to demonstrate a mastery of the language. Aim to utilize most aspects of the language. All datastructures, file handling, input/output, OOP, Modules, Annotations, Lambdas, Inheritance, Namespaces, Documentation, Decorators, Exceptions. |
| **Django**          | Web framework for all of the servers. Extremely on-demand.                                                                                                                                                                                                                                                                 |
| **Flask**           | As the focus of the project is to demonstrate understanding of common technologies, implementing the worker services in Flask allows for less overhead on services that will need to be spawned based on load.                                                                                                             |
| **logging library** | Python debugging logs to the terminal.                                                                                                                                                                                                                                                                                     |
| **JWT**             | Security                                                                                                                                                                                                                                                                                                                   |
| **Jenkins**         | CI/CD                                                                                                                                                                                                                                                                                                                      |
| **Java & Spring**   | Multiple Servers = Multiple opportunities for different technologies. Use Spring for the **Authentication Server** and the **Jobs Server**.                                                                                                                                                                                |
## All Services

| Service Name              | Manages                        | Functionalities                                                                                                                                                              | RESTful Endpoints                                                                                                                                                                       | Communicates With                                                                                                                                                                           | Logs Consideration (MongoDB)                                                                                                                                              |
| ------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **HTTPS Server**          | User requests                  | • User registration and authentication<br>• User Requests                                                                                                                    | `/register(POST)`<br>`/login(POST)`<br>`/add_job(POST)`<br>`/job/{id}/status(GET)`<br>`/jobs/list_all(GET)`<br>`/jobs/[id,...]/status(GET)`                                             | • **Authentication Server** → HTTP<br>• **Jobs Server**                                                                                                                                     | User activity. (Login (attempts, success, failure), Registration (success, failure), Request (couples user identifier with job requested))                                |
| **Jobs Server**           | Job Queue Database             | • Receives jobs from **HTTP Server**.<br>• Serves jobs to **Worker Services**<br>• Updates job status based on **Worker Service** requests<br>• Manages PostgreSQL database. | `/add_job(POST)`<br>`/get_job(GET)`<br>`/job/{id}/status`: `GET` → `job.status`, `POST` → `updates job.status`                                                                          | • **Authentication Server** to authenticate the worker service.<br>• **PostgreSQL DB** to store and remove jobs.                                                                            | Database activity: Job queued, type, time, de-queued by worker with id of, etc.<br>Worker activity: successful or unsuccessful worker authentication. Job status changes. |
| **Output Server**         | Job Output Database            | • Receives output from **Worker Service**s.<br>• Manages PostgreSQL database.                                                                                                | `/store_output(POST)`                                                                                                                                                                   | • **Authentication Server** to authenticate the worker service.<br>• **PostgreSQL DB** to store output.                                                                                     | Database activity: Output stored, type, time,by worker with id of, etc.<br>Worker activity: successful or unsuccessful worker authentication.                             |
| **Authentication Server** | User and Worker Authentication | • Manage `Users` PostgreSQL database.<br>• Manage `Workers` PostgreSQL database.<br>• Authenticate users and workers.<br>• Manage `active_workers` cache.                    | `/user/login(POST)`<br>`/user/register(POST)`<br>`/worker/generate_token`<br>`/worker/authenticate` (Differentiate between **Jobs** or **Output Server** for requests to this endpoint) | • **PostgreSQL DB** to store and check registered users and workers.<br>• **In-memory Cache** to validate active workers.                                                                   | Database activity.<br>User registration and authentication activity.<br>Worker registration and authentication activity.                                                  |
| **Worker Service**        | Job processing                 | • Authenticates itself with **Authentication Server**<br>• Requests jobs from **Jobs Server**<br>• Sends output results to **Output Server**                                 | N/A                                                                                                                                                                                     | •(HTTP) **Authentication Server** to generate its own `JWT` token.<br>• (WebSockets)**Jobs Server** to receive job for processing.<br>• (gRPC?)**Output Server** to store final job result. | Lifetime logging. Creation, deletion. Authorization. Job status. Errors. times, etc.                                                                                      |
| **Logger Service**        | Log storage                    | • Receives logs from every other service and server and stores them.                                                                                                         | `/store_log(POST)`                                                                                                                                                                      | • __MongoDB__ database for `Log` storage.                                                                                                                                                   |                                                                                                                                                                           |

---
## CI/CD Pipeline
Jenkins will be used to handle CI/CD. The structure will be as follows:
-  `main` will be protected. Only way to push code into `main` is through `PR`s.
- `integration` will serve as the intermediary branch bet ween `dev-*` branches and `main`. Code ready for production will be merged into `integration`.
- Once `integration` receives new code, Jenkins will pull and run all of the required tests.
- On completion, Jenkins will submit a `PR` outlining test results. 
 - Once the `PR` is ready for production, it is safe to merge into `main`
## API Endpoints Details
All requests will contain a `JSON` body. All per-endpoint request body will be within a root level `body` property. At the root level, there will be project-wide parameters.
### Authentication Parameter
All relevant properties necessary for authentication will be embedded in this root property.
```json
{
  "authentication":
  { // user body. for /login endpoint
    "email": "john_doe@test.com",
    "token": "sdfuoi21u3IOUS__2''?DFOMbnSDKLF`o7u987vb,.sf*j*m,.}",
  },
  // ------------
  { // worker body
    "service": "worker",
	"UUID": "sdoifusdiouf", // Generated by the worker itself. Used by the servers to match a token to a worker.
	"token": "sdoifu239847dsjflkujOITUOjdsflksdmnfSOKJTGB*(71293)",
  }
}
```

#TODO - Outline request and response `body` property for completed API implementations.
## Database Schemas
### `USER` Table

| id  | username | email | password |
| --- | -------- | ----- | -------- |

#### `ROLES` Table

| id  | name |
| --- | ---- |
#### `ROLES_PERMISSIONS` Table

| role_id | permission |
| ------- | ---------- |

#### `USER_ROLES` Table

| user_id | role_id |
| ------- | ------- |

### `QUEUED_JOB` Table

| id  | job_type | status | submitted_by | created_at | started_at | completed_at | parameters |
| --- | -------- | ------ | ------------ | ---------- | ---------- | ------------ | ---------- |

### `Log` Collection

| _id | timestamp | request_id | service_name | log_level | user_id | job_id | message | additional_metadata |
| --- | --------- | ---------- | ------------ | --------- | ------- | ------ | ------- | ------------------- |

## Security Considerations - JWT Handshake Schema and Shared Secrets
### Transfer Layer Security
All services will need a signed certificate.
### Server-to-Server Authentication
Servers shall have a `shared_secret` (for the project, simply a shared directory/file that is untracked by the `.git` repo. For a deployed project, we’d used a Secrets Manager) that they use for `HMAC` generation and authentication.
### Worker-to-Server Authentication - JWT Handhsake
- **Worker**:
	- Generate its own `public/private key pair`.
	- Generate its own `UUID`.
	- Sends `public_key` and `UUID` to `/worker/generate_token`.
	- **Further consideration** → Include `capabilities` in request.
- **Authentication Server**:
	- Generate its own `private_key` for `JWT` generation.
	- Uses `private_key`, `worker.public_key`, `worker.UUID`, and `worker.capabilities` to generate `JWT`.
	- Returns `Token`.

---

# Project Development Structure
## Phase 1 - VCS and CI/CD
### Stage 1 - Repository
- Create the base, empty monorepository for the project.
- Protect `main` to only receive code through `PR`s.
- Create `integration` branch.
- Set up the base directories for the laid out servers and services and submit it as a `PR` into `main`.
### Stage 2 - Jenkins Server
- Download and install all the necessary plugins.
- Set up a local Jenkins server to use for all development.
### Stage 3 - Jenkins Configuration
- Hook up Jenkins into `integration` branch on the GitHub repo.
- Configure all the Jenkins `stages` with mock data and tests.
- Use `README` updates to test base pipeline.
**Beyond this point →** All added functionality is to be accompanied by tests. Whether fully implemented, or as stand-ins always returning success to be implemented later on.

## Phase 2 - Containerization & MVP
### MVP Definition:
- **HTTPS Server** → `/add_job` & `/jobs/{id}/status`
- **Job Server** → `/add_job`, `/get_job`, `jobs/{id}/status(GET)`
- **Output Server** →`/store_output`
- **Worker Service** → Job fetch and output storage.
### Software Architecture Considerations
Dependency Injection and Separation of Concerns rule our world. Outside of per-service business logic, they will all need the following components:
- **Security**
- **Networking**
- **Logging**
Need to make considerations for designing wrappers around the libraries provided by `Django` and `Spring` to allow for stubbing during testing.
This will likely be achieved using Aspect Oriented Programming. Spring makes this seamless through configuration, while in Django I’ll define custom decorators (wherever needed), or middleware.
### Stage 1 - Docker Images
- Define all `Dockerfile`s for each of the services.
- Define `docker-compose.yml` in root directory
### Stage 2 - Jenkins Stages
- Define tests and configurations for Jenkins to build the `Dockerimage`s.
### Stage 3 - MVP
- Develop MVP functionality in the order laid out in “MVP Definition” section.
## Phase 3 - Full-Feature / All Endpoints
- Order of priority remains the same as in “MVP Definition”.
- `/register` and `/login` will rely on temporary stubs that always return success.
## Phase 4 - Security
- Building upon the “Software Architecture Considerations” section, all services should already have stand-in `Security` components. These will be configured to always succeed.
- Implement all `Authentication` and `Authorization` functionality.
- Establish `JWT` implementation.
- Implement `TLS`
## Phase 5 - Logging (Both Local and Server) & MongoDB
- Building upon the “Software Architecture Considerations” section, all services should already have stand-in `Logging` components. These will be configured to always succeed.
### Stage 1 - Local Logging
- Implement all `Logging` functionality at a local stage. 
- Separate by levels of detail (`INFO`, `ERROR`, `DEBUGGING`, etc).
### Stage 2 - MongoDB
- Set up the `MongoDB` server.
- Define a standard `logs` schema.
### Stage 3 - Logging Server
- Implement the **Logging Server** and all of its relevant functionality.

## Phase 6 - Error Handling and Recovery
During the development process, I will assume all failure points will succeed for the sake of expediting development. During development, I will note `FIXME - Recovery Required` on every portion of the code that will be revised in this phase.
- Consider the type of error handling for each of the error points. Retry? Poll? Error? Throw?
- Implement handling functionality
- Implement `[ERROR]` level logging.
## Phase 7 - Out of scope considerations
### Scaling
- Identify potentials for bottleneck and how to scale.
- Change communications protocol? HTTP → gRPC?
- Scale horizontally?
- Scale vertically?
- Load Balancing?
- Caching?
- Replication or Sharding?
- Dynamic Worker Deployment
- 
### Security
- Token Expiration & Renewal.
- Key Rotation
