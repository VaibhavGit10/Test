# DSV360 Sprint Module - POC Document

**Project:** DSV360 Time Entry Management System  
**Module:** Sprint Management (Jira-like Sprint Planning)  
**Platform:** Zoho Catalyst  
**Date:** February 11, 2026  
**Environment:** Development → Production

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Current System Analysis](#current-system-analysis)
3. [Proposed Architecture](#proposed-architecture)
4. [Data Model & Database Schema](#data-model--database-schema)
5. [API Design](#api-design)
6. [Implementation Plan](#implementation-plan)
7. [AI Agent Integration Strategy](#ai-agent-integration-strategy)
8. [Testing & Validation](#testing--validation)

---

## 1. Executive Summary

### Objective
Build a Jira-like Sprint Module for DSV360 that enables agile project management with proper hierarchy, sprint planning, and velocity tracking capabilities.

### Key Requirements
- ✅ **Hierarchy Management**: Milestone → Epic → Story → Task → Subtask
- ✅ **Sprint Planning**: Create sprints, assign stories, track progress
- ✅ **Issue Integration**: Link existing Issues table to Stories
- ✅ **Capacity Planning**: Story points, velocity, sprint capacity
- ✅ **Multi-tenant**: Organization-level data isolation
- ✅ **Role-based Access**: Leverage existing 13 roles
- ✅ **AI-Ready**: Designed for future AI agent integration

### Architecture Decision
- **Backend**: Node.js (Zoho Catalyst Functions)
- **Database**: Zoho Data Store (relational, ZCQL queries)
- **Frontend**: React (existing react-app)
- **Authentication**: Zoho Catalyst User Management

---

## 2. Current System Analysis

### 2.1 Existing Infrastructure

**Platform Stack:**
- Zoho Catalyst Serverless
- Node.js Functions (Advanced I/O)
- Zoho Data Store (SQL-like)
- React Frontend

**Existing Tables (Production):**
```
✅ Issues         - Bug/issue tracking
✅ Users          - User management with roles
✅ Projects       - Project data
✅ Tasks          - Task management
```

**New Tables (Development - Already Created):**
```
✅ Milestones     - Project milestones
✅ Epics          - Epic tracking
✅ Stories        - User stories
✅ Sprints        - Sprint cycles
```

**Missing Tables (Need to Create):**
```
❌ Sprint_Items   - Stories/Issues in sprints
❌ Subtasks       - Task breakdown
❌ Story_Points   - Point history/estimation
```

### 2.2 Current Implementation Analysis

**Existing Code Structure:**
```
functions/sprints_management_function/
├── index.js                    # Express routes
├── controller/
│   ├── sprintController.js     # Sprint CRUD
│   └── hierarchyController.js  # Milestone/Epic/Story CRUD
├── models/
│   ├── sprintRepo.js          # Sprint data access
│   ├── milestoneRepo.js       # Milestone data access
│   ├── epicRepo.js            # Epic data access
│   └── storyRepo.js           # Story data access
├── services/
│   ├── sprintService.js       # Sprint business logic
│   └── hierarchyService.js    # Hierarchy business logic
├── middlewares/
│   └── requireUser.js         # Auth middleware
└── utils/
    ├── constants.js           # Table names
    ├── user.js                # User helpers
    └── validate.js            # Validation
```

**Current Hierarchy in Code:**
```
Milestone
  └── Epic
      └── Story (with SprintID)
```

### 2.3 Issues with Current Flow

**Your Proposed Flow:**
> "Milestone/Project → Epic → Story → (Issues as stories) → Sprint"

**Problems Identified:**

1. **Missing Task/Subtask Layer**: No granular breakdown below Story
2. **Sprint-Story Relationship**: Direct SprintID in Stories table limits flexibility
3. **Issue Integration**: No proper link between Issues and Stories
4. **No Point System**: Missing story points, velocity tracking
5. **Status Management**: Limited workflow states
6. **Capacity Planning**: No team capacity or sprint metrics

---

## 3. Proposed Architecture

### 3.1 Corrected Hierarchy

```
Organization
  └── Project
      └── Milestone (Release/Phase)
          └── Epic (Feature Set)
              └── Story (User Story)
                  ├── Task (Development Task)
                  │   └── Subtask (Granular Work)
                  └── Issue (Bug/Defect) [Optional Link]

Sprint (Separate Entity)
  └── Sprint_Items
      ├── Stories (primary work items)
      └── Issues (bugs/hotfixes)
```

### 3.2 Key Design Principles

**1. Separation of Concerns**
- Sprint is NOT a container in the hierarchy
- Sprint is a **time-boxed planning unit** that references work items
- Work items (Stories, Issues) can exist independently of sprints

**2. Flexibility**
- Stories can be planned (assigned to sprint) or unplanned (backlog)
- Issues can be independent or linked to Stories
- Tasks are always children of Stories

**3. Many-to-Many Relationships**
- Sprint ↔ Stories: Many-to-many via Sprint_Items
- Story ↔ Issues: One-to-many (Story can have multiple linked Issues)
- Story ↔ Tasks: One-to-many (Story has multiple Tasks)
- Task ↔ Subtasks: One-to-many (Task has multiple Subtasks)

**4. Status Workflow**
```
Story/Task States: Todo → In Progress → In Review → Done
Sprint States: Planning → Active → Completed → Closed
Issue States: Open → In Progress → Resolved → Closed
```

---

## 4. Data Model & Database Schema

### 4.1 Existing Tables (Keep As-Is)

#### Users Table (Production)
```sql
-- Already exists, no changes needed
ROWID, User_id, Username, OrgID, RoleID, TeamID, 
TeamName, ReporterID, ReporterName, EmpID, Location
```

#### Projects Table (Production)
```sql
-- Already exists, no changes needed
ROWID, Project_Name, Status, Start_Date, End_Date, 
Description, Owner, Client_ID, Client_Name
```

#### Issues Table (Production)
```sql
-- Already exists, add StoryID for linking
ROWID, Issue_Name, Reporter_ID, Status, Due_Date, 
Severity, Project_ID, Description, Assignee_ID,
StoryID (NEW - nullable, links to Stories)
```

### 4.2 Hierarchy Tables (Keep & Extend)

#### Milestones Table (Development)
```sql
-- Already created
TABLE: Milestones
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  ProjectID        VARCHAR(255)
  Title            VARCHAR(255)
  Description      TEXT
  DueDate          DATE
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)
```

#### Epics Table (Development)
```sql
-- Already created
TABLE: Epics
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  ProjectID        VARCHAR(255)
  MilestoneID      VARCHAR(255) (FK → Milestones.ROWID)
  Title            VARCHAR(255)
  Description      TEXT
  Color            VARCHAR(50)
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)
```

#### Stories Table (Development - MODIFY)
```sql
-- Already created, REMOVE SprintID, add new fields
TABLE: Stories
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  ProjectID        VARCHAR(255)
  EpicID           VARCHAR(255) (FK → Epics.ROWID)
  -- SprintID      REMOVE THIS FIELD (use Sprint_Items instead)
  Title            VARCHAR(255)
  Description      TEXT
  Points           INT (story points: 1,2,3,5,8,13,21)
  Priority         VARCHAR(50) (Highest, High, Medium, Low, Lowest)
  Status           VARCHAR(50) (Todo, In Progress, In Review, Done)
  AssigneeID       VARCHAR(255) (FK → Users.ROWID)
  AcceptanceCriteria TEXT
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)
```

### 4.3 New Tables (Need to Create)

#### Tasks Table (NEW)
```sql
TABLE: Tasks
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  ProjectID        VARCHAR(255)
  StoryID          VARCHAR(255) (FK → Stories.ROWID)
  Title            VARCHAR(255)
  Description      TEXT
  Status           VARCHAR(50) (Todo, In Progress, In Review, Done)
  AssigneeID       VARCHAR(255) (FK → Users.ROWID)
  EstimatedHours   DECIMAL(10,2)
  ActualHours      DECIMAL(10,2)
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)

Indexes:
  - OrgID, StoryID
  - AssigneeID
```

**Note:** The existing Tasks table in production is for general project tasks. 
This new Tasks table is specifically for Sprint/Story breakdown. Consider:
- Option 1: Rename existing → Project_Tasks, create new → Sprint_Tasks
- Option 2: Add StoryID column to existing Tasks table
- Recommended: Option 1 for cleaner separation

#### Subtasks Table (NEW)
```sql
TABLE: Subtasks
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  TaskID           VARCHAR(255) (FK → Tasks.ROWID)
  Title            VARCHAR(255)
  Description      TEXT
  Status           VARCHAR(50) (Todo, In Progress, Done)
  AssigneeID       VARCHAR(255) (FK → Users.ROWID)
  EstimatedHours   DECIMAL(10,2)
  ActualHours      DECIMAL(10,2)
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)

Indexes:
  - OrgID, TaskID
  - AssigneeID
```

#### Sprints Table (Development - MODIFY)
```sql
-- Already created, add new fields
TABLE: Sprints
Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  ProjectID        VARCHAR(255)
  SprintName       VARCHAR(255)
  Goal             TEXT
  StartDate        DATE
  EndDate          DATE
  Status           VARCHAR(50) (Planning, Active, Completed, Closed)
  Capacity         INT (total story points capacity)
  IsDeleted        TINYINT (0/1)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)

Indexes:
  - OrgID, ProjectID, Status
  - StartDate, EndDate
```

#### Sprint_Items Table (NEW - CRITICAL)
```sql
TABLE: Sprint_Items
Purpose: Many-to-many relationship between Sprints and work items

Columns:
  ROWID            BIGINT (PK, auto)
  OrgID            VARCHAR(255)
  SprintID         VARCHAR(255) (FK → Sprints.ROWID)
  ItemType         VARCHAR(50) (Story, Issue)
  ItemID           VARCHAR(255) (FK → Stories.ROWID or Issues.ROWID)
  CommittedPoints  INT (points at time of sprint planning)
  AddedDate        DATETIME
  RemovedDate      DATETIME (nullable, if removed from sprint)
  IsActive         TINYINT (1 = currently in sprint, 0 = removed)
  CREATEDTIME      DATETIME (auto)
  MODIFIEDTIME     DATETIME (auto)

Indexes:
  - OrgID, SprintID, IsActive
  - ItemType, ItemID
  - UNIQUE(SprintID, ItemType, ItemID, IsActive)

Note: This table allows:
- Adding/removing stories from sprint
- Tracking sprint changes
- Historical sprint data
- Multiple work item types
```

### 4.4 Data Model Relationships

```
┌──────────────┐
│ Organization │
└──────┬───────┘
       │
       ├─► Projects ──┬─► Milestones ──► Epics ──► Stories ──┬─► Tasks ──► Subtasks
       │              │                                       │
       │              │                                       └─► Issues (linked)
       │              │
       └──────────────┴─► Sprints ◄─────── Sprint_Items ──────┘
                                              (Many-to-Many)
```

### 4.5 Role-Based Access Control

**Existing Roles (Use as-is):**
- App Administrator: Full access
- Super Admin: Full access
- Admin: Full access
- Manager/Team Lead: Full sprint management, can edit all items
- Team Lead: Sprint management for their team
- Developer: Can update assigned items
- Business Analyst: Can create/edit stories, view sprints
- MIS Analyst: View only
- Sales/Pre Sales: View only
- App User: Limited access
- Contacts: External view
- Intern: Limited access

**Permission Matrix:**
```
Action                  | Admin | Manager | Team Lead | Developer | Analyst
─────────────────────────────────────────────────────────────────────────────
Create Sprint           |  ✓    |    ✓    |     ✓     |           |
Edit Sprint             |  ✓    |    ✓    |     ✓     |           |
Delete Sprint           |  ✓    |    ✓    |           |           |
Add Story to Sprint     |  ✓    |    ✓    |     ✓     |           |    ✓
Create Story            |  ✓    |    ✓    |     ✓     |           |    ✓
Edit Story              |  ✓    |    ✓    |     ✓     |    ✓*     |    ✓
Create Task             |  ✓    |    ✓    |     ✓     |    ✓      |
Update Task Status      |  ✓    |    ✓    |     ✓     |    ✓**    |
View Sprint Board       |  ✓    |    ✓    |     ✓     |    ✓      |    ✓

* Only if assigned
** Only for assigned tasks
```

---

## 5. API Design

### 5.1 Base Configuration

**Existing Function:**
```
Function: sprints_management_function
URL: https://api.catalyst.zoho.in/baas/v1/project/{PROJECT_ID}/function/sprints_management_function/execute
Method: POST with request body OR GET with query params
Auth: Zoho Catalyst Session (catalyst.initialize(req))
```

### 5.2 API Endpoints

#### 5.2.1 Sprint Management APIs

```javascript
// ==================== SPRINTS ====================

// GET /sprints
// List all sprints for a project
Query Params: ?projectId=123&status=Active
Response: {
  success: true,
  data: [
    {
      ROWID: "1001",
      SprintName: "Sprint 1",
      Goal: "Authentication module",
      StartDate: "2026-02-10",
      EndDate: "2026-02-23",
      Status: "Active",
      Capacity: 40,
      CommittedPoints: 35,
      CompletedPoints: 20,
      VelocityPercent: 57
    }
  ]
}

// POST /sprints
// Create new sprint
Body: {
  projectId: "123",
  sprintName: "Sprint 2",
  goal: "User profile",
  startDate: "2026-02-24",
  endDate: "2026-03-09",
  capacity: 45
}

// GET /sprints/:id
// Get sprint details with all items
Response: {
  success: true,
  data: {
    sprint: { /* sprint data */ },
    stories: [ /* stories in sprint */ ],
    issues: [ /* issues in sprint */ ],
    metrics: {
      totalPoints: 35,
      completedPoints: 20,
      remainingPoints: 15,
      progressPercent: 57,
      daysRemaining: 5
    }
  }
}

// PATCH /sprints/:id
// Update sprint (name, dates, capacity, status)
Body: {
  sprintName: "Updated Name",
  status: "Completed"
}

// DELETE /sprints/:id
// Soft delete sprint
Response: { success: true }

// POST /sprints/:id/start
// Start a sprint (change status to Active)
Response: { success: true, sprintId: "1001" }

// POST /sprints/:id/complete
// Complete a sprint (move incomplete items to backlog)
Body: { moveIncompleteToNextSprint: false }
Response: {
  success: true,
  completedPoints: 20,
  velocity: 20,
  uncompletedItems: 3
}
```

#### 5.2.2 Sprint Planning APIs

```javascript
// ==================== SPRINT PLANNING ====================

// POST /sprints/:id/items
// Add story/issue to sprint
Body: {
  itemType: "Story",
  itemId: "5001",
  committedPoints: 5
}

// DELETE /sprints/:sprintId/items/:itemId
// Remove story/issue from sprint
Response: { success: true }

// GET /sprints/:id/backlog
// Get unplanned stories for sprint planning
Query Params: ?epicId=2001
Response: {
  success: true,
  data: [
    {
      ROWID: "5001",
      Title: "User login",
      Points: 5,
      Priority: "High",
      Status: "Todo",
      EpicName: "Authentication"
    }
  ]
}

// POST /sprints/:id/items/bulk
// Add multiple items to sprint at once
Body: {
  items: [
    { itemType: "Story", itemId: "5001", committedPoints: 5 },
    { itemType: "Story", itemId: "5002", committedPoints: 3 }
  ]
}
```

#### 5.2.3 Hierarchy APIs (Extend Existing)

```javascript
// ==================== STORIES ====================

// GET /stories
// List stories (already exists)
Query Params: ?projectId=123&epicId=2001&status=Todo

// POST /stories
// Create story (already exists)

// PATCH /stories/:id
// Update story (already exists)

// GET /stories/:id/tasks
// Get tasks for a story
Response: {
  success: true,
  data: [
    {
      ROWID: "7001",
      Title: "Create login form",
      Status: "In Progress",
      AssigneeID: "1001",
      AssigneeName: "John Doe",
      EstimatedHours: 4,
      ActualHours: 2
    }
  ]
}

// ==================== TASKS ====================

// POST /stories/:storyId/tasks
// Create task under story
Body: {
  title: "Create login form",
  description: "Build React login component",
  assigneeId: "1001",
  estimatedHours: 4
}

// PATCH /tasks/:id
// Update task
Body: {
  status: "In Progress",
  actualHours: 2
}

// DELETE /tasks/:id
// Soft delete task

// GET /tasks/:id/subtasks
// Get subtasks for a task

// ==================== SUBTASKS ====================

// POST /tasks/:taskId/subtasks
// Create subtask under task

// PATCH /subtasks/:id
// Update subtask

// DELETE /subtasks/:id
// Soft delete subtask

// ==================== ISSUES (Extend) ====================

// PATCH /issues/:id
// Add storyId field to link issue to story
Body: { storyId: "5001" }
```

#### 5.2.4 Analytics & Reporting APIs

```javascript
// ==================== METRICS ====================

// GET /sprints/:id/burndown
// Sprint burndown chart data
Response: {
  success: true,
  data: {
    days: ["2026-02-10", "2026-02-11", ...],
    ideal: [35, 32, 29, 26, ...],
    actual: [35, 35, 32, 28, ...]
  }
}

// GET /sprints/:id/velocity
// Historical velocity chart
Response: {
  success: true,
  data: [
    { sprint: "Sprint 1", committed: 40, completed: 35, velocity: 35 },
    { sprint: "Sprint 2", committed: 35, completed: 35, velocity: 35 },
    { sprint: "Sprint 3", committed: 45, completed: 38, velocity: 38 }
  ]
}

// GET /team/:teamId/capacity
// Team capacity analysis
Response: {
  success: true,
  data: {
    totalMembers: 5,
    availableHours: 200,
    allocatedHours: 180,
    utilizationPercent: 90
  }
}

// GET /epics/:id/progress
// Epic progress across sprints
Response: {
  success: true,
  data: {
    totalStories: 15,
    completedStories: 8,
    totalPoints: 75,
    completedPoints: 40,
    progressPercent: 53
  }
}
```

### 5.3 Response Format Standards

```javascript
// Success Response
{
  success: true,
  data: { /* response data */ },
  message: "Operation completed successfully" // optional
}

// Error Response
{
  success: false,
  error: "Error message",
  code: "SPRINT_NOT_FOUND", // error code for client handling
  details: { /* additional error details */ } // optional
}

// Validation Error
{
  success: false,
  error: "Validation failed",
  code: "VALIDATION_ERROR",
  details: {
    fields: {
      sprintName: "Sprint name is required",
      startDate: "Start date must be before end date"
    }
  }
}
```

---

## 6. Implementation Plan

### 6.1 Phase 1: Foundation (Week 1-2)

**Database Setup**
```sql
-- Step 1: Modify existing tables
ALTER TABLE Issues ADD COLUMN StoryID VARCHAR(255);
ALTER TABLE Stories DROP COLUMN SprintID; -- if exists
ALTER TABLE Stories ADD COLUMN AcceptanceCriteria TEXT;
ALTER TABLE Sprints ADD COLUMN Capacity INT;

-- Step 2: Create new tables
CREATE TABLE Sprint_Items ( /* schema from section 4.3 */ );
CREATE TABLE Sprint_Tasks ( /* renamed from Tasks */ );
CREATE TABLE Subtasks ( /* schema from section 4.3 */ );

-- Step 3: Create indexes
CREATE INDEX idx_sprint_items_sprint ON Sprint_Items(SprintID, IsActive);
CREATE INDEX idx_sprint_items_item ON Sprint_Items(ItemType, ItemID);
CREATE INDEX idx_tasks_story ON Sprint_Tasks(StoryID);
CREATE INDEX idx_subtasks_task ON Subtasks(TaskID);
```

**Backend Structure**
```javascript
// New files to create:
functions/sprints_management_function/
├── models/
│   ├── taskRepo.js           // NEW
│   ├── subtaskRepo.js        // NEW
│   └── sprintItemRepo.js     // NEW
├── services/
│   ├── sprintPlanningService.js  // NEW
│   └── metricsService.js         // NEW
├── controllers/
│   ├── taskController.js     // NEW
│   └── metricsController.js  // NEW
```

**Tasks:**
1. ✅ Design database schema
2. ⏳ Create migration scripts
3. ⏳ Set up new tables in Development environment
4. ⏳ Create repository layer (taskRepo, subtaskRepo, sprintItemRepo)
5. ⏳ Write unit tests for repositories

### 6.2 Phase 2: Core Sprint APIs (Week 3-4)

**Sprint Management**
```javascript
// Implement sprint lifecycle
services/sprintService.js:
- createSprint()
- startSprint()
- completeSprint()
- calculateSprintMetrics()
- validateSprintDates()
```

**Sprint Planning**
```javascript
// Implement sprint planning
services/sprintPlanningService.js:
- addItemToSprint()
- removeItemFromSprint()
- bulkAddItems()
- getSprintBacklog()
- validateCapacity()
```

**Tasks:**
1. ⏳ Implement sprint CRUD operations
2. ⏳ Implement sprint planning APIs
3. ⏳ Add sprint status workflow
4. ⏳ Implement capacity validation
5. ⏳ Write integration tests
6. ⏳ Test with Postman/Insomnia

### 6.3 Phase 3: Hierarchy & Tasks (Week 5-6)

**Task Management**
```javascript
// Implement task operations
services/taskService.js:
- createTask()
- updateTask()
- getTasksByStory()
- updateTaskStatus()
- trackHours()

services/subtaskService.js:
- createSubtask()
- updateSubtask()
- getSubtasksByTask()
```

**Issue Integration**
```javascript
// Link issues to stories
services/issueService.js:
- linkIssueToStory()
- unlinkIssueFromStory()
- getIssuesByStory()
```

**Tasks:**
1. ⏳ Implement task CRUD
2. ⏳ Implement subtask CRUD
3. ⏳ Implement issue-story linking
4. ⏳ Add time tracking
5. ⏳ Write tests

### 6.4 Phase 4: Metrics & Analytics (Week 7-8)

**Sprint Metrics**
```javascript
services/metricsService.js:
- calculateBurndown()
- calculateVelocity()
- getSprintProgress()
- getTeamCapacity()
- getEpicProgress()
```

**Tasks:**
1. ⏳ Implement burndown chart
2. ⏳ Implement velocity tracking
3. ⏳ Implement capacity analytics
4. ⏳ Create dashboard APIs
5. ⏳ Write tests

### 6.5 Phase 5: Frontend Integration (Week 9-10)

**React Components**
```javascript
react-app/src/Admin/Sprints/
├── SprintBoard.jsx          // Kanban board view
├── SprintPlanning.jsx       // Sprint planning modal
├── SprintBacklog.jsx        // Product backlog
├── SprintMetrics.jsx        // Burndown, velocity charts
├── TaskCard.jsx             // Story/task card component
└── sprintsApi.js           // API integration

// Update existing:
react-app/src/redux/Sprints/
└── sprintSlice.js          // Redux state management
```

**Tasks:**
1. ⏳ Create sprint board UI
2. ⏳ Implement drag-and-drop
3. ⏳ Create sprint planning modal
4. ⏳ Implement metrics visualization
5. ⏳ Add task management UI
6. ⏳ Integrate with backend APIs

### 6.6 Phase 6: Testing & Deployment (Week 11-12)

**Testing**
```bash
# Unit tests
npm test -- --coverage

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e
```

**Deployment Steps**
```bash
# 1. Deploy to Development
catalyst deploy --env development

# 2. Run migration scripts
node scripts/migrate-dev.js

# 3. Test in Development
npm run test:dev

# 4. Deploy to Production
catalyst deploy --env production

# 5. Run production migrations
node scripts/migrate-prod.js

# 6. Monitor and verify
node scripts/verify-prod.js
```

**Tasks:**
1. ⏳ Write comprehensive test suite
2. ⏳ Perform load testing
3. ⏳ Create migration scripts
4. ⏳ Deploy to Development
5. ⏳ User acceptance testing
6. ⏳ Deploy to Production
7. ⏳ Monitor and fix issues

---

## 7. AI Agent Integration Strategy

### 7.1 AI Agent Use Cases

**Sprint Planning Assistant**
```
User: "Plan sprint 5 with focus on authentication"
AI: Analyzes backlog → Suggests stories → Validates capacity → Creates sprint
```

**Story Creation from Requirements**
```
User: "Create stories for user profile feature"
AI: Generates epic → Creates stories → Estimates points → Assigns to backlog
```

**Sprint Health Check**
```
User: "How is sprint 3 going?"
AI: Analyzes metrics → Identifies risks → Suggests actions
```

### 7.2 AI-Ready Data Structure

**Design Principles:**
1. **Descriptive Fields**: Rich text descriptions for context
2. **Status History**: Track all status changes
3. **Activity Logs**: Record all modifications
4. **Relationships**: Clear parent-child links

**AI Context Tables (Future)**
```sql
-- Track all changes for AI learning
TABLE: Sprint_Activity_Log
- ROWID, OrgID, SprintID, EntityType, EntityID
- Action, OldValue, NewValue, UserID, Timestamp

-- Store AI suggestions and outcomes
TABLE: AI_Suggestions
- ROWID, OrgID, SuggestionType, Context
- Suggestion, Accepted, Outcome, Timestamp

-- Track sprint metrics for ML
TABLE: Sprint_Metrics_History
- ROWID, SprintID, Date, CommittedPoints
- CompletedPoints, Velocity, Timestamp
```

### 7.3 AI Integration Points

**API Endpoints for AI**
```javascript
// GET /ai/suggest-stories
// AI suggests stories based on epic description
Body: { epicId: "2001", description: "User authentication" }
Response: {
  success: true,
  suggestions: [
    { title: "User registration", points: 5 },
    { title: "Login with email", points: 3 }
  ]
}

// GET /ai/estimate-points
// AI estimates story points
Body: { title: "...", description: "..." }
Response: { success: true, estimatedPoints: 5 }

// GET /ai/sprint-risks
// AI identifies sprint risks
Body: { sprintId: "1001" }
Response: {
  success: true,
  risks: [
    { type: "OVERCOMMITTED", severity: "High" },
    { type: "BLOCKED_TASKS", severity: "Medium" }
  ]
}
```

### 7.4 Existing AI Integration

**Your Current Setup:**
```javascript
// functions/ai_service/index.js
- RAG_ENDPOINT: Document Q&A
- LLM_ENDPOINT: Chat/Code generation
- VLM_ENDPOINT: Vision/Image analysis
```

**Integration Strategy:**
```javascript
// In sprints_management_function
const aiService = {
  async estimateStoryPoints(title, description) {
    const prompt = `
      As a scrum master, estimate story points for:
      Title: ${title}
      Description: ${description}
      Use Fibonacci scale: 1,2,3,5,8,13,21
    `;
    return await callLLM(prompt);
  },

  async suggestTasks(storyDescription) {
    const prompt = `
      Break down this user story into tasks:
      ${storyDescription}
      Format: JSON array of {title, estimatedHours}
    `;
    return await callLLM(prompt);
  },

  async analyzeSprintHealth(sprintData) {
    const prompt = `
      Analyze sprint health and identify risks:
      ${JSON.stringify(sprintData)}
    `;
    return await callLLM(prompt);
  }
};
```

---

## 8. Testing & Validation

### 8.1 Test Data Setup

**Sample Data Structure:**
```javascript
// Organization
OrgID: "60040289923"

// Project
Project: { id: "P001", name: "DSV360 Enhancement" }

// Milestone
Milestone: { id: "M001", title: "Q1 Release", projectId: "P001" }

// Epic
Epic: { id: "E001", title: "User Management", milestoneId: "M001" }

// Stories
Story 1: { id: "S001", title: "User registration", epicId: "E001", points: 5 }
Story 2: { id: "S002", title: "User login", epicId: "E001", points: 3 }

// Tasks
Task 1: { id: "T001", title: "Create form", storyId: "S001" }
Task 2: { id: "T002", title: "API integration", storyId: "S001" }

// Sprint
Sprint: { id: "SP001", name: "Sprint 1", capacity: 40 }

// Sprint Items
SprintItem 1: { sprintId: "SP001", itemType: "Story", itemId: "S001" }
SprintItem 2: { sprintId: "SP001", itemType: "Story", itemId: "S002" }
```

### 8.2 Test Scenarios

**Scenario 1: Create Complete Hierarchy**
```
1. Create Milestone "Q1 Release"
2. Create Epic "User Management"
3. Create Story "User registration"
4. Create 2 Tasks under Story
5. Create 2 Subtasks under each Task
6. Verify hierarchy is correct
```

**Scenario 2: Sprint Planning**
```
1. Create Sprint "Sprint 1" with capacity 40
2. Add 5 stories (total 35 points)
3. Try to add story with 10 points → Should warn about capacity
4. Start sprint → Status changes to Active
5. Complete story → Points update
6. Complete sprint → Calculate velocity
```

**Scenario 3: Issue Linking**
```
1. Create Story "User login"
2. Discover bug during testing
3. Create Issue "Login fails on Chrome"
4. Link Issue to Story
5. Verify Issue appears under Story
```

**Scenario 4: Role-Based Access**
```
1. Login as Developer
2. Try to create Sprint → Should fail
3. Try to update assigned task → Should succeed
4. Try to delete sprint → Should fail
```

### 8.3 Validation Checklist

**Data Integrity**
- [ ] All foreign keys resolve correctly
- [ ] Soft delete works (IsDeleted flag)
- [ ] OrgID isolation is enforced
- [ ] Dates are validated (StartDate < EndDate)
- [ ] Story points use Fibonacci sequence

**Business Logic**
- [ ] Cannot add more points than capacity
- [ ] Cannot start sprint without items
- [ ] Cannot delete active sprint
- [ ] Story must belong to Epic
- [ ] Task must belong to Story
- [ ] Subtask must belong to Task

**API Behavior**
- [ ] All endpoints return correct status codes
- [ ] Error messages are clear and actionable
- [ ] Pagination works for large datasets
- [ ] Concurrent requests are handled
- [ ] Authentication is enforced

**Performance**
- [ ] Queries complete in < 500ms
- [ ] Bulk operations are optimized
- [ ] Indexes are used effectively
- [ ] No N+1 query problems

---

## 9. Deployment Checklist

### 9.1 Pre-Deployment

**Code Review**
- [ ] All code reviewed and approved
- [ ] No console.log in production code
- [ ] Error handling is comprehensive
- [ ] Security best practices followed

**Database**
- [ ] Migration scripts tested
- [ ] Rollback plan documented
- [ ] Backup taken
- [ ] Indexes created

**Testing**
- [ ] All tests passing
- [ ] Load testing completed
- [ ] Security testing done
- [ ] UAT sign-off received

### 9.2 Deployment Steps

**Development Environment**
```bash
# 1. Deploy function
cd functions/sprints_management_function
catalyst deploy --env development

# 2. Run migrations
node scripts/migrate.js --env development

# 3. Seed test data
node scripts/seed-data.js --env development

# 4. Verify
npm run test:integration -- --env development
```

**Production Environment**
```bash
# 1. Take backup
node scripts/backup.js --env production

# 2. Deploy function
catalyst deploy --env production

# 3. Run migrations (with caution)
node scripts/migrate.js --env production

# 4. Verify critical paths
node scripts/verify.js --env production

# 5. Monitor for 24 hours
node scripts/monitor.js --env production
```

### 9.3 Post-Deployment

**Monitoring**
- [ ] Error rates normal
- [ ] Response times acceptable
- [ ] No data corruption
- [ ] All features working

**Documentation**
- [ ] API documentation updated
- [ ] User guide created
- [ ] Admin guide created
- [ ] Troubleshooting guide created

**Training**
- [ ] Admin training completed
- [ ] User training completed
- [ ] Support team trained

---

## 10. Summary & Recommendations

### 10.1 Key Takeaways

**✅ Correct Flow:**
```
Milestone → Epic → Story → Task → Subtask
                    ↓
                 Sprint (via Sprint_Items)
                    ↓
                 Issues (optional link)
```

**✅ Critical Design Decisions:**
1. **Sprint_Items table**: Enables flexibility in sprint planning
2. **Separate Tasks/Subtasks**: Provides granular work breakdown
3. **Issue linking**: Integrates bug tracking with sprint workflow
4. **Story points**: Enables velocity tracking
5. **Soft deletes**: Maintains data integrity

**✅ Implementation Priority:**
1. Database schema and migrations (CRITICAL)
2. Core sprint APIs
3. Task/subtask hierarchy
4. Frontend integration
5. Metrics and analytics
6. AI agent features

### 10.2 Next Steps

**Immediate Actions (This Week):**
1. Review and approve this POC document
2. Set up Development environment tables
3. Create migration scripts
4. Start implementing repository layer

**Short-term Actions (Next 2 Weeks):**
1. Implement sprint CRUD APIs
2. Implement sprint planning APIs
3. Test with Postman
4. Deploy to Development

**Medium-term Actions (Next 1-2 Months):**
1. Complete task/subtask hierarchy
2. Build frontend components
3. Integrate with existing UI
4. User acceptance testing

**Long-term Actions (Next 3-6 Months):**
1. Implement advanced metrics
2. Build AI agent features
3. Optimize performance
4. Scale for larger teams

### 10.3 Risk Mitigation

**Risk 1: Existing Tasks Table Conflict**
- **Solution**: Rename existing → Project_Tasks, create new → Sprint_Tasks
- **Alternative**: Add StoryID column to existing table

**Risk 2: Data Migration Issues**
- **Solution**: Extensive testing in Development first
- **Mitigation**: Create rollback scripts

**Risk 3: Performance with Large Datasets**
- **Solution**: Proper indexing and query optimization
- **Mitigation**: Implement pagination early

**Risk 4: User Adoption**
- **Solution**: Comprehensive training and documentation
- **Mitigation**: Phased rollout starting with pilot team

---

## Appendix A: Quick Reference

### Table Names
```
Production:  Issues, Users, Projects, Tasks (general)
Development: Milestones, Epics, Stories, Sprints
New:         Sprint_Items, Sprint_Tasks, Subtasks
```

### Key Relationships
```
Sprint → Sprint_Items → Stories/Issues (many-to-many)
Story → Sprint_Tasks → Subtasks (one-to-many)
Story → Issues (one-to-many, optional link)
Epic → Stories (one-to-many)
Milestone → Epics (one-to-many)
Project → Milestones (one-to-many)
```

### Status Values
```
Sprint:  Planning | Active | Completed | Closed
Story:   Todo | In Progress | In Review | Done
Task:    Todo | In Progress | In Review | Done
Issue:   Open | In Progress | Resolved | Closed
```

### Story Points (Fibonacci)
```
1, 2, 3, 5, 8, 13, 21
```

---

## Appendix B: Code Samples

### Sample Repository Implementation

```javascript
// models/sprintItemRepo.js
"use strict";

const TABLE = "Sprint_Items";

async function addItemToSprint(datastore, { orgId, sprintId, itemType, itemId, committedPoints }) {
  return await datastore.table(TABLE).insertRow({
    OrgID: orgId,
    SprintID: sprintId,
    ItemType: itemType,
    ItemID: itemId,
    CommittedPoints: committedPoints,
    IsActive: 1,
    AddedDate: new Date().toISOString()
  });
}

async function removeItemFromSprint(datastore, { orgId, sprintId, itemId }) {
  // Soft remove by setting IsActive = 0 and RemovedDate
  const query = `
    SELECT ROWID FROM ${TABLE}
    WHERE OrgID='${orgId}' AND SprintID='${sprintId}' AND ItemID='${itemId}' AND IsActive=1
    LIMIT 1
  `;
  
  const result = await zcql.executeZCQLQuery(query);
  if (result.length === 0) {
    throw new Error("Item not found in sprint");
  }
  
  return await datastore.table(TABLE).updateRow({
    ROWID: result[0].ROWID,
    IsActive: 0,
    RemovedDate: new Date().toISOString()
  });
}

async function getSprintItems(zcql, { orgId, sprintId }) {
  const query = `
    SELECT si.*, 
           s.Title as StoryTitle, s.Points as StoryPoints, s.Status as StoryStatus,
           i.Issue_Name as IssueTitle, i.Status as IssueStatus
    FROM ${TABLE} si
    LEFT JOIN Stories s ON si.ItemType='Story' AND si.ItemID=s.ROWID
    LEFT JOIN Issues i ON si.ItemType='Issue' AND si.ItemID=i.ROWID
    WHERE si.OrgID='${orgId}' AND si.SprintID='${sprintId}' AND si.IsActive=1
    ORDER BY si.AddedDate ASC
  `;
  
  return await zcql.executeZCQLQuery(query);
}

module.exports = {
  addItemToSprint,
  removeItemFromSprint,
  getSprintItems
};
```

### Sample Service Implementation

```javascript
// services/sprintPlanningService.js
"use strict";

const sprintItemRepo = require("../models/sprintItemRepo");
const storyRepo = require("../models/storyRepo");
const sprintRepo = require("../models/sprintRepo");

class SprintPlanningService {
  
  async addStoryToSprint(req, { sprintId, storyId, committedPoints }) {
    const orgId = getOrgId(req.user);
    const datastore = req.catalystApp.datastore();
    const zcql = req.catalystApp.zcql();
    
    // 1. Validate sprint exists and is not completed
    const sprintResult = await sprintRepo.getById(zcql, { orgId, id: sprintId });
    const sprint = sprintResult[0];
    
    if (!sprint) {
      throw new Error("Sprint not found");
    }
    
    if (sprint.Status === "Completed" || sprint.Status === "Closed") {
      throw new Error("Cannot add items to completed sprint");
    }
    
    // 2. Validate story exists
    const storyQuery = `
      SELECT ROWID, Points FROM Stories 
      WHERE OrgID='${orgId}' AND ROWID='${storyId}' AND (IsDeleted IS NULL OR IsDeleted=0)
    `;
    const storyResult = await zcql.executeZCQLQuery(storyQuery);
    const story = storyResult[0];
    
    if (!story) {
      throw new Error("Story not found");
    }
    
    // 3. Check if story already in sprint
    const existingQuery = `
      SELECT ROWID FROM Sprint_Items
      WHERE OrgID='${orgId}' AND SprintID='${sprintId}' AND ItemType='Story' AND ItemID='${storyId}' AND IsActive=1
    `;
    const existing = await zcql.executeZCQLQuery(existingQuery);
    
    if (existing.length > 0) {
      throw new Error("Story already in sprint");
    }
    
    // 4. Check capacity
    const currentItems = await sprintItemRepo.getSprintItems(zcql, { orgId, sprintId });
    const currentPoints = currentItems.reduce((sum, item) => sum + (item.CommittedPoints || 0), 0);
    const newTotal = currentPoints + committedPoints;
    
    if (newTotal > sprint.Capacity) {
      throw new Error(`Adding this story would exceed sprint capacity (${newTotal} > ${sprint.Capacity})`);
    }
    
    // 5. Add to sprint
    const result = await sprintItemRepo.addItemToSprint(datastore, {
      orgId,
      sprintId,
      itemType: "Story",
      itemId: storyId,
      committedPoints
    });
    
    return {
      success: true,
      data: result,
      capacity: {
        total: sprint.Capacity,
        used: newTotal,
        remaining: sprint.Capacity - newTotal
      }
    };
  }
  
  async getBacklog(req, { projectId, epicId }) {
    const orgId = getOrgId(req.user);
    const zcql = req.catalystApp.zcql();
    
    // Get stories not in any active sprint
    let where = [`s.OrgID='${orgId}'`, `s.ProjectID='${projectId}'`, `(s.IsDeleted IS NULL OR s.IsDeleted=0)`];
    
    if (epicId) {
      where.push(`s.EpicID='${epicId}'`);
    }
    
    const query = `
      SELECT s.*, e.Title as EpicTitle
      FROM Stories s
      LEFT JOIN Epics e ON s.EpicID=e.ROWID
      WHERE ${where.join(" AND ")}
        AND s.ROWID NOT IN (
          SELECT ItemID FROM Sprint_Items
          WHERE OrgID='${orgId}' AND ItemType='Story' AND IsActive=1
        )
      ORDER BY s.Priority ASC, s.CREATEDTIME DESC
    `;
    
    const result = await zcql.executeZCQLQuery(query);
    
    return {
      success: true,
      data: result
    };
  }
  
  async completeSprint(req, { sprintId, moveIncompleteToNextSprint }) {
    const orgId = getOrgId(req.user);
    const datastore = req.catalystApp.datastore();
    const zcql = req.catalystApp.zcql();
    
    // 1. Get sprint items
    const items = await sprintItemRepo.getSprintItems(zcql, { orgId, sprintId });
    
    // 2. Calculate metrics
    const completedPoints = items
      .filter(item => item.StoryStatus === "Done")
      .reduce((sum, item) => sum + (item.CommittedPoints || 0), 0);
    
    const totalPoints = items.reduce((sum, item) => sum + (item.CommittedPoints || 0), 0);
    
    const velocity = completedPoints;
    
    // 3. Get incomplete items
    const incompleteItems = items.filter(item => item.StoryStatus !== "Done");
    
    // 4. Update sprint status
    await sprintRepo.softDelete(datastore, { orgId, id: sprintId });
    await datastore.table("Sprints").updateRow({
      ROWID: sprintId,
      Status: "Completed"
    });
    
    // 5. Handle incomplete items
    if (moveIncompleteToNextSprint && incompleteItems.length > 0) {
      // Find next sprint in planning state
      const nextSprintQuery = `
        SELECT ROWID FROM Sprints
        WHERE OrgID='${orgId}' AND Status='Planning' AND IsDeleted=0
        ORDER BY StartDate ASC
        LIMIT 1
      `;
      const nextSprint = await zcql.executeZCQLQuery(nextSprintQuery);
      
      if (nextSprint.length > 0) {
        const nextSprintId = nextSprint[0].ROWID;
        
        // Move incomplete items
        for (const item of incompleteItems) {
          await sprintItemRepo.removeItemFromSprint(datastore, {
            orgId,
            sprintId,
            itemId: item.ItemID
          });
          
          await sprintItemRepo.addItemToSprint(datastore, {
            orgId,
            sprintId: nextSprintId,
            itemType: item.ItemType,
            itemId: item.ItemID,
            committedPoints: item.CommittedPoints
          });
        }
      }
    }
    
    return {
      success: true,
      completedPoints,
      totalPoints,
      velocity,
      incompleteItemsCount: incompleteItems.length
    };
  }
}

module.exports = new SprintPlanningService();
```

---

**End of POC Document**

---

## Document Metadata

- **Version**: 1.0
- **Last Updated**: February 11, 2026
- **Author**: DSV360 Development Team
- **Status**: Draft for Review
- **Next Review**: February 18, 2026

---

## Contact & Support

For questions or clarifications about this POC:
- **Technical Lead**: [Your Name]
- **Project Manager**: [PM Name]
- **Email**: support@dsv360.ai
- **Slack**: #dsv360-sprints-module

---
