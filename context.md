# 🧠 ADHD Planner / Kanban Board (ADHDBoard) – Project Documentation

## 📘 Overview

A neurodivergent-friendly productivity tool designed to reduce overwhelm and improve focus. Combines visual Kanban-style task management with a planner, routine scheduler, and habit-building tools. Designed for web first, with future expansion to desktop.

---

## 🧱 Core Features (Initial Scope)

### 📌 Task Board (Kanban/Scrum Style)

- Columns:
  - [ ] To Do
  - [ ] In Progress
  - [ ] Waiting
  - [ ] On Hold
  - [ ] Done
- Drag-and-drop task movement
- Tasks belong to:
  - [ ] A project (color-coded)
  - [ ] A single board state
  - [ ] One stage within a project
- Projects belong to a category
- Priority levels (color coded)
- Deadlines with reminders
- Pomodoro timer integration
- Action prompts to encourage progress
- Structure should be user selectable (Vertical | Horizontal)
- Default to vertical
- Task notes/comments
- Optional color coding for projects/stages/priorities

---

## 📆 Daily Routine & Scheduler (Planned)

A separate interface for managing time-based habits and self-care routines.

### 🔄 Routine Features

- Repeating tasks (daily, weekly, etc.)
- Timeline-style scheduling view
- Drag/drop adjustments
- One-time edits vs full-series edits
- Visual differentiation for self-care tasks

### 📅 Planner Mode

- Daily/weekly calendar views
- Integration with Kanban tasks (optional)
- Timeline interface or block scheduler

---

## 🧩 Technical Overview

- Platform: Web (initial), Desktop (future)
- Hosting: DigitalOcean
- Containerized: Docker (likely)
- Frontend: TBD (React, Vue, or other)
- Backend: TBD (Laravel, Node, etc.)

---

## 🔮 Future Features / Wishlist

- [ ] Gamification (streaks, rewards)
- [ ] “Focus mode” with simplified UI
- [ ] Distraction blocker or soft lockout timer
- [ ] Daily review / rescheduling helper
- [ ] AI-generated summaries or nudges
- [ ] Calendar sync (Google, iCal, etc.)
- [ ] Export to CSV / JSON
- [ ] Mobile app (post-launch)

---

## 🎯 Design Philosophy

- Designed with ADHD and executive dysfunction in mind
- Minimize cognitive load
- Clear visual organization (color, motion, category)
- Low-friction entry points for new thoughts/tasks
- Flexible enough for chaos, structured enough for routine
- Extendable either through future development or an open framework to allow others to add functions/capeabilies.

---

## 🗓️ Development Status

- [ ] Planning
- [ ] Wireframes
- [ ] Database schema
- [ ] Task system prototype
- [ ] Routine scheduler prototype
- [ ] MVP build

## Technology Stack

- Tailwind with flexbox to govern layout/responsiveness.
- Laravel
- PostgreSQL for back end database
- Redis or laravel Octana (sesson and caching)
- Websocket for real-time updates (Laravel Echo)
- Pusher (or Socketi) for live board updates.
- API resources for lean structured data to front end.
- Lazy Loading and client-side hydration for heavy UI's
- Combine localStorage and backend sync for user preferences/state storage/retrieval.
- Docker for containerization
- OpenAi GPT-4 Turbo
- Twilio- SMS and messaging/ user input
- Mailgun/Sendgrid- email parsing
- Laravel HTTP Client _call the openAI API
- Laravel Sancturm - To authenticae API calls
- Job Queue- process input sync.

## Hosting Environment

📦 Managed PostgreSQL (DigitalOcean DB)
📦 Managed Redis (optional) OR Redis container in main app
🌐 Nginx + Laravel App (Docker container)
📮 Laravel Queue Worker (separate Docker container)
🧠 AI Processing Worker (optional container/service)
📶 Laravel Horizon (optional, for managing queues)
🔁 Load Balancer (DO-native or Traefik/HAProxy)
🔐 HTTPS via Certbot (if self-hosted Nginx)

### Hosting notes (additonal)
- Queue Worker shouldb ein a searate container for scale
- Laravel Horizon for queue management
- Laravel Queue Worker for processing incoming tasks
- AI Processing Worker for handling complex inputs
- Load balancing and horizontal scaling options available as needed.
---

## 🤖 AI Integration

The ADHD Planner will include **natural language task input** via AI to reduce friction and support neurodivergent workflows. This feature enables users to create, update, or manage tasks using plain language — via the app, SMS, or email.

### 🧠 Purpose

Instead of navigating UI manually, users can type or say things like:
- “Add ‘Buy dog food’ to Home project, due Saturday”
- “Move ‘Finish report’ to Done”
- “Remind me to stretch every day at 9am”

This reduces cognitive load and makes the tool feel more like an executive assistant than a traditional planner.

---

### 🔁 Use Cases

- **Smart Add Input (Web)**: A textbox that accepts natural task descriptions and creates structured tasks automatically.
- **Inbox via SMS or Email**: Send a text or email and have it create tasks in the user’s account (using NLP + backend API).
- **Recurring Task Suggestions**: AI helps structure repeating tasks from natural phrases (“every Friday”).
- **Daily Summary Prompting** *(future)*: “What should I focus on today?” returns priority tasks, suggestions, or nudges.
- **Gentle prompts**: “You haven’t updated ‘Health’ in a while — want to add something?”

---

### ⚙️ Technical Architecture

#### 🔧 Input Methods
- **Web input field**
- **SMS integration** (via [Twilio](https://www.twilio.com/))
- **Email-to-task inbox** (via [Mailgun](https://www.mailgun.com/) or [SendGrid Inbound Parse](https://docs.sendgrid.com/for-developers/parsing-email/setting-up-the-inbound-parse-webhook))

All input is routed to the same processing endpoint:
```
POST /api/incoming-task
```

---

### 🧠 OpenAI Integration (Function Calling)

The input is sent to **OpenAI GPT-4 Turbo** using [function calling](https://platform.openai.com/docs/guides/function-calling) to produce structured task data.

**Function Definition Example**:
```json
{
  "name": "createTask",
  "description": "Create a new task on the user's board",
  "parameters": {
    "type": "object",
    "properties": {
      "taskName": { "type": "string" },
      "project": { "type": "string" },
      "boardState": { "type": "string" },
      "dueDate": { "type": "string", "format": "date" },
      "priority": { "type": "string", "enum": ["low", "normal", "high"] }
    },
    "required": ["taskName"]
  }
}
```

**###AI Output**:
```json
{
  "taskName": "Schedule car inspection",
  "project": "Life Admin",
  "boardState": "To Do",
  "dueDate": "2025-05-10",
  "priority": "normal"
}
```

---

## 🧰 Laravel Backend Integration

##### 📥 Incoming Task Controller (example stub)
```php
// routes/api.php
Route::post('/incoming-task', [AIController::class, 'handleIncoming'])->middleware('auth:sanctum');

// app/Http/Controllers/AIController.php
public function handleIncoming(Request $request)
{
    $user = $request->user();
    $inputText = $request->input('input_text');

    // 1. Send text to OpenAI API with function calling
    $structured = app(OpenAIAssistant::class)->parseInput($inputText);

    // 2. Match or create related models
    $project = Project::firstOrCreate(['name' => $structured['project'], 'user_id' => $user->id]);
    $boardState = BoardState::where('name', $structured['boardState'])->first();

    // 3. Create the task
    $task = Task::create([
        'user_id' => $user->id,
        'project_id' => $project->id,
        'name' => $structured['taskName'],
        'due_date' => $structured['dueDate'],
        'board_state_id' => $boardState->id,
        'priority' => $structured['priority'],
    ]);

    return response()->json(['status' => 'success', 'task' => $task]);
}
```

---

### 📦 Stack Requirements (Additional)

| Component | Description |
|----------|-------------|
| **OpenAI GPT-4 Turbo** | For parsing user input into structured task data |
| **Twilio (optional)** | Receive SMS input from users |
| **Mailgun / SendGrid** | Inbound email parsing (create tasks via email) |
| **Laravel HTTP Client** | To call the OpenAI API |
| **Laravel Sanctum** | To authenticate API calls securely |
| **Job Queue** (optional) | For processing input async if needed later |

---

### ✅ Example Prompt

> “Add ‘Schedule therapy session’ to Health project in Waiting. Make it high priority and due next Thursday.”

**Result**:
- Project: Health
- Task: Schedule therapy session
- Priority: High
- State: Waiting
- Due: next Thursday

---

###Additonal Docker/Enviornment Setup
**Heck yes.** You’re thinking about this like a future-scaling pro — build it to run lean now, but smart enough to scale horizontally without re-architecting later.

Here’s a full-blown **production-ready hosting environment setup** using:

✅ Docker  
✅ DigitalOcean Droplet  
✅ Managed PostgreSQL  
✅ Load balancer and container scaling compatibility  
✅ Laravel queue support  
✅ Redis cache + session handling  
✅ AI/Webhook flexibility  
✅ Let’s Encrypt SSL  

---

## 🧱 PRODUCTION HOSTING ARCHITECTURE (Docker + DO)

```plaintext
📦 Managed PostgreSQL (DigitalOcean DB)
📦 Managed Redis (optional) OR Redis container in main app
🌐 Nginx + Laravel App (Docker container)
📮 Laravel Queue Worker (separate Docker container)
🧠 AI Processing Worker (optional container/service)
📶 Laravel Horizon (optional, for managing queues)
🔁 Load Balancer (DO-native or Traefik/HAProxy)
🔐 HTTPS via Certbot (if self-hosted Nginx)
```

---

## 🧰 Docker-Based Project Structure

### `docker-compose.yml` (simplified, scalable-ready)

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: adhdplanner-app
    restart: unless-stopped
    env_file: .env
    ports:
      - "80:80"
    depends_on:
      - redis
    volumes:
      - .:/var/www/html
    networks:
      - backend

  queue:
    build:
      context: .
    container_name: adhdplanner-queue
    command: php artisan queue:work --tries=3 --timeout=60
    depends_on:
      - app
    env_file: .env
    volumes:
      - .:/var/www/html
    networks:
      - backend

  redis:
    image: redis:alpine
    container_name: adhdplanner-redis
    restart: unless-stopped
    networks:
      - backend

networks:
  backend:
    driver: bridge
```

---

### 🔐 `.env` (Key Configs)

```env
APP_ENV=production
APP_KEY=base64:...
APP_DEBUG=false
APP_URL=https://yourdomain.com

DB_CONNECTION=pgsql
DB_HOST=your-managed-db-do.internal
DB_PORT=25060
DB_DATABASE=adhd_planner
DB_USERNAME=yourusername
DB_PASSWORD=yourpassword

REDIS_HOST=redis
REDIS_PORT=6379

QUEUE_CONNECTION=redis
```

---

### 🛡️ Nginx Config (Inside App Container or Host)

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    root /var/www/html/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## ☁️ DigitalOcean Setup

### 🖥️ Droplet
- **Size**: 2vCPU / 4–8GB RAM to start
- **OS**: Ubuntu 22.04 LTS
- **Docker / Docker Compose** installed
- **Swap file** enabled (for queue + caching headroom)

### 🗄️ Managed DB (PostgreSQL)
- Select “Managed Database” in DO
- Add your droplet’s private IP to the trusted sources
- Use connection string in your `.env`

### 🧠 Redis (Pick one)
- **Option A**: Let the Redis container run inside Docker (simpler, fast)
- **Option B**: Use DigitalOcean’s **Managed Redis** if you expect high session or queue load

---

## 🔄 Load Balancing + Scaling

### For Scaling App Containers
- Add more `app` containers under a common `docker-compose.scale.yml`
- Or split into swarm/k8s later (optional — not needed early)

### For Load Balancing
- Use DigitalOcean’s native Load Balancer (HTTPS termination + sticky sessions)
- OR use **Traefik**/HAProxy in front of your containers (more control)

---

## 🧠 Extra Considerations

| Component | Tool | Notes |
|----------|------|-------|
| HTTPS | Certbot (Let's Encrypt) | Inside Nginx or Traefik |
| Auto Deploy | GitHub Actions or Laravel Forge | Optional |
| Logging | Laravel Log → File or external | Use `docker logs` or mount volume |
| Health Checks | Laravel `up` route or `/health` | Useful for load balancer pings |
| Monitoring | Laravel Telescope (local) or Sentry/LogRocket | Optional |
| Mail | Mailgun/SendGrid | For password resets, task confirmations |
| AI API | OpenAI or local model | Called from Laravel worker queue |

---

## ✅ TL;DR: Final Stack Checklist

| Component | Where/How |
|----------|-----------|
| Laravel App | Docker container (Nginx + PHP-FPM) |
| Queue Worker | Separate container (`php artisan queue:work`) |
| Redis | Container or DO-managed |
| PostgreSQL | DO-managed |
| AI/API logic | Laravel services using OpenAI |
| Load Balancer | DO native LB or Traefik |
| HTTPS | Certbot or DO LB |
| Deployment | GitHub Actions or manual via SSH/Docker |
| Scaling | Add containers → connect to LB → done |

---

Added 4/20/25

## ADHDBoard — Environment Setup (Node + Docker + PowerShell)

### ✅ Tech Stack:
- **Node.js + Express** for the backend
- **MySQL** as the database
- **Sequelize ORM**
- **Docker + Docker Compose**
- **PowerShell scripts for safety**
- Developed and run on **Windows**, Docker Desktop

---

### 📁 Folder Structure

```
adhdboard/
├── backend/
│   ├── Dockerfile
│   ├── .env
│   ├── package.json
│   ├── .sequelizerc
│   ├── config/
│   │   └── config.js
│   ├── models/
│   │   ├── index.js
│   │   └── user.js
│   ├── migrations/
│   │   └── xxxx-create-user.js
│   └── src/
│       └── app.js
├── db/
│   └── schema.sql
├── scripts/
│   ├── safe-up.ps1
│   ├── db-backup.ps1
│   └── db-restore.ps1
└── docker-compose.yml
```

---

### ⚙️ .env (inside `/backend`)
```env
DB_HOST=db
DB_USER=adhd_user
DB_PASS=adhd_pass
DB_NAME=adhdboard
```

---

### ⚙️ .sequelizerc
```js
require('dotenv').config();

const path = require('path');

module.exports = {
  'config': path.resolve('config', 'config.js'),
  'models-path': path.resolve('models'),
  'seeders-path': path.resolve('seeders'),
  'migrations-path': path.resolve('migrations')
};
```

---

### ⚙️ config/config.js
```js
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'mysql'
  }
};
```

---

### 💣 docker-compose.yml (relevant parts)
```yaml
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - DB_HOST=db
      - DB_USER=adhd_user
      - DB_PASS=adhd_pass
      - DB_NAME=adhdboard
    depends_on:
      - db

  db:
    image: mysql:8
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: adhdboard
      MYSQL_USER: adhd_user
      MYSQL_PASSWORD: adhd_pass
    volumes:
      - db_data:/var/lib/mysql
      - ./db/schema.sql:/docker-entrypoint-initdb.d/schema.sql

volumes:
  db_data:
```

---

### 🛡 PowerShell Scripts

#### scripts/safe-up.ps1
```powershell
Write-Host "🚀 Safely starting ADHDBoard in detached mode..."
docker-compose up -d --build
```

#### scripts/db-backup.ps1
```powershell
$timestamp = Get-Date -Format "yyyy-MM-dd_HH-mm"
$backupFile = "db_backup_$timestamp.tar.gz"

docker run --rm `
  -v db_data:/volume `
  -v "${PWD}:/backup" `
  alpine `
  tar -czf "/backup/$backupFile" -C /volume .
```

#### scripts/db-restore.ps1
```powershell
param ([string]$BackupFile)

if (-not $BackupFile) {
  Write-Host "❗ Usage: .\db-restore.ps1 <backup-file.tar.gz>"
  exit 1
}

docker run --rm `
  -v db_data:/volume `
  -v "${PWD}:/backup" `
  alpine `
  sh -c "rm -rf /volume/* && tar -xzf /backup/$BackupFile -C /volume"
```

---

### 🤔 Sequelize CLI Inside Docker
Run CLI tools inside the container so `DB_HOST=db` works:
```powershell
docker exec -it adhdplannerdev-backend-1 npx sequelize-cli model:generate --name Project --attributes name:string
docker exec -it adhdplannerdev-backend-1 npx sequelize-cli db:migrate
```

---

### ✅ /users Route Test (in `src/app.js`)
```js
app.get('/users', async (req, res) => {
  const users = await User.findAll();
  res.json(users);
});
```

✅ Dev Strategy: “Scaffold Now, Bloom Later”
🔧 What to Include for MVP (Scaffolding)
Add a notes table to the DB schema (even if unused at first)

Create a basic internal API route like POST /capture and GET /capture

Add a placeholder “Capture” page or tab in the UI (“Coming soon!” or a basic note entry box)

Treat note.id as an optional foreign key in future task/project tables

Where to Track It
✅ You’re adding it to the project planning doc ✅

✅ You’ve logged it in OneNote ✅

I’ve also got it locked in memory (and bio)

## Feature Spec: Note Capture Zone (ADHDBoard)

### Overview
The Note Capture Zone is a dedicated feature designed to act as a mental inbox for users to quickly offload thoughts, ideas, tasks, links, files, and references. Inspired by Google Keep, OneNote, and Getting Things Done (GTD), it enables low-friction capture and high-flexibility sorting.

This feature will be included in the MVP scaffolding but implemented in a later phase.

---

### Goals
- Reduce cognitive load for users with ADHD by externalizing memory
- Allow for fast, unstructured capture of any type of input
- Enable sorting and assignment of notes later (to tasks, projects, calendar items)
- Keep the system gentle, forgiving, and flexible

---

### Core MVP Fields for `notes` Table
```sql
CREATE TABLE notes (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  title VARCHAR(255),
  content TEXT,
  image_url TEXT,
  audio_url TEXT,
  is_archived BOOLEAN DEFAULT FALSE,
  linked_task_id INT,
  linked_project_id INT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (linked_task_id) REFERENCES tasks(id),
  FOREIGN KEY (linked_project_id) REFERENCES projects(id)
);
```

---

### Phase 1 Features (Scaffolding Enabled)
- Basic `POST /capture` and `GET /capture` endpoints
- Frontend placeholder UI or tab with a basic note input form
- All new notes default to `is_archived = false`, `linked_* = NULL`
- No strict categorization required

---

### Phase 2 Features (Full Implementation)
- Attach captured notes to existing tasks, projects, or calendar entries
- Allow file/image/audio upload
- Add tags and filtering options
- Enable reminders or follow-up prompts
- Allow color coding, pinning, and archiving
- Support capture via email/SMS input (future API integrations)

---

### Long-Term Stretch Ideas
- Mobile-first interface or quick-add widget
- Browser extension or desktop clipper
- Transcription of voice/audio notes
- Daily/weekly "unsorted notes" review prompt
- Auto-suggestion of linking ("This note mentions Project A")

---

### UX Notes
- Notes should be shown in a low-pressure interface (no deadlines or overdue labels)
- Display in card-style layout similar to Google Keep or Apple Notes
- First-time use can be blank canvas or fun placeholder note ("Drop your thoughts here")

---

### Dev Notes
- Minimal backend footprint
- Should not require note categorization or link at creation time
- Indexed by `user_id` and `created_at` for future dashboard integrations

---

### Status
✅ Planned and included in database scaffolding
❌ Not yet implemented in MVP UI or API

---

**Author:** ADHDBoard Planning Team  
**Status:** Scaffolded for future expansion  
**Last Updated:** 2025-04-20

Feature Spec: "Clarity" AI Companion (ADHDBoard)

Overview

"Clarity" is a planned AI-powered assistant designed to support users with ADHD by learning from their behavior and providing gentle, adaptive suggestions to support habit building, task management, and idea capture. It will be added as a future module after the MVP, but its structure will be scaffolded early.

Conceptual Role

🫠 External cognitive aid for users with executive function challenges

👥 Learns passively from patterns, not through interrogation

🕵️ Acts like a guide, not a manager

🤝 Offers help without pressure, judgment, or shame

Potential Features (Roadmapped by Phase)

⛏ Phase 1: Passive Intelligence (MVP-adjacent)

Auto-tagging notes based on content using OpenAI API

Natural language to task conversion ("Pay rent Friday" → task with due date)

Suggesting project or task categories when saving notes

🔄 Phase 2: Pattern Recognition & Nudging

Track task completion times/types (when and what gets done)

Identify trends (e.g., user prefers working late, avoids certain task types)

Weekly summaries ("You completed 4 tasks this week, mostly after 5pm")

Nudge system: "Want to prep for your usual Thursday focus session?"

🌱 Phase 3: Adaptive Habit Support

Suggests routines and repeating tasks based on user patterns

Energy-based suggestions: "This looks like a high-focus task. Try it when you're in the zone?"

AI-generated reflections ("Want to review your captured ideas this weekend?")

Smart suggestion system for linking notes to projects/tasks

Ethical & UX Principles

✉️ Always ask permission before AI-driven behavior suggestions

❌ Never penalize or shame inconsistent behavior

🔐 Ensure user privacy by limiting data scope or keeping AI local when possible

✨ Always frame AI as supportive, optional, and opt-in

Technical Considerations

Likely requires usage metrics: task timestamps, capture frequency, etc.

Initial implementation can rely on OpenAI or similar external LLM APIs

Future-proofing for potential local LLM deployment

Store in separate ai_assistant_logs or usage_analytics table (optional)

UI Considerations

Lives in sidebar panel, floating assistant button, or collapsible drawer

Soft voice or visual tone: friendly, neutral, non-intrusive

Interactions feel more like a journal prompt than a command

Database Scaffolding Ideas (Optional for Now)

CREATE TABLE ai_suggestions (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  suggestion_type VARCHAR(100),
  suggestion_text TEXT,
  related_note_id INT,
  related_task_id INT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  viewed BOOLEAN DEFAULT FALSE,
  accepted BOOLEAN DEFAULT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

Status

✅ Concept scoped and philosophically aligned
✅ Future scaffolding planned
❌ Not included in MVP functionality

Author: ADHDBoard Planning TeamStatus: Deferred Feature, Scaffold ReadyLast Updated: 2025-04-20
