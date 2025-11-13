# PRISM Gradio UI v2
## Web Interface for Research Synthesis

**Purpose:** User-friendly interface for uploading data and viewing analysis

**File:** `gradio_interface.py` or part of `THE_AI_TEMPLATE.ipynb`

---

## Complete Implementation

```python
import gradio as gr
from prism_brain_v2 import PRISMBrainV2
import json

# ============================================================
# INITIALIZE PRISM BRAIN
# ============================================================

# You'll need to set your FigJam token here
FIGMA_TOKEN = "figd_YOUR_TOKEN_HERE"

brain = PRISMBrainV2(figma_token=FIGMA_TOKEN)

# Store current project ID globally
current_project_id = None

# ============================================================
# INTERFACE FUNCTIONS
# ============================================================

def create_new_project(project_name):
    """Create a new research project"""
    global current_project_id
    
    if not project_name:
        return "❌ Please enter a project name", ""
    
    current_project_id = brain.create_project(project_name)
    
    return f"✅ Created project: {project_name}", current_project_id

def upload_figjam(figjam_url):
    """Process FigJam board URL"""
    global current_project_id
    
    if not current_project_id:
        return "❌ Please create a project first"
    
    if not figjam_url:
        return "❌ Please enter a FigJam URL"
    
    result = brain.ingest_figjam_url(current_project_id, figjam_url)
    
    if 'error' in result:
        return f"❌ Error: {result['error']}"
    else:
        return f"✅ Processed FigJam board: {result['board_name']}\n   Extracted {result['notes']} items"

def upload_audio(audio_file):
    """Process audio file"""
    global current_project_id
    
    if not current_project_id:
        return "❌ Please create a project first"
    
    if audio_file is None:
        return "❌ Please upload an audio file"
    
    result = brain.ingest_audio_file(current_project_id, audio_file.name)
    
    if 'error' in result:
        return f"❌ Error: {result['error']}"
    else:
        return f"✅ Processed audio file\n   Extracted {result['notes']} insights\n   Transcript length: {result['transcript_length']} characters"

def upload_document(doc_file):
    """Process document file"""
    global current_project_id
    
    if not current_project_id:
        return "❌ Please create a project first"
    
    if doc_file is None:
        return "❌ Please upload a document"
    
    result = brain.ingest_document_file(current_project_id, doc_file.name)
    
    if 'error' in result:
        return f"❌ Error: {result['error']}"
    else:
        return f"✅ Processed document\n   Extracted {result['notes']} insights\n   Pages: {result['pages']}"

def analyze_project():
    """Generate synthesis report"""
    global current_project_id
    
    if not current_project_id:
        return "❌ Please create a project first"
    
    synthesis = brain.synthesize_project(current_project_id)
    
    if 'error' in synthesis:
        return f"❌ Error: {synthesis['error']}"
    
    # Format synthesis as readable text
    report = f"""
# 📊 PRISM SYNTHESIS REPORT

## Project: {synthesis['project_name']}
**Last Updated:** {synthesis['last_updated']}

---

## 📈 Overview

- **Total Notes:** {synthesis['total_notes']}
- **Sources:** {synthesis['total_sources']}
- **Contributors:** {synthesis['contributors']}

---

## 📑 Content Distribution

"""
    
    for content_type, count in synthesis['by_type'].items():
        report += f"- **{content_type.replace('_', ' ').title()}:** {count}\n"
    
    report += f"""

---

## 🎯 Priority Breakdown

- **High Priority:** {synthesis['by_priority']['high']} items
- **Medium Priority:** {synthesis['by_priority']['medium']} items
- **Low Priority:** {synthesis['by_priority']['low']} items

---

## 💡 Key Themes

"""
    
    for theme in synthesis['themes'][:5]:
        report += f"- **{theme['theme']}** (mentioned {theme['frequency']} times)\n"
    
    report += f"""

---

## ⚠️ Action Items

"""
    
    for i, action in enumerate(synthesis['action_items'][:5], 1):
        report += f"{i}. {action.get('content', 'N/A')} (Priority: {action.get('priority', 0):.2f})\n"
    
    report += f"""

---

## 📊 Sentiment Analysis

- **Positive:** {synthesis['stats']['sentiment_distribution']['positive']} notes
- **Negative:** {synthesis['stats']['sentiment_distribution']['negative']} notes
- **Neutral:** {synthesis['stats']['sentiment_distribution']['neutral']} notes

---

## 👥 Contributors

"""
    
    for contributor, data in list(synthesis['by_contributor'].items())[:5]:
        report += f"\n### {contributor}\n"
        report += f"- **Total Contributions:** {data['count']}\n"
        report += "- **By Type:**\n"
        for note_type, count in data['types'].items():
            report += f"  - {note_type.replace('_', ' ').title()}: {count}\n"
    
    report += f"""

---

## 📅 Timeline

"""
    
    for event in synthesis['timeline']:
        report += f"- **{event['date'][:10]}:** {event['event']} ({event['note_count']} items)\n"
    
    return report

def refresh_analysis():
    """Refresh the analysis"""
    return analyze_project()

# ============================================================
# GRADIO INTERFACE
# ============================================================

with gr.Blocks(title="PRISM Research Synthesis Engine") as interface:
    
    gr.Markdown("""
    # 🔬 PRISM Research Synthesis Engine
    **Pattern Recognition & Insight Structure Module**
    
    Upload FigJam boards, audio recordings, and documents to generate organized research insights.
    """)
    
    # Section 1: Project Setup
    with gr.Row():
        with gr.Column():
            gr.Markdown("## 1. Create Project")
            project_name_input = gr.Textbox(
                label="Project Name",
                placeholder="e.g., Mobile App Redesign Research",
                lines=1
            )
            create_project_btn = gr.Button("Create Project", variant="primary")
            project_status = gr.Textbox(label="Status", lines=2)
            project_id_display = gr.Textbox(label="Project ID", lines=1, interactive=False)
    
    create_project_btn.click(
        fn=create_new_project,
        inputs=[project_name_input],
        outputs=[project_status, project_id_display]
    )
    
    gr.Markdown("---")
    
    # Section 2: Data Upload
    gr.Markdown("## 2. Upload Data Sources")
    
    with gr.Tabs():
        # Tab 1: FigJam
        with gr.Tab("📌 FigJam Board"):
            figjam_url_input = gr.Textbox(
                label="FigJam Board URL",
                placeholder="https://www.figma.com/file/...",
                lines=1
            )
            figjam_btn = gr.Button("Process FigJam Board", variant="primary")
            figjam_status = gr.Textbox(label="Status", lines=3)
            
            gr.Markdown("""
            **Note:** Analyzes full board including:
            - Sticky notes
            - Text boxes
            - Arrows and connections
            - Diagrams and shapes
            """)
        
        figjam_btn.click(
            fn=upload_figjam,
            inputs=[figjam_url_input],
            outputs=[figjam_status]
        )
        
        # Tab 2: Audio
        with gr.Tab("🎤 Audio File"):
            audio_file_input = gr.File(
                label="Upload Audio File",
                file_types=[".mp3", ".wav", ".m4a", ".mov"],
                type="filepath"
            )
            audio_btn = gr.Button("Process Audio", variant="primary")
            audio_status = gr.Textbox(label="Status", lines=3)
            
            gr.Markdown("""
            **Supported formats:** MP3, WAV, M4A, MOV
            
            **Processing includes:**
            - Whisper AI transcription
            - Tone detection (emphatic, questioning, hesitant, etc.)
            - Organized insight extraction
            - Speaker pattern analysis
            """)
        
        audio_btn.click(
            fn=upload_audio,
            inputs=[audio_file_input],
            outputs=[audio_status]
        )
        
        # Tab 3: Documents
        with gr.Tab("📄 Document"):
            doc_file_input = gr.File(
                label="Upload Document",
                file_types=[".pdf", ".pptx", ".ppt", ".txt", ".docx"],
                type="filepath"
            )
            doc_btn = gr.Button("Process Document", variant="primary")
            doc_status = gr.Textbox(label="Status", lines=3)
            
            gr.Markdown("""
            **Supported formats:** PDF, PowerPoint, TXT, DOCX
            
            **Processing includes:**
            - Text extraction
            - Paragraph analysis
            - Key insight identification
            - Structure recognition
            """)
        
        doc_btn.click(
            fn=upload_document,
            inputs=[doc_file_input],
            outputs=[doc_status]
        )
    
    gr.Markdown("---")
    
    # Section 3: Analysis
    gr.Markdown("## 3. Generate Analysis")
    
    with gr.Row():
        analyze_btn = gr.Button("🔍 Analyze Project", variant="primary", scale=2)
        refresh_btn = gr.Button("🔄 Refresh Analysis", scale=1)
    
    analysis_output = gr.Markdown(
        label="Synthesis Report",
        value="Click 'Analyze Project' to generate synthesis report"
    )
    
    analyze_btn.click(
        fn=analyze_project,
        outputs=[analysis_output]
    )
    
    refresh_btn.click(
        fn=refresh_analysis,
        outputs=[analysis_output]
    )
    
    gr.Markdown("---")
    
    gr.Markdown("""
    ## 📚 How to Use PRISM
    
    1. **Create a Project:** Enter a descriptive name for your research project
    2. **Upload Data:** Add FigJam boards, audio recordings, and/or documents
    3. **Analyze:** Click "Analyze Project" to generate comprehensive insights
    4. **Refresh:** Use "Refresh Analysis" after adding more data sources
    
    ### 🎯 What PRISM Does
    
    - Organizes research data by content type (pain points, questions, insights, ideas, action items, quotes)
    - Calculates priority levels for each item
    - Tracks contributors and timeline
    - Identifies recurring themes
    - Analyzes sentiment (positive, negative, neutral)
    - Extracts actionable items
    - Generates comprehensive synthesis reports
    
    ### 🔐 Privacy & Security
    
    - Your FigJam token is required for API access
    - All processing happens locally
    - No data is sent to external servers (except Figma API)
    """)

# ============================================================
# LAUNCH INTERFACE
# ============================================================

if __name__ == "__main__":
    interface.launch(
        server_name="0.0.0.0",  # Allow external access
        server_port=7860,
        share=False  # Set to True for public link
    )
```

---

## Installation Requirements

```bash
pip install gradio
pip install openai-whisper
pip install PyPDF2
pip install python-pptx
pip install requests
pip install numpy
```

---

## Configuration

Before running, set your FigJam token:

```python
FIGMA_TOKEN = "figd_YOUR_ACTUAL_TOKEN_HERE"
```

Get your token from: https://www.figma.com/developers/api#access-tokens

---

## Launch Commands

```bash
# Standard launch
python gradio_interface.py

# Public sharing (generates shareable link)
python gradio_interface.py --share

# Custom port
python gradio_interface.py --port 7861
```

---

## Interface Sections

### 1. Project Creation
- Enter project name
- Creates unique project ID
- Initializes project workspace

### 2. Data Upload (3 Tabs)

**📌 FigJam Board**
- Paste FigJam URL
- Analyzes: stickies, text, arrows, connections, diagrams
- Auto-extracts board name

**🎤 Audio File**
- Upload: .mp3, .wav, .m4a, .mov
- Whisper transcription
- Tone detection
- Organized insights

**📄 Document**
- Upload: .pdf, .pptx, .ppt, .txt, .docx
- Text extraction
- Paragraph analysis
- Key insights

### 3. Analysis Generation
- **Analyze Project:** Full synthesis report
- **Refresh Analysis:** Update with new data
- Comprehensive markdown-formatted output

---

## Output Format

The synthesis report includes:

1. **Overview:** Total notes, sources, contributors
2. **Content Distribution:** Breakdown by type
3. **Priority Levels:** High/medium/low breakdown
4. **Key Themes:** Most frequent topics
5. **Action Items:** Top priority tasks
6. **Sentiment Analysis:** Positive/negative/neutral distribution
7. **Contributors:** Individual contribution statistics
8. **Timeline:** Chronological data addition

---

## Example Workflow

```
1. Create Project: "Mobile Banking App Research"
2. Upload FigJam: https://figma.com/file/ABC123/user-interviews
   ✅ 47 items extracted
3. Upload Audio: user_interview_transcript.mp3
   ✅ 23 insights extracted
4. Upload Document: competitive_analysis.pdf
   ✅ 18 insights extracted
5. Click "Analyze Project"
   📊 Generates comprehensive synthesis with 88 total items organized
```

---

## Features

✅ **Clean, intuitive interface**  
✅ **Tab-based data upload**  
✅ **Real-time status updates**  
✅ **Markdown-formatted reports**  
✅ **Project management**  
✅ **Refresh capability**  
✅ **Responsive design**  
✅ **Error handling**

---

## Next Steps

- Add styling/CSS for professional appearance
- Implement visualization charts
- Add export functionality (PDF, Excel)
- Enable project switching
- Add data filtering options
- Implement user authentication
