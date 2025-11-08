# GroupInn

## System Architecture Documentation

This document outlines the technical architecture, core components, data model, and deployment strategy for **GroupInn** — a digital home for communities that makes group discovery, communication, and collaboration seamless.

__________________________________________________  ARCHITECTURE OVERVIEW  _________________________________________________

GroupInn uses a **modular client–server architecture** optimized for performance, scalability, and real-time communication.  
The system consists of three core layers:

1. **Frontend (Client Layer)** – The user interface where people create, discover, and interact with groups.  
2. **Backend (Application Layer)** – Handles business logic, authentication, and real-time messaging.  
3. **Database and Storage (Data Layer)** – Manages user, group, and message data, as well as uploaded media.

This separation of concerns ensures flexibility, clean maintainability, and reliable performance across different environments.

__________________________________________________  TECHNOLOGY STACK  _________________________________________________

**Frontend:** React (Next.js) with Tailwind CSS  
**Backend:** Node.js (Express) with Socket.IO for real-time chat  
**Database:** PostgreSQL using Prisma ORM  
**Authentication:** JWT-based authentication  
**File Storage:** Amazon S3 or compatible object storage  
**Deployment:** Dockerized services hosted on Railway, Render, or Vercel  

**Disclaimer:**  
The technologies listed above represent the current plan and may evolve as the product grows or as more suitable tools emerge.

__________________________________________________  SYSTEM COMPONENTS  _________________________________________________

### 1. Frontend (Client Layer)
- Built with **Next.js** for fast rendering and SEO optimization.  
- Uses **Tailwind CSS** for responsive and modular styling.  
- Connects to backend through REST APIs and WebSockets.  
- Key functions:
  - Group discovery and search  
  - Viewing trending and joined groups  
  - Posting updates, polls, and comments  
  - Real-time group chat and event access  

### 2. Backend (Application Layer)
- Developed with **Node.js (Express)** for efficient routing and middleware.  
- Uses **Socket.IO** for bi-directional real-time messaging.  
- Provides secure REST endpoints for users, groups, and messages.  
- Includes input validation, role-based access, and session handling.  

### 3. Data and Storage Layer
- **PostgreSQL** manages structured data for users, groups, messages, and events.  
- **Prisma ORM** streamlines schema migrations and query safety.  
- **S3-compatible storage** handles images and shared files securely.

__________________________________________________  DATA MODEL  _________________________________________________

The following ERD represents relationships between the main entities in GroupInn:

![GroupInn Data Model Diagram](docs/images/groupinn-data-model.jpeg)


erDiagram
  USER ||--o{ MEMBERSHIP : has
  GROUP ||--o{ MEMBERSHIP : contains
  GROUP ||--o{ CHANNEL : owns
  CHANNEL ||--o{ MESSAGE : has
  USER ||--o{ MESSAGE : authors
  GROUP ||--o{ EVENT : hosts

  USER {
    UUID id PK
    STRING email
    STRING password_hash
    STRING display_name
    DATETIME created_at
  }

  GROUP {
    UUID id PK
    STRING name
    STRING description
    DATETIME created_at
  }

  MEMBERSHIP {
    UUID id PK
    UUID user_id FK
    UUID group_id FK
    STRING role
    DATETIME created_at
  }

  CHANNEL {
    UUID id PK
    UUID group_id FK
    STRING name
    BOOLEAN is_default
    DATETIME created_at
  }

  MESSAGE {
    UUID id PK
    UUID channel_id FK
    UUID author_id FK
    TEXT body
    DATETIME created_at
    DATETIME deleted_at
  }

  EVENT {
    UUID id PK
    UUID group_id FK
    STRING title
    DATETIME event_time
    STRING description
  }


__________________________________________________  API ENDPOINTS  _________________________________________________

### Authentication
- `POST /auth/signup` — Register a new user  
- `POST /auth/login` — Authenticate and receive tokens  
- `POST /auth/refresh` — Renew access tokens  

### Groups
- `POST /groups` — Create new group  
- `GET /groups` — List available groups  
- `GET /groups/:id` — View group details  
- `POST /groups/:id/join` — Join group  
- `DELETE /groups/:id/leave` — Leave group  

### Channels and Messages
- `POST /groups/:groupId/channels` — Create new channel  
- `GET /groups/:groupId/channels` — Get channels in group  
- `GET /channels/:channelId/messages` — Retrieve chat history  
- `POST /channels/:channelId/messages` — Send message  
- `DELETE /messages/:id` — Delete message  

### Events
- `POST /groups/:groupId/events` — Create event  
- `GET /groups/:groupId/events` — List events  
- `DELETE /events/:id` — Delete event  

__________________________________________________  REAL-TIME COMMUNICATION  _________________________________________________

GroupInn leverages **Socket.IO** for seamless real-time interactions.

**Client → Server**
- `message:send` — Send a chat message  
- `presence:typing` — Show when a user is typing  
- `presence:online` — Notify connection state  

**Server → Client**
- `message:created` — Broadcast new message  
- `message:deleted` — Notify message removal  
- `presence:update` — Broadcast presence  
- `event:created` — Notify group members of new events  

__________________________________________________  SECURITY AND PRIVACY  _________________________________________________

- Enforces **HTTPS/TLS** for all communication.  
- Uses **JWT** for stateless authentication.  
- Passwords hashed with **bcrypt** or **Argon2**.  
- Role-based access for owners, moderators, and members.  
- File uploads validated by size and MIME type.  
- Implements input sanitization and request rate limiting.

__________________________________________________  DEPLOYMENT AND SCALING  _________________________________________________

- **Frontend:** Deployed via Vercel or Netlify.  
- **Backend:** Containerized using Docker and deployed on Railway or Render.  
- **Database:** Managed PostgreSQL instance with backups (Neon, Supabase, or Railway).  
- **Storage:** S3 or R2 for object management.  
- **Scaling:** Future-ready Socket.IO Redis adapter for chat scaling.  
- **Monitoring:** Logs managed with Winston and viewed via dashboards.

__________________________________________________  LOCAL DEVELOPMENT  _________________________________________________

### Prerequisites
- Node.js v20+  
- PostgreSQL  
- Docker (optional for container setup)

### Environment Variables
```
DATABASE_URL=postgresql://user:password@localhost:5432/groupinn
JWT_SECRET=your_secret_key
S3_BUCKET=your_bucket_name
S3_KEY=your_access_key
S3_SECRET=your_secret
S3_REGION=your_region
```

### Development Commands
```bash
npm install
npm run dev
```

__________________________________________________  SYSTEM DIAGRAM  _________________________________________________

```mermaid
flowchart LR
  U[User Browser] -->|HTTPS| FE[Next.js Frontend]
  FE -->|REST / WebSocket| BE[Express + Socket.IO API]
  BE -->|SQL| DB[(PostgreSQL Database)]
  BE -->|S3 API| S3[(Object Storage)]
  BE --> AUTH[JWT Auth Service]
  FE --> CDN[(Static Assets/CDN)]
```

__________________________________________________  AUTHOR AND ATTRIBUTION  _________________________________________________

**Author:** Henry Elue  
**Program:** HNG 123  
**Stage:** Stage 4A  
**E-mail:** henryelue@gmail.com  
**Slack ID:** Henry Elue  
**Figma Prototype:** [View GroupInn Prototype](https://www.figma.com/design/QjZJo3sloNRaevukAjgPo4/Untitled?node-id=0-1&t=AlK4hHu40oR2CJvb-1)

**Wireframe Preview:** Covers the following core screens:
- **Splash Screen:** Introduces users to GroupInn with branding and quick entry options.  
- **Login & Register:** Secure authentication flow linked to the backend API and JWT system.  
- **Group Discovery & Search:** Explore and join communities of shared interest.  
- **Group Feed:** View posts, polls, and updates shared within joined groups.  
- **Real-Time Chat:** Connect instantly using Socket.IO-powered group messaging.  
- **Event Listings:** Discover upcoming group events and details.

__________________________________________________  END OF DOCUMENT  _________________________________________________
