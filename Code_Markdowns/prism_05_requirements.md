# PRISM Requirements & Setup Guide

**Purpose:** Complete installation and configuration instructions

---

## Requirements File

Create `requirements.txt`:

```txt
# Core dependencies
numpy>=1.24.0
requests>=2.31.0

# UI Frameworks
gradio>=4.0.0
streamlit>=1.28.0

# Audio Processing
openai-whisper>=20230314
torch>=2.0.0
torchaudio>=2.0.0

# Document Processing
PyPDF2>=3.0.1
python-pptx>=0.6.21
python-docx>=1.1.0

# Machine Learning (for future upgrades)
transformers>=4.35.0
sentence-transformers>=2.2.2
scikit-learn>=1.3.0

# Utilities
python-dateutil>=2.8.2
```

---

## Installation Steps

### 1. Create Virtual Environment

```bash
# Create environment
python -m venv prism_env

# Activate (Linux/Mac)
source prism_env/bin/activate

# Activate (Windows)
prism_env\Scripts\activate
```

### 2. Install Dependencies

```bash
# Install all requirements
pip install -r requirements.txt

# Verify installation
python -c "import gradio; import whisper; import PyPDF2; print('✅ All imports successful')"
```

### 3. Test GPU (if available)

```bash
# Test CUDA availability
python -c "import torch; print(f'CUDA Available: {torch.cuda.is_available()}')"
```

---

## Project Structure

```
PRISM/
│
├── data/
│   ├── synthetic_data/
│   │   ├── test_100/
│   │   │   └── training_data.json
│   │   └── full_10k/
│   │       └── training_data.json
│   │
│   └── uploads/
│       ├── audio/
│       ├── documents/
│       └── temp/
│
├── models/
│   └── (Whisper models download here automatically)
│
├── outputs/
│   ├── reports/
│   └── exports/
│
├── src/
│   ├── synthetic_data_generator.py
│   ├── prism_brain_v2.py
│   ├── gradio_interface.py
│   └── streamlit_app.py
│
├── notebooks/
│   ├── Generator_V1.ipynb
│   └── THE_AI_TEMPLATE.ipynb
│
├── requirements.txt
├── README.md
└── .env (for API tokens)
```

---

## Environment Configuration

Create `.env` file:

```bash
# FigJam API Token
FIGMA_TOKEN=figd_YOUR_TOKEN_HERE

# Optional: HuggingFace Token (for future ML models)
HF_TOKEN=your_huggingface_token_here

# Project Paths
DATA_DIR=./data
OUTPUT_DIR=./outputs
```

---

## FigJam API Setup

### Get Your Token

1. Go to https://www.figma.com/developers/api
2. Click "Get personal access token"
3. Name it: "PRISM Research Tool"
4. Required scopes: `file_content:read`
5. Copy the token (starts with `figd_`)
6. Add to `.env` file

### Test Token

```python
import requests

token = "figd_YOUR_TOKEN_HERE"
headers = {'X-Figma-Token': token}

response = requests.get(
    'https://api.figma.com/v1/files/YOUR_FILE_KEY',
    headers=headers
)

print(f"Status: {response.status_code}")
```

---

## HP AI Studio Setup

### Upload to Workspace

```bash
# From your local machine, upload to HP AI Studio
scp -r PRISM/ user@hp-ai-studio:/home/jovyan/local/DeepLearning/

# Or use JupyterLab file upload interface
```

### Configure Environment

```bash
# On HP AI Studio
cd /home/jovyan/local/DeepLearning/PRISM

# Create virtual environment
python3.12 -m venv prism_env

# Activate
source prism_env/bin/activate

# Install requirements
pip install -r requirements.txt
```

---

## Quick Start Commands

### Generate Training Data

```bash
cd src
python synthetic_data_generator.py
```

### Launch Gradio Interface

```bash
cd src
python gradio_interface.py
```

Access at: `http://localhost:7860`

### Launch Streamlit App

```bash
cd src
streamlit run streamlit_app.py
```

Access at: `http://localhost:8501`

### Run in Jupyter

```bash
jupyter notebook notebooks/THE_AI_TEMPLATE.ipynb
```

---

## Common Issues & Solutions

### Issue: Whisper Model Download Fails

**Solution:**
```bash
# Pre-download Whisper model
python -c "import whisper; whisper.load_model('base')"
```

### Issue: CUDA Out of Memory

**Solution:**
```python
# In prism_brain_v2.py, use smaller Whisper model
self.whisper_model = whisper.load_model("tiny")  # Instead of "base"
```

### Issue: FigJam API Rate Limiting

**Solution:**
- Wait 60 seconds between requests
- Implement request throttling
- Use pagination for large boards

### Issue: Port Already in Use

**Solution:**
```bash
# For Gradio
python gradio_interface.py --server-port 7861

# For Streamlit
streamlit run streamlit_app.py --server.port 8502
```

---

## Testing Checklist

- [ ] Python environment activated
- [ ] All dependencies installed (`pip list`)
- [ ] FigJam token configured in `.env`
- [ ] Whisper model downloaded successfully
- [ ] Can import all modules without errors
- [ ] Training data generated (optional)
- [ ] Gradio interface launches
- [ ] Streamlit app launches (if using)
- [ ] Can create new project
- [ ] Can upload test files
- [ ] Analysis generates successfully

---

## Performance Tips

### For HP AI Studio

```python
# Enable GPU for Whisper (if available)
import torch
device = "cuda" if torch.cuda.is_available() else "cpu"
whisper_model = whisper.load_model("base", device=device)
```

### For Large Files

```python
# Process files in chunks
def process_large_pdf(pdf_path, chunk_size=10):
    pdf_reader = PyPDF2.PdfReader(pdf_path)
    total_pages = len(pdf_reader.pages)
    
    for i in range(0, total_pages, chunk_size):
        chunk = pdf_reader.pages[i:i+chunk_size]
        # Process chunk
```

### Memory Management

```python
# Clear memory after processing
import gc
import torch

def cleanup():
    gc.collect()
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
```

---

## Next Steps

1. **Generate Training Data:** Run `synthetic_data_generator.py`
2. **Test Basic Flow:** Create project → Upload file → Analyze
3. **Train ML Model:** Use training data to improve classifier (future)
4. **Deploy:** Share Gradio public link or deploy to server
5. **Iterate:** Add features based on user feedback

---

## Useful Commands

```bash
# Check Python version
python --version

# List installed packages
pip list

# Update package
pip install --upgrade gradio

# Freeze current environment
pip freeze > requirements_frozen.txt

# Check GPU
nvidia-smi

# Monitor GPU usage
watch -n 1 nvidia-smi

# Kill process on port
lsof -ti:7860 | xargs kill -9
```

---

## Support Resources

- **Gradio Docs:** https://www.gradio.app/docs
- **Streamlit Docs:** https://docs.streamlit.io
- **Whisper GitHub:** https://github.com/openai/whisper
- **FigJam API:** https://www.figma.com/developers/api
- **HP AI Studio:** Internal documentation

---

## Troubleshooting Contacts

For issues specific to:
- **HP AI Studio:** Contact IT support
- **FigJam API:** Check Figma developer forums
- **PRISM Code:** See project README or open GitHub issue
