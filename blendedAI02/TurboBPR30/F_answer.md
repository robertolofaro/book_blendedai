Our **Back-end Developer** and **PC-based Expert** have designed this schema to be "locally-first." In the 90s, the original TurboBPR used a flat-file approach; we are upgrading this to a relational **SQLite** structure. 

This ensures that all data remains on the user's machine (maintaining DoD-level privacy) while allowing the **AI Engine** to perform complex cross-references between the "Current" and "Future" states.



---

### 🗄️ TurboBPR 3.0 SQLite Schema

We’ve structured this to support **Versioning**. A "Project" can have multiple "Process Models" (e.g., one AS-IS and several TO-BE experiments).

```sql
-- 1. Project Container
CREATE TABLE projects (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_modified DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 2. Process Models (AS-IS vs TO-BE)
CREATE TABLE processes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    project_id INTEGER,
    name TEXT NOT NULL,
    version_type TEXT CHECK(version_type IN ('AS-IS', 'TO-BE')),
    total_cycle_time_mins INTEGER DEFAULT 0,
    total_cost REAL DEFAULT 0.0,
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
);

-- 3. Individual Process Steps
CREATE TABLE steps (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    process_id INTEGER,
    sequence_order INTEGER NOT NULL,
    description TEXT,
    actor TEXT, -- e.g., "Department Head", "System", "Vendor"
    duration_mins INTEGER,
    is_value_added BOOLEAN DEFAULT 1,
    is_bottleneck BOOLEAN DEFAULT 0,
    FOREIGN KEY (process_id) REFERENCES processes(id) ON DELETE CASCADE
);

-- 4. AI Insights & Recommendations (Ollama Output)
CREATE TABLE ai_insights (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    process_id INTEGER,
    raw_ollama_output TEXT,
    summary TEXT,
    efficiency_score INTEGER, -- 1 to 100
    generated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (process_id) REFERENCES processes(id) ON DELETE CASCADE
);
```

---

### 🧠 The Experts' Rationales

* **The BPR Expert:** "Notice the `is_value_added` flag in the `steps` table. This is the heart of BPR. By storing this, we can instantly generate a 'Waste vs. Value' chart for the user, showing them exactly where the bureaucracy is hiding."
* **The Ergonomics Expert:** "We’ve included `sequence_order`. In the Web UI, this will allow users to **drag-and-drop** steps to reorder them, and the database will update in real-time. Much smoother than the old 'Insert Row' menus of the 90s."
* **The PC-based Expert:** "Using SQLite means the entire project is just one `.db` file. Users can email their 'Procurement_Study.db' to a colleague, and they can open it instantly in their own TurboBPR 3.0 instance. No server setup required."
* **The Business/Marketing Expert:** "By storing `efficiency_score` in the `ai_insights` table, we can track the 'ROI of Re-engineering' over time. This data is gold for presenting to stakeholders."

---

### 📈 What this enables
With this schema, we can now run a **Delta Query**. We can compare an AS-IS process with a TO-BE process and calculate the **"Turbo Gap"**:
* **Time Saved:** (AS-IS Time - TO-BE Time)
* **Cost Avoided:** (AS-IS Cost - TO-BE Cost)
* **Complexity Reduction:** (AS-IS Steps - TO-BE Steps)

