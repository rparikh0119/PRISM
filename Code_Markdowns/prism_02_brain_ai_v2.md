# PRISM Brain AI v2
## Multi-Modal Research Synthesis Engine

**Purpose:** Ingest and analyze FigJam boards, audio files, and documents to generate organized insights

**File:** `THE_AI_TEMPLATE.ipynb` or `prism_brain_v2.py`

---

## Complete Implementation

```python
import json
import os
import re
import hashlib
from datetime import datetime
from collections import defaultdict
import requests
import numpy as np

# Audio processing
import whisper

# Document processing
import PyPDF2
from pptx import Presentation

# Future: Machine Learning
# from transformers import pipeline

# ============================================================
# CELL 1: PRISM BRAIN AI V2 CLASS
# ============================================================

class PRISMBrainV2:
    """
    Multi-modal research synthesis engine
    Processes FigJam boards, audio files, and documents
    """
    
    def __init__(self, figma_token=None):
        self.figma_token = figma_token
        self.projects = {}
        
        # Load Whisper model for audio transcription
        print("🧠 Initializing PRISM Brain v2...")
        try:
            self.whisper_model = whisper.load_model("base")
            print("✅ Whisper model loaded")
        except Exception as e:
            print(f"⚠️  Whisper not available: {e}")
            self.whisper_model = None
        
        print("✅ PRISM Brain v2 initialized")
    
    # ==================== PROJECT MANAGEMENT ====================
    
    def create_project(self, project_name):
        """Create a new research project"""
        project_id = hashlib.md5(project_name.encode()).hexdigest()[:8]
        
        self.projects[project_id] = {
            'name': project_name,
            'created_at': datetime.now().isoformat(),
            'sources': [],
            'notes': [],
            'timeline': [],
            'contributors': set()
        }
        
        print(f"✅ Created project: {project_name} (ID: {project_id})")
        return project_id
    
    def get_project(self, project_id):
        """Retrieve project data"""
        return self.projects.get(project_id)
    
    # ==================== FIGJAM PROCESSING ====================
    
    def ingest_figjam_url(self, project_id, figjam_url):
        """
        Ingest FigJam board via URL
        Analyzes sticky notes, arrows, connections, and diagrams
        """
        print(f"📌 Processing FigJam board from URL...")
        
        if not self.figma_token:
            return {"error": "FigJam token not configured"}
        
        # Extract file key from URL
        # URL format: https://www.figma.com/file/FILE_KEY/...
        match = re.search(r'/file/([^/]+)', figjam_url)
        if not match:
            return {"error": "Invalid FigJam URL"}
        
        file_key = match.group(1)
        
        # Fetch board data via API
        try:
            headers = {'X-Figma-Token': self.figma_token}
            response = requests.get(
                f'https://api.figma.com/v1/files/{file_key}',
                headers=headers
            )
            
            if response.status_code != 200:
                return {"error": f"API error: {response.status_code}"}
            
            data = response.json()
            
            # Extract board name
            board_name = data.get('name', 'Unnamed Board')
            
            # Parse sticky notes and connections
            notes = self._extract_figjam_content(data)
            
            # Add to project
            if project_id in self.projects:
                source_entry = {
                    'type': 'figjam',
                    'name': board_name,
                    'url': figjam_url,
                    'added_at': datetime.now().isoformat(),
                    'note_count': len(notes)
                }
                self.projects[project_id]['sources'].append(source_entry)
                self.projects[project_id]['notes'].extend(notes)
                
                # Track contributors
                for note in notes:
                    if note.get('contributor'):
                        self.projects[project_id]['contributors'].add(note['contributor'])
            
            print(f"✅ Processed {len(notes)} items from FigJam board")
            return {"success": True, "notes": len(notes), "board_name": board_name}
            
        except Exception as e:
            print(f"❌ Error processing FigJam: {e}")
            return {"error": str(e)}
    
    def _extract_figjam_content(self, figma_data):
        """
        Extract sticky notes, arrows, and diagrams from FigJam data
        """
        notes = []
        
        def traverse_nodes(node, depth=0):
            """Recursively traverse FigJam nodes"""
            
            # Check for sticky notes
            if node.get('type') == 'STICKY':
                note_data = {
                    'id': node.get('id'),
                    'content': node.get('characters', ''),
                    'color': self._map_figjam_color(node.get('fills', [])),
                    'position': node.get('absoluteBoundingBox', {}),
                    'contributor': node.get('lastModified', {}).get('user', 'Unknown'),
                    'timestamp': node.get('lastModified', {}).get('time', ''),
                    'source_type': 'figjam_sticky'
                }
                
                # Analyze content
                analysis = self._analyze_content(note_data['content'])
                note_data.update(analysis)
                
                notes.append(note_data)
            
            # Check for text nodes
            elif node.get('type') == 'TEXT':
                text_data = {
                    'id': node.get('id'),
                    'content': node.get('characters', ''),
                    'color': 'TEXT',
                    'position': node.get('absoluteBoundingBox', {}),
                    'contributor': 'Unknown',
                    'timestamp': datetime.now().isoformat(),
                    'source_type': 'figjam_text'
                }
                
                analysis = self._analyze_content(text_data['content'])
                text_data.update(analysis)
                
                notes.append(text_data)
            
            # Check for connectors/arrows
            elif node.get('type') == 'CONNECTOR':
                connector_data = {
                    'id': node.get('id'),
                    'from': node.get('connectorStart', {}).get('endpointNodeId'),
                    'to': node.get('connectorEnd', {}).get('endpointNodeId'),
                    'relationship_type': 'relates_to',
                    'source_type': 'figjam_connection'
                }
                notes.append(connector_data)
            
            # Traverse children
            if 'children' in node:
                for child in node['children']:
                    traverse_nodes(child, depth + 1)
        
        # Start traversal from document root
        if 'document' in figma_data:
            traverse_nodes(figma_data['document'])
        
        return notes
    
    def _map_figjam_color(self, fills):
        """Map FigJam RGB colors to standard color names"""
        if not fills or len(fills) == 0:
            return 'GRAY'
        
        # Get the first fill color
        fill = fills[0]
        if fill.get('type') != 'SOLID':
            return 'GRAY'
        
        color = fill.get('color', {})
        r = color.get('r', 0) * 255
        g = color.get('g', 0) * 255
        b = color.get('b', 0) * 255
        
        # Map to closest standard color
        if r > 200 and g < 100 and b < 100:
            return 'RED'
        elif r < 100 and g < 100 and b > 200:
            return 'BLUE'
        elif r < 100 and g > 200 and b < 100:
            return 'GREEN'
        elif r > 200 and g > 200 and b < 100:
            return 'YELLOW'
        elif r > 150 and g < 100 and b > 150:
            return 'PURPLE'
        elif r > 200 and g < 150 and b > 150:
            return 'PINK'
        elif r > 200 and g > 100 and b < 100:
            return 'ORANGE'
        else:
            return 'GRAY'
    
    # ==================== AUDIO PROCESSING ====================
    
    def ingest_audio_file(self, project_id, audio_file_path):
        """
        Ingest audio file and generate organized insights
        Uses Whisper for transcription + tone analysis
        """
        print(f"🎤 Processing audio file: {audio_file_path}")
        
        if not self.whisper_model:
            return {"error": "Whisper model not available"}
        
        if not os.path.exists(audio_file_path):
            return {"error": "Audio file not found"}
        
        try:
            # Transcribe with Whisper
            print("  Transcribing...")
            result = self.whisper_model.transcribe(audio_file_path)
            
            transcript = result['text']
            segments = result.get('segments', [])
            
            # Extract organized insights from transcript
            notes = self._extract_audio_insights(transcript, segments, audio_file_path)
            
            # Add to project
            if project_id in self.projects:
                source_entry = {
                    'type': 'audio',
                    'name': os.path.basename(audio_file_path),
                    'path': audio_file_path,
                    'added_at': datetime.now().isoformat(),
                    'duration': result.get('duration', 0),
                    'note_count': len(notes)
                }
                self.projects[project_id]['sources'].append(source_entry)
                self.projects[project_id]['notes'].extend(notes)
            
            print(f"✅ Extracted {len(notes)} insights from audio")
            return {"success": True, "notes": len(notes), "transcript_length": len(transcript)}
            
        except Exception as e:
            print(f"❌ Error processing audio: {e}")
            return {"error": str(e)}
    
    def _extract_audio_insights(self, transcript, segments, filename):
        """
        Extract organized insights from audio transcript
        """
        notes = []
        
        # Split transcript into sentences
        sentences = re.split(r'[.!?]+', transcript)
        
        for i, sentence in enumerate(sentences):
            sentence = sentence.strip()
            if len(sentence) < 10:
                continue
            
            # Create note from sentence
            note_data = {
                'id': f"audio_{i}",
                'content': sentence,
                'source_type': 'audio_transcript',
                'source_file': filename,
                'contributor': 'Audio Speaker',
                'timestamp': datetime.now().isoformat(),
                'segment_index': i
            }
            
            # Analyze tone and content
            analysis = self._analyze_content(sentence)
            analysis['tone'] = self._detect_tone(sentence)
            
            note_data.update(analysis)
            notes.append(note_data)
        
        return notes
    
    def _detect_tone(self, text):
        """Detect speaker tone/emphasis in text"""
        text_lower = text.lower()
        
        # Emphatic
        if any(word in text_lower for word in ['must', 'really', 'definitely', 'absolutely']):
            return 'emphatic'
        # Questioning
        elif '?' in text or any(word in text_lower for word in ['should we', 'what if', 'how about']):
            return 'questioning'
        # Hesitant
        elif any(word in text_lower for word in ['maybe', 'perhaps', 'might', 'not sure']):
            return 'hesitant'
        # Negative
        elif any(word in text_lower for word in ['problem', 'issue', 'difficult', 'frustrated']):
            return 'concerned'
        # Positive
        elif any(word in text_lower for word in ['great', 'love', 'excellent', 'perfect']):
            return 'positive'
        else:
            return 'neutral'
    
    # ==================== DOCUMENT PROCESSING ====================
    
    def ingest_document_file(self, project_id, doc_file_path):
        """
        Ingest document (PDF, PowerPoint, TXT)
        Extracts text and generates insights
        """
        print(f"📄 Processing document: {doc_file_path}")
        
        if not os.path.exists(doc_file_path):
            return {"error": "Document file not found"}
        
        filename = os.path.basename(doc_file_path)
        ext = os.path.splitext(filename)[1].lower()
        
        try:
            # Route to appropriate processor
            if ext == '.pdf':
                text, pages = self._extract_pdf(doc_file_path)
            elif ext in ['.pptx', '.ppt']:
                text, pages = self._extract_powerpoint(doc_file_path)
            elif ext == '.txt':
                with open(doc_file_path, 'r', encoding='utf-8') as f:
                    text = f.read()
                pages = 1
            else:
                return {"error": f"Unsupported file type: {ext}"}
            
            # Extract insights from text
            notes = self._extract_document_insights(text, filename, pages)
            
            # Add to project
            if project_id in self.projects:
                source_entry = {
                    'type': 'document',
                    'name': filename,
                    'path': doc_file_path,
                    'added_at': datetime.now().isoformat(),
                    'pages': pages,
                    'note_count': len(notes)
                }
                self.projects[project_id]['sources'].append(source_entry)
                self.projects[project_id]['notes'].extend(notes)
            
            print(f"✅ Extracted {len(notes)} insights from document")
            return {"success": True, "notes": len(notes), "pages": pages}
            
        except Exception as e:
            print(f"❌ Error processing document: {e}")
            return {"error": str(e)}
    
    def _extract_pdf(self, pdf_path):
        """Extract text from PDF"""
        text = ""
        with open(pdf_path, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            pages = len(pdf_reader.pages)
            
            for page in pdf_reader.pages:
                text += page.extract_text() + "\n\n"
        
        return text, pages
    
    def _extract_powerpoint(self, pptx_path):
        """Extract text from PowerPoint"""
        text = ""
        prs = Presentation(pptx_path)
        
        for slide in prs.slides:
            for shape in slide.shapes:
                if hasattr(shape, "text"):
                    text += shape.text + "\n"
            text += "\n\n"
        
        return text, len(prs.slides)
    
    def _extract_document_insights(self, text, filename, pages):
        """Extract insights from document text"""
        notes = []
        
        # Split into paragraphs
        paragraphs = [p.strip() for p in text.split('\n\n') if p.strip()]
        
        for i, para in enumerate(paragraphs):
            if len(para) < 20:  # Skip very short paragraphs
                continue
            
            # Create note from paragraph
            note_data = {
                'id': f"doc_{i}",
                'content': para[:500],  # Limit length
                'source_type': 'document_paragraph',
                'source_file': filename,
                'contributor': 'Document Author',
                'timestamp': datetime.now().isoformat(),
                'paragraph_index': i
            }
            
            # Analyze content
            analysis = self._analyze_content(para)
            note_data.update(analysis)
            
            notes.append(note_data)
        
        return notes
    
    # ==================== CONTENT ANALYSIS ====================
    
    def _analyze_content(self, text):
        """
        Analyze content to determine type, priority, sentiment, tags
        """
        text_lower = text.lower()
        
        # Determine content type
        content_type = self._classify_content(text_lower)
        
        # Calculate priority (0-1 scale)
        priority = self._calculate_priority(text_lower)
        
        # Detect sentiment
        sentiment = self._detect_sentiment(text_lower)
        
        # Extract tags
        tags = self._extract_tags(text_lower)
        
        return {
            'predicted_type': content_type,
            'priority': priority,
            'sentiment': sentiment,
            'tags': tags,
            'confidence': 0.4  # Rule-based, so lower confidence
        }
    
    def _classify_content(self, text):
        """Classify content type based on keywords"""
        
        # Pain point keywords
        if any(word in text for word in ['frustrated', 'problem', 'issue', 'difficult', 'confusing', 'error', 'broken']):
            return 'pain_point'
        
        # Question keywords
        elif any(word in text for word in ['how', 'what', 'why', 'should', '?']):
            return 'question'
        
        # Insight keywords
        elif any(word in text for word in ['finding', 'discovered', 'noticed', 'observed', 'users love', 'works well']):
            return 'insight'
        
        # Action item keywords
        elif any(word in text for word in ['todo', 'action', 'must', 'need to', 'should complete', '[]']):
            return 'action_item'
        
        # Quote indicators
        elif '"' in text or "said" in text or "quote" in text:
            return 'quote'
        
        # Idea keywords
        elif any(word in text for word in ['idea', 'what if', 'consider', 'brainstorm', 'concept']):
            return 'idea'
        
        else:
            return 'general'
    
    def _calculate_priority(self, text):
        """Calculate priority score (0.0 to 1.0)"""
        priority = 0.3  # Base priority
        
        # High priority keywords
        if any(word in text for word in ['urgent', 'asap', 'critical', 'must', 'immediately']):
            priority += 0.4
        
        # Negative sentiment increases priority
        if any(word in text for word in ['broken', 'error', 'frustrated', 'failing']):
            priority += 0.2
        
        # Action words increase priority
        if any(word in text for word in ['fix', 'resolve', 'address', 'complete']):
            priority += 0.1
        
        return min(priority, 1.0)
    
    def _detect_sentiment(self, text):
        """Detect sentiment: positive, negative, or neutral"""
        positive_words = ['love', 'great', 'excellent', 'perfect', 'works well', 'helpful', 'easy']
        negative_words = ['hate', 'frustrated', 'difficult', 'confusing', 'problem', 'error', 'broken']
        
        pos_count = sum(1 for word in positive_words if word in text)
        neg_count = sum(1 for word in negative_words if word in text)
        
        if pos_count > neg_count:
            return 'positive'
        elif neg_count > pos_count:
            return 'negative'
        else:
            return 'neutral'
    
    def _extract_tags(self, text):
        """Extract relevant tags from content"""
        tags = []
        
        # Common UX research tags
        tag_keywords = {
            'usability': ['usability', 'user-friendly', 'intuitive'],
            'navigation': ['navigation', 'menu', 'find'],
            'design': ['design', 'layout', 'visual'],
            'performance': ['slow', 'fast', 'loading'],
            'accessibility': ['accessibility', 'screen reader', 'contrast'],
            'mobile': ['mobile', 'phone', 'responsive'],
            'onboarding': ['onboarding', 'first-time', 'tutorial']
        }
        
        for tag, keywords in tag_keywords.items():
            if any(keyword in text for keyword in keywords):
                tags.append(tag)
        
        return tags[:5]  # Limit to 5 tags
    
    # ==================== SYNTHESIS & REPORTING ====================
    
    def synthesize_project(self, project_id):
        """
        Generate comprehensive synthesis report for a project
        """
        if project_id not in self.projects:
            return {"error": "Project not found"}
        
        project = self.projects[project_id]
        notes = project['notes']
        
        # Organize by content type
        by_type = defaultdict(list)
        for note in notes:
            note_type = note.get('predicted_type', 'general')
            by_type[note_type].append(note)
        
        # Organize by priority
        high_priority = [n for n in notes if n.get('priority', 0) > 0.7]
        medium_priority = [n for n in notes if 0.4 <= n.get('priority', 0) <= 0.7]
        low_priority = [n for n in notes if n.get('priority', 0) < 0.4]
        
        # Organize by contributor
        by_contributor = defaultdict(lambda: {'count': 0, 'types': defaultdict(int)})
        for note in notes:
            contributor = note.get('contributor', 'Unknown')
            by_contributor[contributor]['count'] += 1
            note_type = note.get('predicted_type', 'general')
            by_contributor[contributor]['types'][note_type] += 1
        
        # Generate timeline
        timeline = []
        for source in project['sources']:
            timeline.append({
                'date': source['added_at'],
                'event': f"Added {source['type']}: {source['name']}",
                'note_count': source['note_count']
            })
        
        # Identify themes
        themes = self._identify_themes(notes)
        
        # Extract action items
        action_items = by_type.get('action_item', [])
        
        # Calculate stats
        sentiment_dist = {'positive': 0, 'negative': 0, 'neutral': 0}
        for note in notes:
            sentiment = note.get('sentiment', 'neutral')
            sentiment_dist[sentiment] += 1
        
        # Create synthesis
        synthesis = {
            'project_name': project['name'],
            'last_updated': datetime.now().isoformat(),
            'total_notes': len(notes),
            'total_sources': len(project['sources']),
            'contributors': len(project['contributors']),
            'by_type': {k: len(v) for k, v in by_type.items()},
            'by_priority': {
                'high': len(high_priority),
                'medium': len(medium_priority),
                'low': len(low_priority)
            },
            'by_contributor': dict(by_contributor),
            'timeline': timeline,
            'themes': themes,
            'action_items': action_items[:10],  # Top 10
            'stats': {
                'sentiment_distribution': sentiment_dist,
                'avg_confidence': np.mean([n.get('confidence', 0) for n in notes])
            }
        }
        
        return synthesis
    
    def _identify_themes(self, notes):
        """Identify recurring themes in notes"""
        # Simple keyword frequency analysis
        word_freq = defaultdict(int)
        
        for note in notes:
            content = note.get('content', '').lower()
            words = re.findall(r'\b\w{4,}\b', content)  # Words 4+ characters
            for word in words:
                if word not in ['that', 'this', 'with', 'from', 'have', 'been']:
                    word_freq[word] += 1
        
        # Get top themes
        top_themes = sorted(word_freq.items(), key=lambda x: x[1], reverse=True)[:10]
        
        return [{'theme': theme, 'frequency': freq} for theme, freq in top_themes]
    
    def refresh_project(self, project_id):
        """Re-analyze project with latest data"""
        return self.synthesize_project(project_id)

# ============================================================
# EXECUTION EXAMPLE
# ============================================================

# Initialize PRISM Brain
brain = PRISMBrainV2(figma_token="YOUR_FIGMA_TOKEN_HERE")

# Create a project
project_id = brain.create_project("Mobile App Redesign Research")

# Ingest data sources
brain.ingest_figjam_url(project_id, "https://www.figma.com/file/ABC123/...")
brain.ingest_audio_file(project_id, "./recordings/user_interview_1.mp3")
brain.ingest_document_file(project_id, "./reports/competitive_analysis.pdf")

# Generate synthesis
synthesis = brain.synthesize_project(project_id)

print(json.dumps(synthesis, indent=2))
```

---

## Key Features

✅ **Multi-modal input:** FigJam boards, audio files, documents  
✅ **FigJam deep analysis:** Sticky notes + arrows + connections + diagrams  
✅ **Audio transcription:** Whisper AI with tone detection  
✅ **Document parsing:** PDF, PowerPoint, TXT support  
✅ **Content analysis:** Type classification, priority scoring, sentiment  
✅ **Project management:** Multiple projects, timeline tracking  
✅ **Contributor tracking:** Who said what, when  
✅ **Synthesis reports:** Comprehensive organized insights

---

## Output Example

```json
{
  "project_name": "Mobile App Redesign Research",
  "total_notes": 247,
  "total_sources": 3,
  "by_type": {
    "pain_point": 45,
    "question": 38,
    "insight": 52,
    "idea": 41,
    "action_item": 28,
    "quote": 43
  },
  "by_priority": {
    "high": 31,
    "medium": 128,
    "low": 88
  },
  "themes": [
    {"theme": "navigation", "frequency": 23},
    {"theme": "loading", "frequency": 18}
  ],
  "action_items": [
    {
      "content": "TODO: Schedule follow-up user tests",
      "priority": 0.8
    }
  ]
}
```
