# Loopin: Local Events + Small Group Matching Platform
## Compressed 2-Week MVP Planning Document

### Core Idea
Users discover local events, create small groups for those events, approve join requests, chat with accepted members, and earn event-specific badges after the event date.

| Field | Decision |
| :--- | :--- |
| **Project type** | University final year / semester project with startup potential |
| **Current target** | Web application |
| **Backend requirement** | Java / Spring Boot |
| **MVP duration** | **2 weeks (14 Days)** |
| **Main user flow** | Event search -> Interested -> Create group -> Join request -> Accept/reject -> Group chat -> Event badge |
| **AI in MVP** | Stripped for 2-week timeline. Manual keyword rules only to guarantee completion. |

---

## Table of Contents
* [1. Product vision](#1-product-vision)
* [2. Problem and solution](#2-problem-and-solution)
* [3. MVP user flow](#3-mvp-user-flow)
* [4. Feature scope: must-have, dropped](#4-feature-scope)
* [5. Group creation and request system](#5-group-creation-and-request-system)
* [6. Auto archive and event badges](#6-auto-archive-and-event-badges)
* [7. Content moderation (Simplified)](#7-content-moderation)
* [8. Java/Spring Boot technical architecture](#8-javaspring-boot-technical-architecture)
* [9. Database/entity plan](#9-databaseentity-plan)
* [10. API endpoint plan](#10-api-endpoint-plan)
* [11. Frontend pages](#11-frontend-pages)
* [12. Security and safety rules](#12-security-and-safety-rules)
* [13. 2-week development roadmap](#13-2-week-development-roadmap)
* [14. Demo scenario and presentation plan](#14-demo-scenario-and-presentation-plan)
* [15. Startup roadmap after MVP](#15-startup-roadmap-after-mvp)
* [16. Risks and final recommendation](#16-risks-and-final-recommendation)
* [Appendix A: Suggested Enums](#appendix-a-suggested-enums)
* [Appendix B: Acceptance Criteria](#appendix-b-acceptance-criteria)

---

## 1. Product Vision
Loopin is a web platform where users can find nearby events and create small groups to attend those events together. Instead of only showing events, the platform focuses on solving the “I do not want to go alone” problem.

### One-line description
Loopin helps people discover events and find like-minded people to attend those events together through small, request-based groups.

---

## 2. Problem and Solution

| Problem | Loopin solution |
| :--- | :--- |
| People find events but do not want to go alone. | They can create or join a small group for that event. |
| Random public groups can become messy. | Joining requires a request and approval from the group admin. |
| People want to know who else is going. | Each event has related interested groups and members. |
| After events, user activity disappears. | Badges create history and motivation. |

---

## 3. MVP User Flow
1. User registers or logs in.
2. User completes a simple profile: name, city, bio.
3. User opens the Events page and searches/filters events.
4. User opens an event detail page.
5. User clicks “I am interested”.
6. A group creation modal opens.
7. User selects group size: 2, 3, 4, or 4+.
8. User writes an optional group note and creates the group.
9. Other users see the group under the event and send join requests.
10. Group admin reviews requests and accepts or rejects them.
11. Accepted users become group members and can use group chat.
12. When event date passes, a scheduled job archives the group and grants badges.

---

## 4. Feature Scope

| Priority | Feature | Include in 2 weeks? | Notes |
| :--- | :--- | :--- | :--- |
| **Must-have** | Register/Login | **Yes** | Basic JWT-based authentication. |
| **Must-have** | User profile | **Yes** | Fields limited to Name, City, Bio. |
| **Must-have** | Event list/search | **Yes** | Core discovery feature. |
| **Must-have** | Event detail page | **Yes** | Includes interested groups list. |
| **Must-have** | Create group from event | **Yes** | Group size: 2, 3, 4, 4+. |
| **Must-have** | Join request | **Yes** | Optional message. |
| **Must-have** | Accept/reject request | **Yes** | Group admin controls access. |
| **Must-have** | Simple group chat | **Yes** | **HTTP Polling only.** Drop WebSockets. |
| **Must-have** | Auto archive + badge | **Yes** | Scheduled Spring `@Scheduled` task. |
| **Dropped** | AI Moderation | **No** | **Removed.** Replace with a basic local text regex/filter to save time. |
| **Dropped** | Admin review page | **No** | **Removed.** Admin panel takes too long to build. |
| **Dropped** | Notifications | **No** | **Removed.** Rely on direct page reloads. |
| **Dropped** | Maps & Payments | **No** | Not for MVP. |

---

## 5. Group Creation and Request System

### 5.1 Group creation from event
When a user clicks “I am interested” on an event, the platform opens a group creation form.

| Field | Type | MVP rule |
| :--- | :--- | :--- |
| **EventId** | Hidden | Group belongs to exactly one event. |
| **Group title** | Text | Required. |
| **Group size** | Enum | 2, 3, 4, or 4+. |
| **Max member count**| Number | Hardcoded limits based on Enum (2, 3, 4, or max 10 for 4+). |
| **Group note** | Text | Optional. |

### 5.2 Group admin rules
* The user who creates the group automatically becomes `GroupAdmin` and a member.
* Only `GroupAdmin` can accept or reject join requests.
* If group reaches max member count, status becomes `Full` and new requests are disabled.

### 5.3 Join request rules
* **Access to chat:** Only accepted members can see and write in group chat.
* **Rejected user:** Access blocked. Cannot send another request to the same group.

---

## 6. Auto Archive and Event Badges
Every hour, a background backend task checks events where `EndDateTime` has passed. It marks the event as `Completed`, archives related groups, and creates `UserBadge` records for accepted members.

| Badge | Who gets it? | Purpose |
| :--- | :--- | :--- |
| **Event Attendee** | Accepted members of completed event groups | Shows participation history. |
| **Group Creator** | User who created the group | Rewards initiative. |

---

## 7. Content Moderation (Simplified)
To hit the 2-week deadline, **AI integration is dropped**. 
* **MVP Approach:** A utility class checks user-generated text fields (Group Title, Group Note, Chat Messages) against a local string array of forbidden keywords (`BannedWordsList`).
* If a word matches, the backend rejects the request immediately with a `400 Bad Request` HTTP code.

---

## 8. Java / Spring Boot Technical Architecture

| Layer | Responsibility |
| :--- | :--- |
| **Controller** | Receives HTTP requests and returns JSON responses. |
| **Service** | Core logic: group management, requests, badge assignment, word filter. |
| **Repository** | Database access using Spring Data JPA. |
| **Security** | Spring Security + standard JWT filter. |
| **Scheduler** | Core background task execution via `@Scheduled`. |

### 8.1 Stack
* **Backend:** Java 17+, Spring Boot
* **Security:** Spring Security + JWT
* **Database:** H2 (In-Memory for zero setup) or PostgreSQL
* **ORM:** Spring Data JPA
* **Chat:** Short Polling via frontend (`setInterval` fetching data every 3 seconds)
* **Background jobs:** Spring `@Scheduled`
* **Frontend:** React with Tailwind CSS

---

## 9. Database / Entity Plan
* **User:** Identity and security credentials.
* **UserProfile:** Profile data (Name, City, Bio).
* **Event:** Main event information.
* **EventGroup:** Small group tracking an event.
* **GroupMember:** Connection table for users inside a group.
* **GroupJoinRequest:** Join attempts sent by users.
* **GroupMessage:** Chat history logs.
* **UserBadge:** Badge assignment tracking.

---

## 10. API Endpoint Plan

### Auth
* `POST /api/auth/register` - Create account
* `POST /api/auth/login` - Login and return JWT

### Profile & Badges
* `GET /api/profile/me` - Profile info
* `GET /api/profile/me/badges` - Earned badges

### Events
* `GET /api/events` - List/search events
* `GET /api/events/{id}` - Event detail

### Groups & Requests
* `POST /api/events/{eventId}/groups` - Create group
* `GET /api/events/{eventId}/groups` - List groups for event
* `POST /api/groups/{groupId}/requests` - Send join request
* `GET /api/groups/{groupId}/requests` - View pending requests (Admin only)
* `POST /api/requests/{id}/accept` - Accept request (Admin only)
* `POST /api/requests/{id}/reject` - Reject request (Admin only)

### Chat
* `GET /api/groups/{groupId}/messages` - Fetch messages
* `POST /api/groups/{groupId}/messages` - Send message

---

## 11. Frontend Pages
* **Login/Register:** Auth entry point.
* **Events Discovery:** List view with a text search box.
* **Event Detail:** Details + related groups container + "Interested" button trigger.
* **Group Detail Dashboard:** Members grid, incoming requests list (visible only to admin), and the polling chat box panel.
* **My Profile:** User bio and earned badge display grid.

---

## 12. Security and Safety Rules
* Only authenticated users with valid JWTs can touch group, request, or chat endpoints.
* Only the designated `GroupAdmin` can call accept/reject endpoints for their group.
* The messaging GET/POST endpoints must assert that the requesting user's ID exists in the `GroupMember` table for that group.

---

## 13. 2-Week Development Roadmap
