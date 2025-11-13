# PRISM Streamlit App
## Enhanced UI with Glassmorphism Design

**Purpose:** Production-ready Streamlit interface with professional styling

**File:** `streamlit_app.py`

---

## Complete Implementation with Custom CSS

```python
import streamlit as st
import json
from datetime import datetime
from prism_brain_v2 import PRISMBrainV2

# ============================================================
# PAGE CONFIGURATION
# ============================================================

st.set_page_config(
    page_title="PRISM - Research Synthesis Engine",
    page_icon="🔬",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ============================================================
# CUSTOM CSS STYLING
# ============================================================

st.markdown("""
<style>
    /* Import Google Fonts */
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
    
    /* Global Styles */
    * {
        font-family: 'Inter', sans-serif;
    }
    
    /* Main Background Gradient */
    .stApp {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 25%, #f093fb 50%, #4facfe 75%, #00f2fe 100%);
        background-size: 400% 400%;
        animation: gradientShift 15s ease infinite;
    }
    
    @keyframes gradientShift {
        0% { background-position: 0% 50%; }
        50% { background-position: 100% 50%; }
        100% { background-position: 0% 50%; }
    }
    
    /* Glassmorphism Cards */
    .glass-card {
        background: rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(10px);
        -webkit-backdrop-filter: blur(10px);
        border-radius: 20px;
        border: 1px solid rgba(255, 255, 255, 0.18);
        padding: 2rem;
        margin: 1rem 0;
        box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    }
    
    /* Header Styling */
    h1, h2, h3 {
        color: #ffffff;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
        font-weight: 700;
    }
    
    /* Sidebar Styling */
    [data-testid="stSidebar"] {
        background: linear-gradient(180deg, rgba(102, 126, 234, 0.9) 0%, rgba(118, 75, 162, 0.9) 100%);
        backdrop-filter: blur(10px);
    }
    
    [data-testid="stSidebar"] h1, [data-testid="stSidebar"] h2, [data-testid="stSidebar"] h3 {
        color: #ffffff;
    }
    
    /* Button Styling */
    .stButton > button {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        border: none;
        padding: 0.75rem 2rem;
        border-radius: 50px;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 1px;
        transition: all 0.3s ease;
        box-shadow: 0 4px 15px 0 rgba(102, 126, 234, 0.75);
    }
    
    .stButton > button:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 20px 0 rgba(102, 126, 234, 0.95);
    }
    
    /* Input Fields */
    .stTextInput > div > div > input,
    .stTextArea > div > div > textarea {
        background: rgba(255, 255, 255, 0.9);
        border: 2px solid rgba(255, 255, 255, 0.3);
        border-radius: 15px;
        padding: 1rem;
        font-size: 1rem;
        transition: all 0.3s ease;
    }
    
    .stTextInput > div > div > input:focus,
    .stTextArea > div > div > textarea:focus {
        border-color: #667eea;
        box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.25);
    }
    
    /* Tabs */
    .stTabs [data-baseweb="tab-list"] {
        gap: 8px;
        background: rgba(255, 255, 255, 0.1);
        padding: 0.5rem;
        border-radius: 50px;
        backdrop-filter: blur(10px);
    }
    
    .stTabs [data-baseweb="tab"] {
        background: transparent;
        color: white;
        border-radius: 50px;
        padding: 0.5rem 1.5rem;
        font-weight: 600;
        transition: all 0.3s ease;
    }
    
    .stTabs [data-baseweb="tab"]:hover {
        background: rgba(255, 255, 255, 0.2);
    }
    
    .stTabs [aria-selected="true"] {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
        box-shadow: 0 4px 15px 0 rgba(102, 126, 234, 0.75);
    }
    
    /* File Uploader */
    .stFileUploader {
        background: rgba(255, 255, 255, 0.1);
        border: 2px dashed rgba(255, 255, 255, 0.5);
        border-radius: 20px;
        padding: 2rem;
        backdrop-filter: blur(10px);
        transition: all 0.3s ease;
    }
    
    .stFileUploader:hover {
        border-color: #667eea;
        background: rgba(255, 255, 255, 0.15);
    }
    
    /* Success/Info/Warning Messages */
    .stSuccess, .stInfo, .stWarning {
        background: rgba(255, 255, 255, 0.15) !important;
        backdrop-filter: blur(10px);
        border-radius: 15px;
        border-left: 4px solid;
    }
    
    /* Metrics */
    [data-testid="stMetricValue"] {
        color: #ffffff;
        font-size: 2rem;
        font-weight: 700;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
    }
    
    [data-testid="stMetricLabel"] {
        color: rgba(255, 255, 255, 0.9);
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 1px;
    }
    
    /* Progress Bar */
    .stProgress > div > div > div > div {
        background: linear-gradient(90deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
    }
    
    /* Expander */
    .streamlit-expanderHeader {
        background: rgba(255, 255, 255, 0.1);
        border-radius: 10px;
        color: white;
        font-weight: 600;
    }
    
    /* Markdown Content */
    .stMarkdown {
        color: rgba(255, 255, 255, 0.95);
    }
</style>
""", unsafe_allow_html=True)

# ============================================================
# SESSION STATE INITIALIZATION
# ============================================================

if 'brain' not in st.session_state:
    st.session_state.brain = None
if 'current_project_id' not in st.session_state:
    st.session_state.current_project_id = None
if 'figma_token' not in st.session_state:
    st.session_state.figma_token = ""

# ============================================================
# SIDEBAR: PROJECT NAVIGATION
# ============================================================

with st.sidebar:
    st.title("🔬 PRISM")
    st.markdown("### Pattern Recognition & Insight Structure Module")
    
    st.markdown("---")
    
    # Configuration
    st.markdown("### ⚙️ Configuration")
    
    figma_token = st.text_input(
        "FigJam API Token",
        type="password",
        value=st.session_state.figma_token,
        help="Get your token from https://www.figma.com/developers/api"
    )
    
    if figma_token != st.session_state.figma_token:
        st.session_state.figma_token = figma_token
        st.session_state.brain = PRISMBrainV2(figma_token=figma_token)
        st.success("Token updated!")
    
    if not st.session_state.brain and figma_token:
        st.session_state.brain = PRISMBrainV2(figma_token=figma_token)
    
    st.markdown("---")
    
    # Project Selection
    st.markdown("### 📁 Current Project")
    
    if st.session_state.current_project_id:
        project = st.session_state.brain.get_project(st.session_state.current_project_id)
        if project:
            st.info(f"**{project['name']}**")
            st.caption(f"ID: {st.session_state.current_project_id}")
            st.caption(f"Sources: {len(project['sources'])}")
            st.caption(f"Notes: {len(project['notes'])}")
    else:
        st.warning("No project selected")
    
    st.markdown("---")
    
    # Quick Stats
    if st.session_state.current_project_id and st.session_state.brain:
        st.markdown("### 📊 Quick Stats")
        project = st.session_state.brain.get_project(st.session_state.current_project_id)
        
        if project:
            col1, col2 = st.columns(2)
            with col1:
                st.metric("Sources", len(project['sources']))
            with col2:
                st.metric("Notes", len(project['notes']))

# ============================================================
# MAIN CONTENT
# ============================================================

# Header
st.markdown("<h1 style='text-align: center;'>🔬 PRISM Research Synthesis Engine</h1>", unsafe_allow_html=True)
st.markdown("<p style='text-align: center; color: rgba(255,255,255,0.9); font-size: 1.2rem;'>Transform multi-modal research data into actionable insights</p>", unsafe_allow_html=True)

st.markdown("---")

# Tab Navigation
tab1, tab2, tab3, tab4 = st.tabs(["🎯 Project", "📤 Upload Data", "📊 Analysis", "ℹ️ About"])

# ============================================================
# TAB 1: PROJECT MANAGEMENT
# ============================================================

with tab1:
    st.markdown("## Create New Project")
    
    col1, col2 = st.columns([3, 1])
    
    with col1:
        project_name = st.text_input(
            "Project Name",
            placeholder="e.g., Mobile App Redesign Research",
            label_visibility="collapsed"
        )
    
    with col2:
        create_btn = st.button("Create Project", type="primary", use_container_width=True)
    
    if create_btn:
        if not project_name:
            st.error("Please enter a project name")
        elif not st.session_state.brain:
            st.error("Please configure FigJam token in sidebar")
        else:
            project_id = st.session_state.brain.create_project(project_name)
            st.session_state.current_project_id = project_id
            st.success(f"✅ Created project: {project_name}")
            st.balloons()
            st.rerun()
    
    st.markdown("---")
    
    # Project List
    if st.session_state.brain and st.session_state.brain.projects:
        st.markdown("## Your Projects")
        
        for proj_id, proj_data in st.session_state.brain.projects.items():
            with st.expander(f"📁 {proj_data['name']}"):
                col1, col2, col3 = st.columns(3)
                
                with col1:
                    st.metric("Sources", len(proj_data['sources']))
                with col2:
                    st.metric("Notes", len(proj_data['notes']))
                with col3:
                    st.metric("Contributors", len(proj_data['contributors']))
                
                if st.button(f"Load Project", key=f"load_{proj_id}"):
                    st.session_state.current_project_id = proj_id
                    st.success(f"Loaded: {proj_data['name']}")
                    st.rerun()

# ============================================================
# TAB 2: DATA UPLOAD
# ============================================================

with tab2:
    if not st.session_state.current_project_id:
        st.warning("⚠️ Please create or select a project first")
    else:
        st.markdown("## Upload Data Sources")
        
        upload_tabs = st.tabs(["📌 FigJam Board", "🎤 Audio File", "📄 Document"])
        
        # FigJam Upload
        with upload_tabs[0]:
            st.markdown("### Process FigJam Board")
            
            figjam_url = st.text_input(
                "FigJam Board URL",
                placeholder="https://www.figma.com/file/...",
                help="Paste the full URL of your FigJam board"
            )
            
            if st.button("Process FigJam", type="primary", key="figjam_btn"):
                if not figjam_url:
                    st.error("Please enter a FigJam URL")
                else:
                    with st.spinner("Processing FigJam board..."):
                        result = st.session_state.brain.ingest_figjam_url(
                            st.session_state.current_project_id,
                            figjam_url
                        )
                        
                        if 'error' in result:
                            st.error(f"Error: {result['error']}")
                        else:
                            st.success(f"✅ Processed: {result['board_name']}")
                            st.info(f"Extracted {result['notes']} items")
            
            st.markdown("""
            **What gets analyzed:**
            - Sticky notes (all colors)
            - Text boxes
            - Arrows and connections
            - Diagrams and shapes
            - Spatial relationships
            """)
        
        # Audio Upload
        with upload_tabs[1]:
            st.markdown("### Process Audio File")
            
            audio_file = st.file_uploader(
                "Upload Audio",
                type=['mp3', 'wav', 'm4a', 'mov'],
                help="Upload interview recordings or meeting audio"
            )
            
            if st.button("Process Audio", type="primary", key="audio_btn"):
                if not audio_file:
                    st.error("Please upload an audio file")
                else:
                    with st.spinner("Transcribing with Whisper AI..."):
                        # Save temporarily
                        temp_path = f"/tmp/{audio_file.name}"
                        with open(temp_path, 'wb') as f:
                            f.write(audio_file.getbuffer())
                        
                        result = st.session_state.brain.ingest_audio_file(
                            st.session_state.current_project_id,
                            temp_path
                        )
                        
                        if 'error' in result:
                            st.error(f"Error: {result['error']}")
                        else:
                            st.success(f"✅ Processed audio file")
                            st.info(f"Extracted {result['notes']} insights")
            
            st.markdown("""
            **Processing includes:**
            - Whisper AI transcription
            - Tone detection
            - Speaker pattern analysis
            - Key point extraction
            """)
        
        # Document Upload
        with upload_tabs[2]:
            st.markdown("### Process Document")
            
            doc_file = st.file_uploader(
                "Upload Document",
                type=['pdf', 'pptx', 'ppt', 'txt', 'docx'],
                help="Upload research reports, presentations, or notes"
            )
            
            if st.button("Process Document", type="primary", key="doc_btn"):
                if not doc_file:
                    st.error("Please upload a document")
                else:
                    with st.spinner("Processing document..."):
                        # Save temporarily
                        temp_path = f"/tmp/{doc_file.name}"
                        with open(temp_path, 'wb') as f:
                            f.write(doc_file.getbuffer())
                        
                        result = st.session_state.brain.ingest_document_file(
                            st.session_state.current_project_id,
                            temp_path
                        )
                        
                        if 'error' in result:
                            st.error(f"Error: {result['error']}")
                        else:
                            st.success(f"✅ Processed document")
                            st.info(f"Extracted {result['notes']} insights from {result['pages']} pages")
            
            st.markdown("""
            **Supported formats:**
            - PDF documents
            - PowerPoint presentations
            - Text files
            - Word documents
            """)

# ============================================================
# TAB 3: ANALYSIS
# ============================================================

with tab3:
    if not st.session_state.current_project_id:
        st.warning("⚠️ Please create or select a project first")
    else:
        st.markdown("## Generate Synthesis Report")
        
        col1, col2 = st.columns([3, 1])
        
        with col1:
            analyze_btn = st.button("🔍 Analyze Project", type="primary", use_container_width=True)
        with col2:
            refresh_btn = st.button("🔄 Refresh", use_container_width=True)
        
        if analyze_btn or refresh_btn:
            with st.spinner("Generating synthesis..."):
                synthesis = st.session_state.brain.synthesize_project(st.session_state.current_project_id)
                
                if 'error' in synthesis:
                    st.error(f"Error: {synthesis['error']}")
                else:
                    # Display synthesis
                    st.markdown("---")
                    
                    # Overview Metrics
                    st.markdown("### 📈 Overview")
                    col1, col2, col3 = st.columns(3)
                    
                    with col1:
                        st.metric("Total Notes", synthesis['total_notes'])
                    with col2:
                        st.metric("Sources", synthesis['total_sources'])
                    with col3:
                        st.metric("Contributors", synthesis['contributors'])
                    
                    st.markdown("---")
                    
                    # Content Distribution
                    st.markdown("### 📑 Content Distribution")
                    
                    for content_type, count in synthesis['by_type'].items():
                        percentage = (count / synthesis['total_notes']) * 100
                        st.progress(percentage / 100, text=f"{content_type.replace('_', ' ').title()}: {count} ({percentage:.1f}%)")
                    
                    st.markdown("---")
                    
                    # Priority Breakdown
                    st.markdown("### 🎯 Priority Breakdown")
                    col1, col2, col3 = st.columns(3)
                    
                    with col1:
                        st.metric("High Priority", synthesis['by_priority']['high'])
                    with col2:
                        st.metric("Medium Priority", synthesis['by_priority']['medium'])
                    with col3:
                        st.metric("Low Priority", synthesis['by_priority']['low'])
                    
                    st.markdown("---")
                    
                    # Key Themes
                    st.markdown("### 💡 Key Themes")
                    
                    for theme in synthesis['themes'][:5]:
                        st.write(f"**{theme['theme']}** - mentioned {theme['frequency']} times")
                    
                    st.markdown("---")
                    
                    # Action Items
                    st.markdown("### ⚠️ Top Action Items")
                    
                    for i, action in enumerate(synthesis['action_items'][:5], 1):
                        with st.expander(f"{i}. {action.get('content', 'N/A')[:50]}..."):
                            st.write(f"**Full Content:** {action.get('content', 'N/A')}")
                            st.write(f"**Priority:** {action.get('priority', 0):.2f}")
                            st.write(f"**Source:** {action.get('source_type', 'Unknown')}")

# ============================================================
# TAB 4: ABOUT
# ============================================================

with tab4:
    st.markdown("## About PRISM")
    
    st.markdown("""
    **PRISM** (Pattern Recognition & Insight Structure Module) is an AI-powered research synthesis engine
    designed to transform how design teams extract actionable insights from user research data.
    
    ### 🎯 Key Features
    
    - **Multi-modal Input:** Process FigJam boards, audio recordings, and documents
    - **Deep Analysis:** Goes beyond surface-level tagging to understand context and relationships
    - **Automatic Organization:** Categorizes insights by type, priority, and sentiment
    - **Timeline Tracking:** See how research evolves over time
    - **Contributor Attribution:** Track who said what and when
    
    ### 🔬 How It Works
    
    1. **Ingest:** Upload your research data from multiple sources
    2. **Analyze:** PRISM processes and categorizes all content
    3. **Synthesize:** Generate comprehensive reports with actionable insights
    4. **Export:** Share findings with your team
    
    ### 🛠️ Built With
    
    - Streamlit for the interface
    - Whisper AI for audio transcription
    - Custom NLP for content analysis
    - FigJam API for board processing
    
    ### 📚 Learn More
    
    For documentation and support, visit the PRISM GitHub repository.
    """)

# ============================================================
# FOOTER
# ============================================================

st.markdown("---")
st.markdown("""
<div style='text-align: center; color: rgba(255,255,255,0.7); padding: 2rem;'>
    <p>PRISM Research Synthesis Engine v2.0</p>
    <p>Built with ❤️ for UX Researchers</p>
</div>
""", unsafe_allow_html=True)
```

---

## Launch Command

```bash
streamlit run streamlit_app.py
```

---

## Key Features of This Implementation

✅ **Glassmorphism Design:** Beautiful frosted glass effect  
✅ **Purple-Blue-Pink Gradients:** Animated gradient background  
✅ **Professional Typography:** Inter font family  
✅ **Smooth Animations:** Hover effects and transitions  
✅ **Responsive Layout:** Works on all screen sizes  
✅ **Tab-based Navigation:** Clean organization  
✅ **Real-time Updates:** Session state management  
✅ **Progress Indicators:** Visual feedback for all operations

---

## Design Elements

- **Animated gradient background** (purple → blue → pink)
- **Glassmorphism cards** with backdrop blur
- **Rounded buttons** with hover lift effect
- **Custom styled inputs** with focus animations
- **Beautiful tabs** with gradient selection
- **Floating file uploaders** with dashed borders
- **Metrics with shadows** for depth
- **Professional sidebar** with gradient background
