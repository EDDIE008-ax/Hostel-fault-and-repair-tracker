Product Requirements Document: Hostel Fault and Repair Tracker
Course: BU-209 Backend Engineering  
Date: November 23, 2026
---
1. Product Overview
The Hostel Fault and Repair Tracker is a main backend service that helps campus hostels track broken things from start to finish. Instead of losing complaints in group chats or paper books, the system logs every fault, checks who is allowed to update it, spots repeat reports for the same issue, stops duplicate entries when the network is bad, and keeps a permanent history of every change. Right now, we are focusing on how the system works and what data it saves, before picking a programming language or database.
---
2. Problem Statement
Students living in campus hostels have a hard time when things break in their rooms or shared spaces, such as power cuts, water leaks, broken fans, or bad drainage.
Right now, people report problems by talking to hall staff face-to-face, dropping messages in group chats, or writing in a paper notebook. Because there is no single place to track these issues:
Complaints get forgotten, and students keep going back to ask for updates.
There is no record of who changed what and when, making it hard to see where delays happen.
Multiple students in the same room or corridor report the exact same broken item, creating clutter and hiding other urgent issues.
Bad network retries or two staff members editing at the same time can create double records or overwrite updates.
If small problems are not fixed early, they turn into bigger, more expensive problems and leave students uncomfortable.
---
3. Business Objectives
Stop lost complaints: Give hostel staff one clear place to see and track every broken item until it is fixed.
Fix things faster: Help coordinators assign jobs quickly and spot where repairs are getting stuck.
Cut down on repeat work: Automatically flag when multiple students report the same issue in the same spot so repair workers do not waste time.
Keep clear records: Save a permanent history of every update so everyone is accountable for their part of the job.
---
4. Target Users
Student: Lives in the hostel, reports broken items in their room or shared spaces, checks the progress of their report, and confirms when the fix actually works.
Hall Coordinator: Manages reports for their assigned hostel, spots repeat reports, sets how urgent a problem is, and assigns the job to a repair worker.
Repair Worker: Views the repair jobs given to them, marks when they start and finish the work, and writes a short note explaining what they fixed.
---
5. Stakeholders
Students: Want a fast, easy way to report broken things and know when they will be fixed.
Hall Coordinators: Need a clean, organized list of faults so they can manage hostel upkeep without getting flooded by duplicate messages.
Repair Workers: Need clear job details (exact room, problem type, and urgency) so they can bring the right tools and fix the issue.
School Hostel Management: Wants safer, better-maintained hostels, fewer big repair bills, and proof that student complaints are being handled.
Backend Engineering Team (BU-209): Responsible for building, testing, and keeping the backend service reliable and secure.
---
6. User Stories
As a Student, I want to send a fault report with my room details and problem type so that hostel staff know something is broken.
As a Student, I want to check the status of my report so I do not have to keep walking to the porter's lodge to ask for updates.
As a Student, I want to close my report when the fix works—or reopen it with a note if it is still broken—so the issue is truly solved.
As a Hall Coordinator, I want to filter open reports by urgency and problem type so I can handle emergencies first.
As a Hall Coordinator, I want the system to flag possible repeat reports so I can link them instead of sending two workers to the same room.
As a Hall Coordinator, I want to assign a report to an active repair worker in my hostel so the work gets started.
As a Repair Worker, I want to see all jobs assigned to me and update them when I start and finish so everyone knows my progress.
As a Repair Worker, I want to add a short note when I finish a job so the coordinator and student know what was done.
---
7. Functional Requirements
Core Data Records
User: ID, full name, email, role (student, coordinator, repair worker), assigned hostel (if needed), active status, and created/updated times.
Hostel: ID, name, and short code.
Place: ID, hostel ID, floor, room or area name, and whether it is a shared space.
FaultReport: ID, tracking code (like `FLT-001`), student ID, place ID, problem type (electrical, water, cleaning, other), urgency level (low, medium, high, urgent), current status, assigned repair worker ID, possible repeat report ID, title, description, fix note, version number, and times for created, updated, resolved, and closed.
StatusLog: ID, report ID, old status, new status, user ID who made the change, note, and time.
RetryKey: Key value, user ID, request hash, saved status code, saved reply data, created time, and expiry time.
Step-by-Step Workflow & Status Rules
The system must enforce strict rules so nobody can skip steps or make changes outside their role:
New Report $\rightarrow$ `OPEN` (Student): Student sends a report with place, problem type, urgency, title, description, and a retry key.
`OPEN` $\rightarrow$ `ASSIGNED` (Coordinator): Coordinator for that hostel reviews the report, sets or keeps the urgency, and picks an active repair worker.
`ASSIGNED` $\rightarrow$ `IN_PROGRESS` (Assigned Repair Worker or Coordinator): Worker marks that they have started the repair.
`IN_PROGRESS` $\rightarrow$ `RESOLVED` (Assigned Repair Worker or Coordinator): Worker marks the job done and must include a short fix note.
`RESOLVED` $\rightarrow$ `CLOSED` (Original Student): Student checks the repair, sees that it works, and closes the report.
`RESOLVED` $\rightarrow$ `REOPENED` (Original Student): Student checks the repair, sees it is still broken, and reopens it with a note explaining why.
`REOPENED` $\rightarrow$ `ASSIGNED` (Coordinator): Coordinator assigns the reopened fault to an active repair worker again.
Any other status jump (like going straight from `OPEN` to `CLOSED`) must be blocked.
Smart Checks & Protections
Repeat Report Detection: When a new report arrives, the system checks for active reports (`OPEN`, `ASSIGNED`, `IN_PROGRESS`, `REOPENED`) from the last 14 days with the exact same place and problem type. If found, it saves the new report anyway but links the oldest match as a possible repeat.
Retry Protection: Report creation requires a retry key. If a student's bad network sends the exact same request twice, the system returns the saved reply instead of creating a second report.
Permanent History Log: Every new report and every status change adds a new entry to `StatusLog`. Nobody is allowed to edit or delete past log entries.
Search and Filters: Users can view a single report with its full history, or list reports filtered by status, hostel, problem type, urgency, or repair worker using page limits.
Clear Error Responses
400 / 422 (Bad Input): Missing fields, invalid choices, or trying to mark a report `RESOLVED` without a fix note.
401 / 403 (Not Allowed): User is not logged in, or is trying to touch a report outside their role, hostel, or ownership.
404 (Not Found): The report, place, or repair worker ID does not exist.
409 (Conflict): Trying to make a blocked status jump, editing an outdated version of a report at the same time as someone else, or reusing a retry key with different details.
---
8. Non-Functional Requirements
Safe Simultaneous Edits: Use a version number check when updating a report so if two coordinators edit the same report at the exact same second, one gets a conflict warning instead of silently overwriting the other's work.
Fast Searches: Add database indexes on common search combos (`hostel + status + date`, `repair worker + status`, and `place + problem type + status`) so filtering stays fast as reports grow.
Security & Input Safety: Check all inputs on the server, use safe database queries to prevent data attacks, and never show raw system crash details to users.
Audit Logging: Record key system actions with a request ID, user ID, role, report ID, and result so developers can trace bugs easily.
Consistent Time Format: Store all timestamps in one standard format (such as UTC) so logs and 14-day repeat checks are always accurate.
---
9. Scope and Priorities
Included in Version 1 (MVP)
Basic user login and role-based permission checks.
Endpoints to create, view, list, filter, and update fault reports.
Strict status step rules and permanent status history logs.
Automatic 14-day repeat report detection and linking.
Retry key protection for creating reports and version checks for updates.
Out of Scope for Version 1 (Future Updates)
Sending SMS, email, or phone push alerts.
Uploading photos or videos of broken items.
Building a mobile app or visual frontend screens.
Automatic reminder timers, summary charts, or scheduled checkups.
Connecting to other school-wide student management systems.
---
10. Acceptance Criteria
We will know Version 1 is ready when all of these checks pass:
Role Limits Work: A student cannot assign a worker, a repair worker cannot close a report, and a coordinator cannot edit reports from a different hostel.
Status Rules Hold: All 7 allowed status moves succeed, and any skipped or backward jump returns a `409` conflict error.
Notes Are Enforced: Marking a report `RESOLVED` without a fix note, or `REOPENED` without a reason note, is blocked with a `400/422` error.
Repeat Check Works: Sending two reports for the same room and problem type within 14 days saves both, and links the second report to the first.
Retry & Version Checks Work: Sending the same create request twice with the same retry key creates only one report. Two people editing the same report version at once triggers a `409` error for the second person.
History Is Complete: Every report starts with an `OPEN` entry in `StatusLog`, and every status change adds a new row that cannot be deleted.
Speed: Filtering and listing reports returns results quickly during local testing without slowdowns.
---
11. Assumptions and Open Questions
Assumptions
User accounts, hostels, floors, rooms, and active repair worker profiles already exist in the system before students start reporting faults.
Every API client (such as a future web or mobile app) will generate and send a unique retry key when creating a new fault report.
Open Questions
Login Method: Should Version 1 use simple token login (like JWT) or basic session headers while we focus on the core backend logic?
Reopen Limit: Should there be a limit on how many times a student can reopen the same report before a senior manager steps in?
Repeat Window: Is 14 days the right time window for spotting repeat reports, or should urgent issues (like water leaks) use a shorter window?
