Product Requirements Document: Hostel Fault and Repair Tracker
Course: BU-209 Backend Engineering
Date: November 23, 2026
File: PRD.md

1. The Problem
Students living in campus hostels have a hard time when things break in their rooms or other areas, like power problems, water leaks, broken fans, or bad drainage. Right now, people report problems by talking to hall staff, sending chat messages, or writing in a paper book. Because there is no proper tracking system, complaints get forgotten, people keep asking for updates, and multiple students report the same broken thing in the same room or corridor. If nobody fixes these issues early, small problems turn into bigger ones and students stay uncomfortable.

2. Who Uses It
- Student: Sends a report when something breaks in a room or shared space, checks if anything is happening with the report, and confirms when it is fixed.
- Hall Coordinator: Checks new reports for the hostel, spots repeat reports for the same place and problem, picks how urgent it is, and gives the job to a repair man.
- Repair Worker: Sees the jobs given to them, marks when they start and finish the work, and writes a short note on what they fixed.

3. How People Cope Right Now and Why It Fails
How people handle it now:
- Telling hall workers face-to-face or dropping a message in a group chat.
- Writing complaints in a paper notebook or a basic list.
- Going back to ask again when nothing seems to happen.

Why this does not work well:
- No single place to check: Details are spread across chats, paper books, and people's heads, so nobody has a clear view of what is going on.
- Hard to trace delays: Without a record of who changed what and when, it is hard to see where a complaint got stuck.
- Too many repeat reports: If four roommates or corridor mates report the same fault, it creates multiple reports of the same thing and hides other issues.
- Retry and double-update issues: If someone sends a report twice because of bad network, or two staff members update the same item at the same time, things can clash or get duplicated.

4. Our Idea (Before Picking a Programming Language)
We want to build a main backend service that tracks every fault step by step, checks who is allowed to do each action, spots likely repeat reports, stops double creation when a request is retried, and keeps a full history log of every status change. We are focusing on how the system works and what data it holds first, before picking any programming language or database.

How the flow works:
1. A logged-in student sends a fault report with the place (hostel, floor, room or other areas), type of problem (electrical, water, cleaning, other), how serious it is (low, medium, high, urgent), a short title, a description, and a retry key.
2. The backend checks the inputs and permissions, looks for recent open reports with the same place and problem type, marks a possible match if found, saves the report as OPEN, and adds a first entry to the history log.
3. A hall coordinator checks OPEN reports, sets or keeps the urgency level, picks a repair man, and changes the status to ASSIGNED.
4. The repair man starts fixing it and changes the status to IN_PROGRESS.
5. When the fix is done, the repair man writes a short note about the repair and changes the status to RESOLVED.
6. The student who sent the report checks the fix:
   - If it works, the student marks it as CLOSED.
   - If the problem is still there, the student marks it as REOPENED and writes why, so a coordinator can assign it again (REOPENED to ASSIGNED).

5. First Version (MVP)
What we will build in version 1:
- Basic user login and role checks so students, coordinators, and repair men can only do what they are allowed to do.
- Endpoints to create a report, view one report with its history, list and filter reports (by status, hostel, problem type, urgency, or repair worker) with page limits, and edit allowed notes or details.
- Strict rules for moving between statuses so nobody can jump straight from OPEN to CLOSED or make changes they are not allowed to make.
- Checking recent active reports in the same place and problem type when a new report comes in, and saving a link to a possible match without deleting the new report.
- A history log table that records every status change, who did it, any note, and the time, without letting people wipe past history.
- Retry protection on report creation using a key so a bad connection retry does not make two identical reports.

What we will leave out of version 1:
- Sending SMS, email, or push alerts.
- Uploading photos or videos.
- Building a mobile app or fancy frontend screens.
- Automatic timers, charts/reports, or scheduled checkups.
- Connecting to school-wide management systems.

6. Backend Details
Main data tables / records:
- User: id, full name, email, role (student, coordinator, repair worker), assigned hostel if needed, active status, created and updated times.
- Hostel: id, name, short code.
- Place: id, hostel id, floor, room or area, whether it is an other area.
- FaultReport: id, tracking code (like FLT-001), student id who reported it, place id, problem type, urgency level, current status, assigned repair worker id, possible duplicate report id, title, description, fix note, version number, and times for created, updated, resolved, and closed.
- StatusLog: id, report id, old status, new status, user id who made the change, note, and time.
- RetryKey: key value, user id, request hash, saved status code, saved reply data, created time, and expiry time.

Status rules:
- New report -> OPEN (by student)
- OPEN -> ASSIGNED (by coordinator, must pick an active repair worker)
- ASSIGNED -> IN_PROGRESS (by assigned repair worker or coordinator)
- IN_PROGRESS -> RESOLVED (by assigned repair worker or coordinator, must include a fix note)
- RESOLVED -> CLOSED (by the student who reported it)
- RESOLVED -> REOPENED (by the student who reported it, must include a reason note)
- REOPENED -> ASSIGNED (by coordinator, must pick an active repair worker)

Any other status jump is blocked. Also:
- Students can only close or reopen their own reports.
- Coordinators handle reports for their own hostel.
- Repair workers only update jobs assigned to them.
- Duplicate check looks for active reports (OPEN, ASSIGNED, IN_PROGRESS, REOPENED) in the last 14 days with the same place and problem type, and links the earliest one as a possible duplicate.

Error responses:
- 400 / 422: Missing info, bad choice values, or missing fix note when marking RESOLVED.
- 401 / 403: Not logged in, or trying to do something outside your role or hostel.
- 404: Report, place, or repair worker does not exist.
- 409: Bad status change, old version number clash during simultaneous edits, or reusing the same retry key with different data.

Other system needs:
- Use a version check when updating a report so two people editing at the exact same time do not overwrite each other silently.
- Add database indexes on common search fields (hostel + status + date, repair worker + status, place + problem type + status) so searches stay fast.
- Validate all inputs on the server, use safe database queries, and do not leak internal error details.
- Log key actions with request id, user id, role, report id, and result.

7. How We Know It Works, Assumptions, and Risks
Checks for version 1:
- Tests pass for all allowed status moves, blocked status moves, and role limits.
- Retrying a report request with the same key does not create a second report, and stale version edits return a conflict error.
- Every report has a status log entry from the start, and every status change adds another log entry.
- List and filter calls stay fast in local testing.

Assumptions:
- Users, hostels, places, and repair worker accounts already exist in the system before testing.
- Times are stored in a consistent format.

Risks and how we handle them:
- Adding too many extra features too early can slow us down, so we stick strictly to the core flow in version 1.
- Two coordinators updating a report at once could clash, so we use version checks and transactions.
- Students submitting repeat reports for the same issue could cause confusion, so we link possible matches automatically.