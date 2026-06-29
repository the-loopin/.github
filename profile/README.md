# Loopin: Local Events + Small Group Matching Platform
## Expanded MVP Planning Document

### Core Idea
Users discover local events, create small groups for those events, approve join requests, chat with accepted members, and earn event-specific badges after the event date.

| Field | Decision |
| :--- | :--- |
| **Project type** | University final year / semester project with startup potential |
| **Current target** | Web application |
| **Backend requirement** | Java / Spring Boot |
| **MVP duration** | 1 month |
| **Main user flow** | Event search -> Interested -> Create group -> Join request -> Accept/reject -> Group chat -> Event badge |
| **AI in MVP** | Hybrid content moderation: manual rules + AI-based filtering for risky text |

---

## Table of Contents
* [1. Product vision](#1-product-vision)
* [2. Problem and solution](#2-problem-and-solution)
* [3. MVP user flow](#3-mvp-user-flow)
* [4. Feature scope: must-have, should-have, later](#4-feature-scope)
* [5. Group creation and request system](#5-group-creation-and-request-system)
* [6. Auto archive and event badges](#6-auto-archive-and-event-badges)
* [7. AI and manual content moderation](#7-ai-and-manual-content-moderation)
* [8. Java/Spring Boot technical architecture](#8-javaspring-boot-technical-architecture)
* [9. Database/entity plan](#9-databaseentity-plan)
* [10. API endpoint plan](#10-api-endpoint-plan)
* [11. Frontend pages](#11-frontend-pages)
* [12. Security and safety rules](#12-security-and-safety-rules)
* [13. 1-month development roadmap](#13-1-month-development-roadmap)
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

* Users can search and filter events.
* Users can click “I am interested” on an event.
* Users can create a group with size 2, 3, 4, or 4+.
* The group creator becomes the group admin.
* Other users send requests to join the group.
* The group admin accepts or rejects requests.
* Accepted members can use the group chat.
* When the event date passes, the group is archived and members receive an event-specific badge.

---

## 2. Problem and Solution

| Problem | Loopin solution |
| :--- | :--- |
| People find events but do not want to go alone. | They can create or join a small group for that event. |
| Random public groups can become messy. | Joining requires a request and approval from the group admin. |
| People want to know who else is going. | Each event has related interested groups and members. |
| Social platforms can have inappropriate names/messages. | Manual filter + AI moderation checks risky user-generated content. |
| After events, user activity disappears. | Badges create history, reputation, and motivation. |

---

## 3. MVP User Flow
1. User registers or logs in.
2. User completes a simple profile: name, city, bio, interests.
3. User creates event or announcement that looks for someone.
4. User opens the Events page and searches/filters events or group to join as a member.
5. User opens an event detail page.
6. User clicks “I am interested”.
7. A group creation page/modal opens.
8. User selects group size: 2, 3, 4, or 4+.
9. User writes an optional group note and creates the group.
10. Other users see the group under the event and send join requests.
11. Request message is optional. A user can send it empty or explain why they want to join.
12. Group admin reviews requests and accepts or rejects them.
13. Accepted users become group members and can use group chat.
14. When event date passes, a scheduled job archives the group and grants badges to accepted members.

> **MVP flow summary:** Event -> Interested -> Group size -> Group admin -> Join request -> Accept/reject -> Group chat -> Auto archive -> Event badge

---

## 4. Feature Scope

| Priority | Feature | Include in 1 month? | Notes |
| :--- | :--- | :--- | :--- |
| **Must-have** | Register/Login | Yes | JWT-based authentication. |
| **Must-have** | User profile | Yes | Simple fields only. |
| **Must-have** | Event list/search/filter | Yes | Core discovery feature. |
| **Must-have** | Event detail page | Yes | Includes interested groups. |
| **Must-have** | Create group from event | Yes | Group size: 2, 3, 4, 4+. |
| **Must-have** | Join request | Yes | Optional request message. |
| **Must-have** | Accept/reject request | Yes | Group admin controls access. |
| **Must-have** | Simple group chat | Yes | Can be polling first; WebSocket if time allows. |
| **Must-have** | Auto archive + badge | Yes | Use Spring `@Scheduled`. |
| **Must-have** | Basic moderation | Yes | Manual bad words + AI moderation service. |
| **Should-have**| Admin review page | Maybe | Useful for pending/rejected content. |
| **Should-have**| Notifications | Maybe | Can be simple in-app notifications. |
| **Later** | Map view | No | Nice but not needed for MVP. |
| **Later** | Payment | No | Avoid in 1-month MVP. |
| **Later** | AI recommendation/matching | No | Keep for startup roadmap. |
| **Later** | Mobile app | No | Build web MVP first. |

---

## 5. Group Creation and Request System

### 5.1 Group creation from event
When a user clicks “I am interested” on an event, the platform opens a group creation form. The goal is not just to mark interest, but to create a small social unit for that event.

| Field | Type | MVP rule |
| :--- | :--- | :--- |
| **EventId** | Hidden/selected | Group belongs to exactly one event. |
| **Group title** | Text | Required, moderated. |
| **Group size** | Enum | 2, 3, 4, or 4+. |
| **Max member count**| Number | 2/3/4; for 4+ use max 10 in MVP. |
| **Group note** | Text | Optional, moderated. |
| **Preference** | Enum/string | Open to everyone, same interests preferred, beginner friendly, etc. |

### 5.2 Group admin rules
* The user who creates the group automatically becomes `GroupAdmin`.
* Only `GroupAdmin` can accept or reject join requests.
* `GroupAdmin` is also a member of the group.
* `GroupAdmin` can cancel the group before event date if needed.
* If group reaches max member count, status becomes `Full` and new requests are disabled.

### 5.3 Join request rules
* **Request message:** Optional. User can leave it empty.
* **Request statuses:** Pending, Accepted, Rejected, Cancelled.
* **Duplicate requests:** Same user cannot send multiple pending requests to same group.
* **Access to chat:** Only accepted members can see and write in group chat.
* **Rejected user:** Cannot access chat. Can optionally send again if admin allows; for MVP keep it blocked.

---

## 6. Auto Archive and Event Badges
After the event date passes, groups should not stay active forever. The system should automatically archive completed event groups and assign event-specific badges to people who were accepted group members.

### Scheduled job logic
Every day or every hour, the backend checks events where `EndDateTime` is before now and Status is not `Completed`. It marks the event as `Completed`, archives related groups, and creates `UserBadge` records for accepted members.

| Badge | Who gets it? | Purpose |
| :--- | :--- | :--- |
| **Event Attendee** | Accepted members of completed event groups | Shows participation history. |
| **Group Creator** | User who created the group | Rewards initiative. |
| **First Event Joined**| User completing first event | Motivation for new users. |
| **Networking Explorer**| User attending networking/HR/startup events | Category-based badge. |

> **Important MVP simplification:** because real attendance verification is hard, badge should mean “accepted group member for this event”, not guaranteed physical attendance. Later, QR check-in can verify real attendance.

---

## 7. AI and Manual Content Moderation
AI moderation is a good idea for this project because social platforms need safety. But for a 1-month MVP, the moderation system should be simple, reliable, and fail-safe.

### Recommended MVP approach
Use hybrid moderation: first check a manual blocked-words list, then call an AI moderation service for risky user-generated text. If AI fails, do not crash the app; mark content as `PendingReview` or use manual rules only.

### 7.1 What should be moderated?
* **Username / Full name:** Yes (Prevents inappropriate visible identities).
* **Profile bio:** Maybe (Useful, but can be added after core flow).
* **Group title:** Yes (Highly visible under event).
* **Group note:** Yes (User-generated group description).
* **Request message:** Yes (Can contain harassment or spam).
* **Chat messages:** Yes (Social safety).
* **Event title/description:** If users create events (If only admin creates events, less urgent).

### 7.2 Moderation decisions
* **Approved:** Content is safe and visible.
* **PendingReview:** Content may be risky; admin should review.
* **Rejected:** Content is blocked and not visible.
* **Blocked:** Severe violation; user action may be limited.

### Moderation layers
* **Manual bad words list:** Fast, cheap, reliable for obvious inappropriate words.
* **AI moderation service:** Detects more complex toxicity, harassment, sexual content, hate, threats, etc.
* **Admin review:** Human final decision for `PendingReview` items.
* **ModerationLog table:** Stores what happened and why for debugging/demo.

---

## 8. Java / Spring Boot Technical Architecture

| Layer | Responsibility |
| :--- | :--- |
| **Controller** | Receives HTTP requests and returns responses. |
| **Service** | Business logic: create group, accept request, badge assignment, moderation. |
| **Repository** | Database access using Spring Data JPA. |
| **Security** | JWT auth, roles, authorization checks. |
| **Scheduler** | Auto archive events and assign badges. |
| **Moderation client**| Manual + AI moderation integration. |
| **DTO/Mapper** | Separate API models from database entities. |

### 8.1 Suggested stack
* **Backend:** Java 17 or 21, Spring Boot
* **Security:** Spring Security + JWT
* **Database:** PostgreSQL or MySQL
* **ORM:** Spring Data JPA / Hibernate
* **Validation:** Jakarta Bean Validation
* **Chat:** Simple polling first; WebSocket if time allows
* **Background jobs:** Spring `@Scheduled`
* **Frontend:** React or Next.js
* **Styling:** Tailwind CSS or Bootstrap
* **API docs:** Swagger/OpenAPI

---

## 9. Database / Entity Plan
* **User:** Authentication identity.
* **UserProfile:** Profile data: city, bio, interests, image.
* **Event:** Main event information.
* **EventGroup:** Small group created for a specific event.
* **GroupMember:** Users accepted into the group.
* **GroupJoinRequest:** Requests sent by users to join a group.
* **GroupMessage:** Messages inside accepted group chat.
* **UserBadge:** Event-specific badge given after completion.
* **ModerationLog:** Stores moderation checks and decisions.
* **Report:** Optional safety feature for reporting users/groups/messages.

### 9.1 Key fields
* **Event:** `Id`, `Title`, `Description`, `Category`, `Location`, `StartDateTime`, `EndDateTime`, `IsFree`, `Price`, `Status`
* **EventGroup:** `Id`, `EventId`, `CreatorUserId`, `Title`, `Note`, `SizeType`, `MaxMemberCount`, `Preference`, `Status`, `CreatedAt`
* **GroupMember:** `Id`, `GroupId`, `UserId`, `Role(Admin/Member)`, `JoinedAt`
* **GroupJoinRequest:** `Id`, `GroupId`, `RequesterUserId`, `Message`, `Status`, `CreatedAt`, `RespondedAt`, `RespondedByUserId`
* **GroupMessage:** `Id`, `GroupId`, `SenderUserId`, `Message`, `ModerationStatus`, `CreatedAt`
* **UserBadge:** `Id`, `UserId`, `EventId`, `BadgeName`, `Description`, `CreatedAt`
* **ModerationLog:** `Id`, `UserId`, `ContentType`, `ContentId`, `OriginalText`, `RiskScore`, `Decision`, `Reason`, `CreatedAt`

---

## 10. API Endpoint Plan

### Auth
* `POST /api/auth/register` - Create account
* `POST /api/auth/login` - Login and return JWT

### Profile
* `GET /api/profile/me` - Get current user profile
* `PUT /api/profile/me` - Update profile

### Events
* `GET /api/events` - List/search/filter events
* `GET /api/events/{id}` - Event detail
* `POST /api/events` - Create event (admin/organizer only in MVP)

### Groups
* `POST /api/events/{eventId}/groups` - Create group after Interested click
* `GET /api/events/{eventId}/groups` - List groups for event
* `GET /api/groups/{id}` - Group detail

### Requests
* `POST /api/groups/{groupId}/requests` - Send join request
* `GET /api/groups/{groupId}/requests` - Admin sees pending requests
* `POST /api/requests/{id}/accept` - Accept request
* `POST /api/requests/{id}/reject` - Reject request

### Chat
* `GET /api/groups/{groupId}/messages` - Get group messages
* `POST /api/groups/{groupId}/messages` - Send message

### Badges
* `GET /api/profile/me/badges` - Get my badges

### Moderation
* `GET /api/admin/moderation/pending` - Admin review pending content
* `POST /api/admin/moderation/{id}/approve` - Approve content
* `POST /api/admin/moderation/{id}/reject` - Reject content

---

## 11. Frontend Pages
* **Login/Register:** Authentication.
* **Home:** Recommended/free/latest events.
* **Events:** Search and filter events.
* **Event Detail:** Event info + related groups + Interested button.
* **Create Group:** Group size 2/3/4/4+ and group note.
* **Group Detail:** Members, requests if admin, chat if accepted.
* **My Groups:** Groups created or joined by current user.
* **My Requests:** Requests sent by current user.
* **My Badges:** Badges earned after events.
* **Admin Moderation:** Pending content review.

---

## 12. Security and Safety Rules
* Only authenticated users can create groups, send requests, or chat.
* Only group admin can accept or reject requests.
* Only accepted group members can read/write group messages.
* A user cannot accept their own request unless they are already the group admin by creation.
* A user cannot join a full or archived group.
* Content must pass moderation before becoming visible or should be marked `PendingReview`.
* Admin endpoints must require `ADMIN` role.
* Use DTOs and never expose password/hash/internal security fields.
* Validate all inputs: length, empty values, enum values, date rules.

---

## 13. 1-Month Development Roadmap

| Week | Goal | Tasks | Expected Result |
| :--- | :--- | :--- | :--- |
| **Week 1** | Foundation | Project setup, database, auth, profile, event CRUD, event search/filter. | User can login and browse events. |
| **Week 2** | Core group flow | Interested button, create group, group size logic, join request, accept/reject. | Main product idea works end-to-end. |
| **Week 3** | Chat + badge + AI mod | Group chat, manual filter, AI moderation service, scheduled archive, badge assignment. | Project becomes unique and demo-ready. |
| **Week 4** | Polish + presentation| Admin moderation page, seed data, bug fixing, UI cleanup, documentation, slides/demo script. | Ready for university presentation. |

> **Scope discipline:** Do not add map, payment, mobile app, advanced AI matching, or external event scraping in the first month. These are good startup features, but they can destroy the MVP schedule.

---

## 14. Demo Scenario and Presentation Plan

### 14.1 Demo data to prepare
* 10 users with different profiles and interests.
* 8 events: HR session, startup meetup, tech talk, language exchange, sport event, travel meetup, music event, design workshop.
* 6 event groups with different sizes.
* 10 join requests: some pending, some accepted, some rejected.
* Several chat messages.
* A few badges already assigned for completed events.
* A few `PendingReview` moderation items for admin demo.

### 14.2 Live demo flow
1. Login as normal user.
2. Search for an HR session event.
3. Open event detail page.
4. Click “I am interested”.
5. Create a group with size 3.
6. Login as another user and send join request with optional message.
7. Login back as group admin and accept the request.
8. Show that accepted user can access chat.
9. Show scheduled/event-completed badge logic using seeded completed event.
10. Show AI/manual moderation example and admin review page.

---

## 15. Startup Roadmap After MVP

| Phase | Features |
| :--- | :--- |
| **After university MVP** | Better UI, notifications, profile badges, report/block, organizer dashboard. |
| **Product v1** | Map view, calendar integration, verified organizers, richer group preferences. |
| **AI v1** | AI event recommendations, AI buddy suggestions, spam/fake profile detection. |
| **Growth** | University clubs, local organizers, startup communities, HR events, language exchange groups. |
| **Monetization** | Featured events, featured groups, organizer subscription, paid event commission later. |
| **Mobile** | React Native or native mobile app after web platform is stable. |

### IP / patent note
For a startup, protect the brand name with trademark if needed, protect code automatically with copyright, and keep unique algorithms/business logic as trade secret. Patent is usually only realistic if there is a genuinely new technical method, not just a general platform idea. Ask a legal professional before spending money on patent work.

---

## 16. Risks and Final Recommendation

| Risk | How to control it |
| :--- | :--- |
| **Scope becomes too big** | Keep only event + group + request + chat + badge + moderation in MVP. |
| **Chat takes too much time** | Start with simple polling; upgrade to WebSocket only if time remains. |
| **AI API fails or costs money** | Use service interface; fallback to manual filter and `PendingReview`. |
| **Frontend consumes too much time**| Use simple responsive UI and avoid complex animations. |
| **Badge logic becomes controversial**| Define badge as “accepted group member”, not verified attendance. |
| **Safety concerns** | Add moderation, report, and admin review at least in simple form. |

> **Final recommendation:** This idea is strong enough for a university final project and has startup potential. For the first month, build a clean MVP around one powerful flow: event discovery -> small group creation -> request approval -> group chat -> event badge. Add AI only as moderation, not as complex matching yet.

---

## Appendix A: Suggested Enums
* `GroupSizeType`: TWO, THREE, FOUR, FOUR_PLUS
* `GroupStatus`: OPEN, FULL, ARCHIVED, CANCELLED
* `GroupRole`: ADMIN, MEMBER
* `RequestStatus`: PENDING, ACCEPTED, REJECTED, CANCELLED
* `EventStatus`: DRAFT, PUBLISHED, COMPLETED, CANCELLED
* `ModerationStatus`: APPROVED, PENDING_REVIEW, REJECTED, BLOCKED
* `BadgeType`: EVENT_ATTENDEE, GROUP_CREATOR, FIRST_EVENT_JOINED, NETWORKING_EXPLORER

---

## Appendix B: Acceptance Criteria
* **Create group:** Authenticated user can create group from an event and becomes admin.
* **Group size:** 2/3/4 groups stop accepting after max count; 4+ has max 10 in MVP.
* **Join request:** User can send optional message; request is Pending by default.
* **Accept request:** Only group admin can accept; accepted user becomes member.
* **Reject request:** Only group admin can reject; rejected user cannot access chat.
* **Chat:** Only accepted members can read and send messages.
* **Auto archive:** When event `EndDateTime` passes, related groups become Archived.
* **Badge:** Accepted members receive event badge after event completion.
* **Moderation:** Bad content is blocked or marked `PendingReview` before becoming visible.
