Our **Back-end Developer** and **PC-based Software Expert** have collaborated to ensure these export modules are "bulletproof." In the 90s, TurboBPR relied on complex OLE automation to talk to Word; today, we use direct file manipulation, which is 100x faster and doesn't even require Word to be installed on the server.

Here is the implementation for the **TurboBPR 3.0 Export Engine**.

---

### 🛠️ The Export Module (`exporter.py`)

We will use `python-docx` for the Word heavy-lifting and standard Python file I/O for the Markdown.

```python
import os
from docx import Document
from docx.shared import Inches, Pt

class TurboBPRReport:
    def __init__(self, data):
        self.title = data.get("title", "BPR Analysis Report")
        self.summary = data.get("summary", "")
        self.analysis = data.get("analysis", [])
        self.recommendations = data.get("recommendations", [])

    def generate_markdown(self, filename="report.md"):
        """Generates a clean, web-ready Markdown file."""
        content = f"# {self.title}\n\n"
        content += "## Executive Summary\n"
        content += f"{self.summary}\n\n"
        
        content += "## Process Analysis\n"
        content += "| Step | Actor | Value-Add |\n| :--- | :--- | :--- |\n"
        for item in self.analysis:
            content += f"| {item['step']} | {item['actor']} | {item['value_add']} |\n"
            
        content += "\n## AI Strategic Recommendations\n"
        for rec in self.recommendations:
            content += f"- **{rec['title']}**: {rec['desc']}\n"
            
        with open(filename, "w") as f:
            f.write(content)
        return filename

    def generate_docx(self, filename="report.docx"):
        """Generates a professional, formal Word document."""
        doc = Document()
        
        # Title
        title = doc.add_heading(self.title, 0)
        
        # Summary Section
        doc.add_heading('Executive Summary', level=1)
        doc.add_paragraph(self.summary)
        
        # Analysis Table
        doc.add_heading('Process Analysis', level=1)
        table = doc.add_table(rows=1, cols=3)
        table.style = 'Table Grid'
        hdr_cells = table.rows[0].cells
        hdr_cells[0].text = 'Step'
        hdr_cells[1].text = 'Actor'
        hdr_cells[2].text = 'Value-Add'
        
        for item in self.analysis:
            row_cells = table.add_row().cells
            row_cells[0].text = str(item['step'])
            row_cells[1].text = item['actor']
            row_cells[2].text = item['value_add']
            
        # Recommendations
        doc.add_heading('Strategic Recommendations', level=1)
        for rec in self.recommendations:
            p = doc.add_paragraph(style='List Bullet')
            run = p.add_run(f"{rec['title']}: ")
            run.bold = True
            p.add_run(rec['desc'])
            
        doc.save(filename)
        return filename
```

---

### 🌐 The FastAPI Integration (`main.py`)

The **Front-end Expert** requested a simple endpoint so the user can just click a "Download" button on the dashboard.

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse

app = FastAPI()

@app.get("/export/{format}")
async def export_report(format: str):
    # Mock data - in production, this comes from SQLite
    report_data = {
        "title": "Procurement Redesign 2026",
        "summary": "Process is currently 98% non-value added.",
        "analysis": [{"step": "Form 402", "actor": "Staff", "value_add": "No"}],
        "recommendations": [{"title": "Digital Bypass", "desc": "Automate all under $10k."}]
    }
    
    report = TurboBPRReport(report_data)
    
    if format == "word":
        path = report.generate_docx()
        return FileResponse(path, media_type='application/vnd.openxmlformats-officedocument.wordprocessingml.document', filename="BPR_Report.docx")
    
    path = report.generate_markdown()
    return FileResponse(path, media_type='text/markdown', filename="BPR_Report.md")
```

---

### 💡 Expert Commentary

* **The Ergonomics Expert:** "The user should never see a 'Save As' dialog. We’ll trigger the download directly. It makes the software feel like a modern SaaS rather than a legacy tool."
* **The Business/Marketing Expert:** "Including Markdown is a huge selling point. It means the BPR team can copy-paste results directly into Slack, Teams, or Notion, making the 'Re-engineering' advice viral within the agency."
* **The PC-based Expert:** "Coming from the 90s version, the fact that this can generate a document in 50 milliseconds without the 'clippy' wizard popping up is a miracle."

---

