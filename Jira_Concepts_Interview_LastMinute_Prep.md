# Jira Concepts — Last-Minute Interview Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Example** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Covers **core Jira concepts** — issues, workflows, boards, Agile/Scrum/Kanban, JQL, sprints, and reporting — at the level asked of engineers, analysts, and anyone who uses Jira day-to-day. Built for a 1–3 hour final revision.

---

## Table of Contents

1. [What is Jira?](#1-what-is-jira)
2. [Projects & Project Types](#2-projects--project-types)
3. [Issues & Issue Types](#3-issues--issue-types)
4. [Issue Hierarchy: Epic → Story → Task → Sub-task](#4-issue-hierarchy)
5. [Workflows: Statuses & Transitions](#5-workflows-statuses--transitions)
6. [Agile in Jira: Scrum vs Kanban](#6-agile-in-jira-scrum-vs-kanban)
7. [Boards](#7-boards)
8. [Sprints & the Backlog](#8-sprints--the-backlog)
9. [Fields, Screens & Components](#9-fields-screens--components)
10. [JQL (Jira Query Language)](#10-jql-jira-query-language)
11. [Time Tracking & Estimation](#11-time-tracking--estimation)
12. [Reports & Dashboards](#12-reports--dashboards)
13. [Permissions, Roles & Notifications](#13-permissions-roles--notifications)
14. [Automation, Integrations & Add-ons](#14-automation-integrations--add-ons)
15. [Common Scenarios (with answers)](#15-common-scenarios)
16. [Tricky / Gotcha Questions](#16-tricky--gotcha-questions)
17. [Rapid-Fire One-Liners](#17-rapid-fire-one-liners)
18. [Last-Minute Checklist](#18-last-minute-checklist)

---

## 1. What is Jira?

### Theory

**Jira** is a **work management and issue-tracking tool** developed by **Atlassian**, widely used for **project management, bug/issue tracking, and Agile software development**. Teams use it to plan, track, and manage work across the project lifecycle.

**Common uses:** Agile software development (Scrum/Kanban), bug tracking, task/project management, requirements and test-case management, and IT service management (Jira Service Management).

**Editions:** **Jira Software** (Agile dev teams), **Jira Service Management** (ITSM/help desk), **Jira Work Management** (business teams). Hosting: **Cloud** vs **Data Center/Server** (self-hosted).

### Interview Q&A

**Q: What is Jira and what is it used for?**
**Jira** is an **Atlassian** tool for **issue tracking and project management**, especially **Agile (Scrum/Kanban)** software development. Teams use it to **plan, assign, track, and report** on work — from user stories and tasks to bugs — across the whole delivery lifecycle.

**Q: What are the main Jira products?**
**Jira Software** (Agile development), **Jira Service Management** (IT service desk/ITSM), and **Jira Work Management** (general business project management). All share the same core issue-tracking engine.

**Q: Why is Jira popular for Agile teams?**
It provides **Scrum and Kanban boards, backlogs, sprints, customizable workflows, JQL search, and rich reporting** out of the box, plus deep integration with the Atlassian ecosystem (Confluence, Bitbucket) and CI/CD tools — making it a one-stop tool for planning to delivery.

---

## 2. Projects & Project Types

### Theory

A **Project** in Jira is a **container for issues** — it groups related work (e.g., a product, team, or initiative) with its own configuration (workflows, permissions, boards).

- **Project key** — a short prefix (e.g., `DATA`) prepended to issue ids (`DATA-123`).
- **Team-managed projects** (formerly "next-gen") — configured by the team, simpler, self-contained settings.
- **Company-managed projects** (formerly "classic") — administered centrally, shareable configurations (schemes) across projects, more powerful for large orgs.

### Interview Q&A

**Q: What is a project in Jira?**
A **container that groups related issues** and defines their configuration (workflows, permissions, boards, components). Every issue belongs to a project and gets an id like `PROJECTKEY-number` (e.g., `DATA-42`).

**Q: Team-managed vs company-managed projects?**
**Team-managed** projects are **self-contained and configured by the team** — simpler, faster to set up, ideal for autonomous teams. **Company-managed** projects use **shared schemes administered centrally** — more standardized and powerful for large organizations needing consistency across many projects.

---

## 3. Issues & Issue Types

### Theory

An **Issue** is the **fundamental unit of work** in Jira — anything that needs tracking. Everything in Jira is an issue (a story, bug, task, etc.).

**Common issue types:**
- **Epic** — a large body of work spanning multiple stories.
- **Story** — a user-facing feature/requirement (Agile user story).
- **Task** — a piece of work that isn't a user story.
- **Bug** — a defect to fix.
- **Sub-task** — a smaller unit broken out of a parent issue.

**Each issue has fields:** summary, description, **assignee**, **reporter**, **priority**, **status**, labels, components, fix version, etc.

### Interview Q&A

**Q: What is an issue in Jira?**
The **basic unit of work** — anything tracked in Jira, such as a story, task, bug, or sub-task. Each issue has fields (summary, assignee, status, priority) and moves through a **workflow** during its lifecycle.

**Q: What are the common issue types?**
**Epic** (large initiative), **Story** (user feature), **Task** (general work), **Bug** (defect), and **Sub-task** (a breakdown of a parent issue). Issue types are customizable per project.

**Q: Assignee vs reporter?**
The **reporter** is the person who **created/raised** the issue; the **assignee** is the person **currently responsible** for working on it. They can be different people.

---

## 4. Issue Hierarchy

### Theory

Jira organizes work in a hierarchy:

```
Epic            (large feature / initiative)
 └── Story / Task   (deliverable units of work)
       └── Sub-task (small steps within a story/task)
```

(With Advanced Roadmaps you can add higher levels like **Initiative** above Epic.)

- **Epic** groups related stories toward a big goal.
- **Story/Task** is sized to fit (usually) within a sprint.
- **Sub-task** breaks a story into smaller assignable pieces.

### Interview Q&A

**Q: Explain the Epic → Story → Sub-task hierarchy.**
An **Epic** is a large body of work (a major feature/initiative) that's broken into **Stories** (or Tasks) — individual deliverables sized to fit in a sprint — which can be further split into **Sub-tasks** (small steps). This lets teams plan at high level (epics) and execute at detailed level (stories/sub-tasks).

**Q: Epic vs Story?**
An **Epic** is **too big to finish in one sprint** and spans many stories. A **Story** is a **single, sprint-sized user requirement**. Epics provide grouping/roadmap context; stories are the working units.

---

## 5. Workflows: Statuses & Transitions

### Theory

A **Workflow** is the **set of statuses and transitions an issue moves through** during its lifecycle — it models your team's process.

- **Status** — the **current state** of an issue (e.g., To Do, In Progress, In Review, Done).
- **Transition** — a **link/action** that moves an issue from one status to another (e.g., "Start Progress").
- **Resolution** — why an issue is closed (Done, Won't Do, Duplicate). An issue can be in a "Done" status but should also have a **resolution set**.
- Workflows can have **conditions** (who can transition), **validators** (required fields), and **post-functions** (auto-actions on transition, e.g., assign, update field).

**Default simple workflow:** `To Do → In Progress → Done`.

### Example

```
[To Do] --Start Progress--> [In Progress] --Ready for Review--> [In Review]
   ^                                                               |
   |                                                          Approve
   +------------------------ Reopen <----------- [Done] <----------+
```

### Interview Q&A

**Q: What is a Jira workflow?**
A **defined set of statuses and transitions** that an issue travels through during its lifecycle, representing your team's **process** (e.g., To Do → In Progress → In Review → Done). It's fully customizable to match how the team works.

**Q: Status vs transition?**
A **status** is the **state** an issue is in (To Do, In Progress, Done). A **transition** is the **action that moves** an issue between statuses (e.g., "Start Progress" moves it from To Do to In Progress). Statuses are nodes; transitions are the arrows.

**Q: What is a resolution, and the common gotcha?**
A **resolution** records **why an issue was closed** (Done, Won't Do, Duplicate). **Gotcha:** an issue can be moved to a "Done" status but if the **resolution field isn't set**, it can still appear as "unresolved" in filters/boards — workflows should set resolution on closing transitions.

**Q: What are conditions, validators, and post-functions?**
They control transitions: **conditions** restrict **who/when** a transition can happen; **validators** ensure **required input** before allowing it (e.g., a comment); **post-functions** run **automatic actions after** the transition (assign the issue, set resolution, update a field).

---

## 6. Agile in Jira: Scrum vs Kanban

### Theory

Jira Software supports two Agile methodologies:

- **Scrum** — **iterative, sprint-based** delivery. Work is planned into fixed-length **sprints** (1–4 weeks) with a committed scope. Best for **feature/product development** teams. Ceremonies: sprint planning, daily standup, sprint review, retrospective.
- **Kanban** — **continuous flow** with **just-in-time** delivery. Work is pulled as capacity allows; focus on **visualizing flow** and **limiting Work In Progress (WIP)**. No fixed sprints. Best for **operations/support/maintenance** teams.

**Key Agile terms:** **Backlog** (prioritized list of work), **User Story**, **Story Points** (effort estimate), **Velocity** (work completed per sprint), **WIP limit**.

### Interview Q&A

**Q: Scrum vs Kanban in Jira?**
**Scrum** is **sprint-based** — work is committed into fixed-length iterations with defined scope; suited to **feature/product development**. **Kanban** is **continuous flow** — work is pulled continuously with **WIP limits**, no sprints; suited to **operations/support** teams with ongoing, unpredictable work. Scrum optimizes for planned iterations; Kanban for flow and responsiveness.

**Q: When would you choose Kanban over Scrum?**
When work is **continuous, reactive, or unpredictable** (support, ops, maintenance) and doesn't fit fixed sprint commitments — Kanban's **continuous flow + WIP limits** handle it better. Scrum suits teams that can plan committed scope into time-boxed sprints.

**Q: What is a WIP limit?**
A **cap on the number of items allowed in a workflow stage at once** (Kanban). It prevents overloading, exposes bottlenecks, and **improves flow** by forcing the team to finish work before starting new work.

**Q: What are story points and velocity?**
**Story points** estimate the **relative effort/complexity** of a story (not hours). **Velocity** is the **average story points a team completes per sprint** — used to forecast how much the team can take on in future sprints.

---

## 7. Boards

### Theory

A **Board** is the **visual interface** for managing work — it displays issues as cards in columns mapped to workflow statuses.

- **Scrum board** — shows the **current sprint's** issues; has a backlog and sprint planning view.
- **Kanban board** — shows a **continuous flow** of work with WIP limits; no sprints.
- **Columns** map to **statuses**; **swimlanes** group rows (by assignee, epic, priority).
- A board is backed by a **filter (JQL)** that determines which issues appear.

### Interview Q&A

**Q: What is a board in Jira?**
A **visual workspace** that displays issues as cards in columns mapped to **workflow statuses**, letting the team **track and manage work** at a glance. **Scrum boards** focus on the current sprint; **Kanban boards** show continuous flow with WIP limits.

**Q: What are swimlanes?**
**Horizontal rows** on a board that **group issues** by a chosen criterion — assignee, epic, priority, or a custom query — making it easier to visualize and organize work (e.g., one swimlane per team member).

**Q: How does a board decide which issues to show?**
A board is powered by a **board filter (a JQL query)** — only issues matching that filter appear. The board's columns then map those issues to workflow statuses.

---

## 8. Sprints & the Backlog

### Theory

- **Backlog** — the **prioritized list of all pending work** (stories, tasks, bugs) not yet in a sprint. The Product Owner grooms/prioritizes it.
- **Sprint** — a **fixed time-box** (1–4 weeks, Scrum) in which the team commits to completing a set of backlog items.
- **Sprint lifecycle:** plan (pull items from backlog) → start sprint → work → close sprint (incomplete items return to backlog or move to next sprint).
- **Backlog grooming/refinement** — keeping the backlog prioritized, estimated, and ready.

### Interview Q&A

**Q: What is the backlog?**
The **prioritized list of all work not yet scheduled** into a sprint — stories, tasks, and bugs. The team **pulls from the top** of the backlog during sprint planning. It's continuously **refined/groomed** (prioritized and estimated).

**Q: What is a sprint and what happens to unfinished work?**
A **sprint** is a **fixed-length iteration** (commonly 2 weeks) in which the team completes a committed set of items. When a sprint closes, **incomplete issues return to the backlog** (or move to the next sprint) and the completed work counts toward **velocity**.

**Q: Walk me through the sprint lifecycle in Jira.**
**Plan** (estimate and pull backlog items into the sprint) → **start the sprint** → team works items through the board → **daily standups** track progress (burndown) → **close the sprint** at the end; unfinished items go back to the backlog, completed work is reviewed and adds to velocity.

---

## 9. Fields, Screens & Components

### Theory

- **Fields** — pieces of data on an issue. **System fields** (summary, assignee, priority) are built in; **custom fields** are user-defined (e.g., "Environment", "Story Points").
- **Screen** — the **arrangement of fields** shown for an operation (create / edit / transition). A **screen scheme** maps screens to issue operations.
- **Component** — a **sub-section of a project** (e.g., "Backend", "API") used to group issues; can have a default assignee.
- **Label** — a free-form tag for flexible grouping/filtering.
- **Version / Fix Version** — a release that an issue is targeted to/fixed in.

### Interview Q&A

**Q: System field vs custom field?**
**System fields** are **built into Jira** (summary, description, assignee, priority, status). **Custom fields** are **created by admins** to capture project-specific data (e.g., Story Points, Environment, Severity). Custom fields should be used judiciously to avoid clutter and performance issues.

**Q: What is a component?**
A **logical sub-grouping within a project** (e.g., "Frontend", "Database") used to **categorize issues** and optionally **auto-assign** a default owner — helpful for organizing and reporting on parts of a product.

**Q: Component vs label?**
A **component** is a **predefined, admin-managed** sub-section of a project (structured). A **label** is a **free-form, user-added tag** for ad-hoc grouping across projects. Components are structured; labels are flexible.

**Q: What is a fix version?**
The **release version** an issue is **planned to be delivered in / was fixed in** (e.g., `v2.1`). It enables release tracking and reports like the Release/Version report.

---

## 10. JQL (Jira Query Language)

### Theory

**JQL** is Jira's powerful **search query language** for finding and filtering issues — like SQL `WHERE` clauses for Jira. Structure: `field operator value [ORDER BY field]`, combined with `AND`/`OR`/`NOT`.

**Common fields:** `project`, `status`, `assignee`, `reporter`, `priority`, `type`, `sprint`, `labels`, `created`, `updated`, `resolution`.
**Operators:** `=`, `!=`, `>`, `<`, `IN`, `NOT IN`, `~` (contains), `IS`, `IS NOT`, `WAS`, `CHANGED`.
**Functions:** `currentUser()`, `now()`, `startOfDay()`, `openSprints()`, `membersOf()`.

Saved JQL queries are called **filters**, which power boards, dashboards, and subscriptions.

### Example

```sql
-- My open, high-priority bugs in the DATA project
project = DATA AND issuetype = Bug AND priority = High
  AND assignee = currentUser() AND status != Done
ORDER BY created DESC

-- Issues updated in the last 7 days
updated >= -7d ORDER BY updated DESC

-- Unresolved issues in the current sprint
sprint IN openSprints() AND resolution = Unresolved
```

### Interview Q&A

**Q: What is JQL?**
**Jira Query Language** — an advanced, SQL-like syntax to **search and filter issues** by fields, operators, and functions (e.g., `project = DATA AND status = "In Progress" AND assignee = currentUser()`). Saved JQL queries become **filters** that power boards and dashboards.

**Q: Give an example of a useful JQL query.**
`assignee = currentUser() AND status != Done ORDER BY priority DESC` — "all my open issues, highest priority first." Or `sprint IN openSprints() AND resolution = Unresolved` for current-sprint open work.

**Q: What is a filter in Jira?**
A **saved JQL search**. Filters can be **shared, subscribed to (email digests)**, and used as the **data source for boards and dashboard gadgets** — making them central to reporting and views.

**Q: What's the difference between `=` and `~` in JQL?**
`=` is an **exact match**; `~` means **"contains"** (text search), e.g., `summary ~ "login"` finds issues whose summary contains "login".

---

## 11. Time Tracking & Estimation

### Theory

Jira tracks effort on issues:

- **Original Estimate** — initial estimate of time to resolve (shown **blue**).
- **Remaining Estimate** — time left (shown **orange**, labeled "Remaining").
- **Time Spent / Logged** — actual time logged via **work logs** (shown **green**, labeled "Logged").

For Agile estimation, teams often use **Story Points** (relative effort) instead of, or alongside, time.

### Interview Q&A

**Q: What do the three time-tracking colors mean?**
**Blue = Original Estimate** (planned time), **Orange = Remaining** (time left), **Green = Logged** (time actually spent). Together they show estimate vs progress vs reality on an issue.

**Q: Story points vs time estimates?**
**Story points** measure **relative effort/complexity/uncertainty** (abstract, team-relative) and are used for **sprint forecasting via velocity**. **Time estimates** (hours) measure **actual duration**. Many Agile teams prefer story points because absolute time estimates are often inaccurate.

**Q: How do you log work?**
On an issue, use **"Log Work"** to record **time spent** (and optionally adjust the remaining estimate). This accumulates into the issue's **Logged** total and feeds time-tracking reports.

---

## 12. Reports & Dashboards

### Theory

Jira provides reporting to track progress and team performance.

**Common Agile reports:**
- **Burndown chart** — remaining work in a sprint over time (toward zero). Shows if you're on track.
- **Burnup chart** — completed work vs total scope (separates scope changes).
- **Velocity chart** — story points completed per sprint (forecasting).
- **Cumulative Flow Diagram (CFD)** — issue count per status over time (spots bottlenecks/WIP).
- **Control chart**, **Sprint report**, **Created vs Resolved**, **Release/Version report**.

**Dashboards** — customizable pages of **gadgets** (charts, filter results, statistics) that centralize tracking; can be shared with teams.

### Interview Q&A

**Q: What does a burndown chart show?**
The **remaining work in a sprint over time**, ideally trending down to zero by sprint end. It shows whether the team is **on track**, ahead, or behind, and highlights scope changes or stalled progress.

**Q: Burndown vs burnup vs CFD?**
**Burndown** tracks **remaining work** over time. **Burnup** tracks **completed work against total scope** (so scope changes are visible). **Cumulative Flow Diagram** shows the **number of issues in each status over time**, helping spot **bottlenecks and WIP buildup**.

**Q: What is a velocity chart used for?**
It shows **story points completed per sprint**, so teams can compute an **average velocity** and **forecast** how much work to commit to in upcoming sprints.

**Q: What is a Jira dashboard?**
A **customizable page of gadgets** (charts, filter results, statistics, sprint health) that **centralizes project tracking and reporting**. Dashboards can be **personal or shared** with a team for at-a-glance visibility.

---

## 13. Permissions, Roles & Notifications

### Theory

- **Project roles** — named roles (e.g., Administrator, Developer, Member) to which users/groups are assigned per project; used in permission and notification schemes.
- **Permission scheme** — defines **who can do what** in a project (create/edit/transition/admin).
- **Notification scheme** — defines **who gets emailed** on which events (issue created, commented, transitioned).
- **Groups** — global collections of users; **roles** are project-scoped (more flexible).

### Interview Q&A

**Q: What is a permission scheme?**
A **reusable set of rules** defining **what actions users/roles/groups can perform** in a project (create issues, edit, transition, administer). Assigning a scheme to a project centrally controls access.

**Q: Group vs project role?**
A **group** is a **global** collection of users (org-wide). A **project role** is **project-scoped** — the same role (e.g., "Developer") can map to different people in different projects. Roles are more flexible for per-project permissions and notifications.

**Q: How do notifications work?**
A **notification scheme** maps **events** (issue created, updated, commented, transitioned) to **recipients** (reporter, assignee, watchers, roles), so the right people are emailed automatically when relevant things happen.

---

## 14. Automation, Integrations & Add-ons

### Theory

- **Jira Automation** — **no-code rules** (trigger → condition → action) that automate repetitive work: auto-assign, transition on PR merge, notify on SLA breach, sync sub-tasks. E.g., "When all sub-tasks are Done → transition parent to Done."
- **Integrations:** **Confluence** (docs), **Bitbucket/GitHub/GitLab** (link commits/PRs/branches to issues), **CI/CD** (Jenkins, Bamboo), **Slack/Teams**.
- **Marketplace apps/add-ons** — extend Jira (e.g., **Advanced Roadmaps** for planning, **Zephyr/Xray** for test management, **Tempo** for time tracking).
- **Smart commits** — reference issues in Git commit messages to **transition or comment** (e.g., `DATA-123 #close #comment fixed bug`).

### Interview Q&A

**Q: What is Jira Automation?**
A **no-code rules engine** (trigger → condition → action) that **automates repetitive tasks** — auto-assigning issues, transitioning when a PR merges, sending alerts, syncing sub-tasks with parents. It reduces manual work and enforces process consistency.

**Q: How does Jira integrate with Git/CI/CD?**
By **linking commits, branches, and pull requests to issues** (via Bitbucket/GitHub/GitLab integration), and using **smart commits** to transition/comment on issues from commit messages. CI/CD tools (Jenkins, Bamboo) can report **build/deploy status** on issues — giving full traceability from code to ticket.

**Q: What are Jira add-ons/Marketplace apps?**
Third-party (or Atlassian) **plugins that extend Jira** — e.g., **Advanced Roadmaps** (cross-team planning), **Zephyr/Xray** (test management), **Tempo** (time tracking). They add capabilities beyond the core product.

---

## 15. Common Scenarios

**Q: How would you set up Jira for a new Agile team?**
"Create a **project** (team- or company-managed), choose **Scrum or Kanban** based on the team's work style, define **issue types** (Epic/Story/Task/Bug) and a **workflow** matching their process, create a **board** backed by a filter, set up the **backlog**, configure **permissions/notifications**, and add **dashboards** with burndown/velocity gadgets for tracking."

**Q: A team wants to track bugs separately from features. How?**
"Use the **Bug** issue type and **JQL filters/boards** to separate them (e.g., `issuetype = Bug`). I'd add a **component** or **label** for categorization, possibly a **swimlane** for bugs on the board, and a **dashboard gadget** (Created vs Resolved) to monitor bug trends."

**Q: How do you find all overdue, unresolved issues assigned to you?**
"A **JQL query**: `assignee = currentUser() AND resolution = Unresolved AND duedate < now() ORDER BY duedate ASC`. Save it as a **filter** and optionally add it to a **dashboard** or subscribe for email."

**Q: An issue is marked Done but still shows as unresolved in reports. Why?**
"Because the **resolution field wasn't set** — moving an issue to a Done **status** doesn't automatically set a **resolution**. The fix is to ensure the closing **transition has a post-function (or screen) that sets the resolution**, so it's correctly counted as resolved."

**Q: How do you handle incomplete work at the end of a sprint?**
"At sprint close, Jira lets you **move incomplete issues back to the backlog** or **to the next sprint**. I'd review *why* they weren't finished (over-commitment, blockers, scope creep) in the **retrospective** and adjust the next sprint's commitment using **velocity**."

---

## 16. Tricky / Gotcha Questions

**1. Status ≠ Resolution.** An issue can be in a "Done" status but still "unresolved" if the **resolution field isn't set** — a classic reporting bug.

**2. Everything is an issue** — stories, bugs, tasks, sub-tasks are all "issues" with different types.

**3. A board is just a view over a JQL filter** — it doesn't "own" issues; it displays those matching its filter.

**4. Scrum = sprints; Kanban = continuous flow + WIP limits.** Don't mix them up.

**5. Story points ≠ hours** — they measure relative effort, not time.

**6. Component (structured, admin) vs Label (free-form, anyone).**

**7. Group (global) vs Project Role (project-scoped).** Roles are preferred for per-project permissions.

**8. Closing a sprint sends unfinished issues back to backlog/next sprint** — they aren't lost.

**9. Team-managed vs company-managed** projects differ in who configures them and whether schemes are shared.

**10. `~` means "contains"; `=` means exact match** in JQL.

**11. Transitions carry conditions/validators/post-functions** — that's where automation on status change lives.

**12. Velocity is an average for forecasting**, not a target or performance score.

---

## 17. Rapid-Fire One-Liners

- **Jira** = Atlassian issue-tracking + project-management tool, big in Agile.
- **Project** = container of issues; issues are keyed `KEY-123`.
- **Issue** = basic unit of work (Epic, Story, Task, Bug, Sub-task).
- **Hierarchy:** Epic → Story/Task → Sub-task.
- **Workflow** = statuses + transitions an issue moves through.
- **Status** = state; **transition** = action between states; **resolution** = why closed.
- **Conditions/validators/post-functions** govern transitions.
- **Scrum** = sprint-based (dev); **Kanban** = continuous flow + WIP limits (ops).
- **Backlog** = prioritized pending work; **Sprint** = fixed time-box.
- **Board** = visual view over a JQL filter; **swimlanes** group rows.
- **Story points** = relative effort; **velocity** = points/sprint (forecasting).
- **JQL** = SQL-like issue search; saved JQL = **filter**.
- **`=`** exact, **`~`** contains, **`currentUser()`**, **`openSprints()`**.
- **Time tracking:** blue = original, orange = remaining, green = logged.
- **Burndown** = remaining work; **velocity** = throughput; **CFD** = WIP/bottlenecks.
- **Dashboard** = gadgets centralizing tracking.
- **Permission scheme** = who can do what; **notification scheme** = who's emailed.
- **Group** (global) vs **role** (project-scoped).
- **Component** (structured) vs **label** (free-form).
- **Automation** = trigger → condition → action (no-code).
- **Smart commits** link Git commits/PRs to issues; integrates with Bitbucket/CI-CD/Confluence.

---

## 18. Last-Minute Checklist

The hour before:

- [ ] What Jira is and its main products (Software / Service Management / Work Management).
- [ ] Project, issue, and **issue types**; the **Epic → Story → Sub-task hierarchy**.
- [ ] **Workflow**: statuses vs transitions vs resolution; conditions/validators/post-functions.
- [ ] **Scrum vs Kanban** (and when to use each); WIP limits.
- [ ] **Boards** (Scrum vs Kanban), swimlanes, board = filter.
- [ ] **Backlog & sprints**: sprint lifecycle, unfinished-work handling.
- [ ] Fields/screens, **component vs label**, fix version.
- [ ] **JQL**: structure, common fields/operators/functions, `=` vs `~`, filters.
- [ ] **Time tracking** colors (blue/orange/green); story points vs hours; velocity.
- [ ] **Reports**: burndown, burnup, velocity, CFD; dashboards & gadgets.
- [ ] **Permissions/roles** and notification schemes; group vs role.
- [ ] **Automation**, Git/CI-CD integration, smart commits, add-ons.
- [ ] Practice the **scenarios** in §15 (set up a team, find overdue issues, the resolution gotcha).

**Interview tips:** Lead with **Jira = Agile issue tracking and project management**. Nail **Scrum vs Kanban**, the **workflow (status/transition/resolution)** model, and **JQL basics** — these come up most. If you've used Jira on real projects (you have — per your background), describe your actual workflow: "We tracked stories/bugs on a **Scrum board**, planned **2-week sprints** from a groomed **backlog**, used **JQL filters and dashboards** for tracking, and linked **Git commits/PRs** to issues." Concrete experience beats definitions.

---

*Good luck — read it top-to-bottom once, then drill Scrum vs Kanban, the workflow (status/transition/resolution) model, the Epic→Story→Sub-task hierarchy, and JQL basics. Those are the most-asked Jira fundamentals.*
