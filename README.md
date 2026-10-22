SocialConnect — Full-Stack Social Media Platform
SocialConnect is a React/JavaScript social media platform backed by a Node.js/Express REST API and MongoDB. It includes JWT authentication and user profiles, persisted posts and media uploads, likes, comments, saved posts, following, personalized feed modes, direct messaging, call history, and browser-based voice/video calling with WebRTC signaling.
This is a development/learning project, not a production-ready social network. Use synthetic data only. Keep .env files, database credentials, JWT secrets, and other private configuration out of Git.

Run the full local stack
Requirements: Node.js 18+ (Node.js 20+ recommended), npm, and MongoDB running locally or a MongoDB deployment.
1. Copy backend/.env.example to backend/.env and replace JWT_SECRET with a long random secret. Do not commit .env.
2. Open a terminal in backend/, run npm install, then start the API with npm run dev.
3. The backend runs on http://localhost:5002 by default. Health check: http://localhost:5002/api/health.
4. Open a second terminal in frontend/, run npm install, then start Vite with npm run dev.
5. Open the frontend URL shown by Vite, normally http://localhost:5173.
The frontend uses VITE_API_URL for REST API requests and VITE_MEDIA_URL for uploaded profile/post media. The supplied .env.example files are configured for the default local backend port 5002.
Run services separately
Backend API
cd backend
npm install
npm run dev
For a production-style Node start without Nodemon:
cd backend
npm install
npm start
The backend reads configuration from backend/.env:
PORT=5002
MONGODB_URI=mongodb://127.0.0.1:27017/socialconnect
JWT_SECRET=replace_this_with_a_long_random_secret_before_running
Frontend
cd frontend
npm install
npm run dev
For a production frontend build:
cd frontend
npm run build
npm run preview
The frontend .env can contain:
VITE_API_URL=http://localhost:5002/api
VITE_MEDIA_URL=http://localhost:5002
When the backend is deployed to another domain, update these two values to the deployed API/media URL and rebuild the frontend.
Current implementation notes
- User registration and login are implemented through /api/auth/register and /api/auth/login.
- Authentication uses JSON Web Tokens (JWT). The frontend stores the returned token and user information in browser local storage for the current development implementation.
- Passwords are hashed with bcryptjs before being persisted.
- Protected routes use authentication middleware and require a valid Bearer token.
- User profiles support name, username, bio, profile image, followers, and following information.
- User search is available through /api/users/search and supports follow/unfollow actions.
- Posts support text, image, and video content. A post can contain text, media, or both.
- Uploaded media is handled with Multer and served through the /uploads static path.
- Media uploads are limited to 50 MB by the backend middleware.
- Supported media types include common JPEG/PNG/GIF/WEBP images and MP4/WEBM/MOV/OGG video formats.
- Users can like/unlike posts, add comments, save/unsave posts, and delete posts they own.
- The feed supports all, following, and saved modes.
- The Following feed includes posts from followed users and the current user's own posts.
- The Saved feed returns posts saved by the current user.
- Direct messaging supports conversation listing, individual message threads, and sending messages between users.
- Call functionality includes active calls, call history, starting a call, retrieving a call session, and exchanging WebRTC signaling data.
- WebRTC is used for browser-based audio/video peer communication. The backend call API acts as the signaling layer for the development implementation.
- The application currently uses REST APIs and polling for message retrieval rather than a dedicated WebSocket messaging channel.
- CORS is enabled in the development server. Production deployments should restrict allowed origins instead of using an unrestricted CORS policy.
- Uploaded files are stored locally under backend/uploads/. For production, use durable/cloud object storage rather than relying on the application filesystem.
- The project currently has no production-grade moderation, rate limiting, email verification, password recovery, automated test suite, or TURN-server configuration for WebRTC reliability.
API overview
Area	Endpoints / Purpose
Auth	POST /api/auth/register, POST /api/auth/login
Users	Search users, view profiles, get current user, update profile, follow/unfollow
Posts	Create/list posts, like/unlike, comment, save/unsave, delete own posts
Messages	List conversations, retrieve a user thread, send messages
Calls	Active calls, call history, start call, retrieve session, WebRTC signaling
Health	GET /api/health
Media	/uploads/* serves locally uploaded profile and post media


Authentication flow
Register / Login
       ↓
   JWT Token
       ↓
Browser localStorage
       ↓
Authorization: Bearer <token>
       ↓
Express Authentication Middleware
       ↓
Protected REST API
Social feed flow
Create Post
     ↓
MongoDB Post Document
     ↓
Feed API
     ↓
 ┌───────────────┬────────────────┬──────────────┐
 ▼               ▼                ▼
All           Following         Saved
Feed          Feed              Posts
WebRTC call flow
User A
  ↓
Start Call API
  ↓
Call Session in MongoDB
  ↓
User B accepts call
  ↓
WebRTC offer / answer / ICE signaling
  ↓
Browser peer connection
  ↓
Audio / Video communication
Project structure
SocialConnect/
├── frontend/                         React + JavaScript + Vite UI
│   ├── src/
│   │   ├── services/                API client
│   │   ├── App.jsx                  Main application and UI workflows
│   │   ├── main.jsx                 React entry point
│   │   └── styles.css               Application styling
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
│
├── backend/                          Node.js + Express REST API
│   ├── src/
│   │   ├── controllers/             Auth, users, posts, messages, calls
│   │   ├── middleware/              Authentication and media upload
│   │   ├── models/                  User, Post, Message, Call
│   │   ├── routes/                  REST API route definitions
│   │   └── server.js                Express and MongoDB startup
│   ├── uploads/                     Local development media storage
│   ├── .env.example
│   └── package.json
│
└── README.md
Database models
The backend uses Mongoose models for the main application entities:
- User — account, profile, followers/following, and saved-post information.
- Post — post content, media information, author, likes, and comments.
- Message — direct messages exchanged between users.
- Call — call sessions, participants, status/history, and signaling information.
Security and deployment notes
- Never commit backend/.env or frontend/.env to GitHub.
- Replace the development JWT secret before deployment.
- Use a managed MongoDB connection string and a strong database access policy in production.
- Restrict CORS to the deployed frontend origin.
- Use HTTPS for the deployed application.
- Use secure, production-grade authentication/session handling rather than relying only on browser local storage for long-lived credentials.
- Store uploaded media in cloud/object storage for production deployments.
- Configure a TURN server if WebRTC calls must work reliably across restrictive NAT/firewall environments.
- Add rate limiting and abuse protection before exposing authentication, messaging, uploads, and calling endpoints publicly.
- Add content moderation and file/content security controls before accepting untrusted public uploads.
Deployment
The project can be deployed as two services:
React/Vite Frontend
        ↓
Static Hosting / CDN
        ↓
Node.js + Express API
        ↓
MongoDB
        ↓
Persistent Media Storage
For WebRTC calls in a production environment, configure HTTPS and a suitable STUN/TURN infrastructure. The local development implementation should not be considered sufficient for reliable internet-wide calling without those services.
Future improvements
- Real-time messaging with WebSockets or Socket.IO
- Production WebRTC signaling and TURN server support
- Cloudinary, Amazon S3, or equivalent object storage for media
- Push/in-app notifications
- Stories and temporary media
- Group chats and group calls
- Post sharing and reposts
- Advanced search and discovery
- Personalized recommendation/feed ranking
- Content moderation and reporting
- Email verification and password recovery
- Rate limiting and abuse prevention
- Automated frontend/backend tests
- CI/CD pipeline and production monitoring
Project objective
The primary objective of SocialConnect is to build a practical full-stack social media application while learning how frontend interfaces, REST APIs, authentication, databases, file uploads, social relationships, messaging, and WebRTC communication can work together in a single system.

The main application flow is:
User
 ↓
React Frontend
 ↓
Axios REST Requests
 ↓
Express / Node.js API
 ↓
Authentication + Controllers
 ↓
Mongoose Models
 ↓
MongoDB
 ↓
Response to React
For communication features:
Users
 ↓
Messages / Calls API
 ↓
MongoDB
 ↓
WebRTC Signaling
 ↓
Browser Peer Connection
Learning outcomes
Through this project, I gained practical experience in:
- React component-based development
- REST API integration with Axios
- Node.js and Express backend development
- MongoDB and Mongoose data modeling
- JWT authentication and protected routes
- Password hashing with bcrypt
- CRUD operations
- File upload handling with Multer
- Social feed implementation
- Follow/unfollow relationships
- Likes, comments, and saved posts
- Direct messaging
- WebRTC-based voice/video calling concepts
- Environment-based application configuration
- Full-stack project organization and deployment preparation
