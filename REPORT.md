**Course:** Midterm Project  
**Team:** Muhammad Umar Rusjanto - 5025231176 
**Date:** October 22, 2025

---

## 1. Abstract
Briefly describe the project purpose (2–3 sentences).

## 2. Requirements Mapping
List course requirements and how project satisfies each (auth, ≥8 features, FK in DB, deploy, GitHub).

## 3. System Design
### 3.1 Conceptual (ER diagram)
(Insert ER diagram image here — file: `images/er_diagram.png`)

### 3.2 Physical schema
List tables and columns (show migrations excerpt). Mention foreign keys:
- `events.category_id` → `categories.id`
- `events.created_by` → `users.id`
- `tickets.event_id` → `events.id`
- `tickets.user_id` → `users.id`
- `profiles.user_id` → `users.id`
- `comments.user_id` & `comments.event_id`

## 4. Implemented Features
1. User registration / login / logout (Auth).  
2. User profile (CRUD).  
3. Event CRUD (create/read/update/delete).  
4. Event listing, search, pagination.  
5. Ticket booking (create/cancel).  
6. Event categories (CRUD).  
7. Comments & ratings on events.  
8. Admin dashboard (stats).  
(If you implemented more, list them.)

## 5. Screenshots & Explanation
For each feature below: include a screenshot and 2–4 lines describing it.

### 5.1 Login page
_Image: images/login.png_  
Explanation: login using registered email.

### 5.2 Event listing
_Image: images/events_index.png_  
Explanation: search bar, pagination, shows title, date, location.

(Repeat for each feature: registration, profile edit, event create, event detail, ticket confirmation, admin dashboard, GitHub repo page.)

## 6. Work Division
- Name A — 55%  
  - Tasks: DB design, Event CRUD, migrations, views, deployment.

- Name B — 45%  
  - Tasks: Auth & profile, Ticketing, Comments, report preparation, tests.

## 7. Analysis & Evaluation
- What worked well.
- Known issues (e.g., payment not integrated, basic UI).
- Security notes (do not commit `.env`, used policies).
- Future improvements (payment gateway, notifications, search improvements).

## 8. Conclusion
Short summary and lessons learned.

## Appendix
- Commands used
- Migrations list
- Important code excerpts
