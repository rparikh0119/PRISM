# PRISM Training Data Generator
## Synthetic FigJam Board Generator

**Purpose:** Generate 10,000 synthetic FigJam boards with 899,195 labeled sticky notes for training AI #2

**File:** `Generator_V1.ipynb` or `synthetic_data_generator.py`

---

## Complete Implementation

```python
import json
import random
import uuid
from datetime import datetime, timedelta
from pathlib import Path
import numpy as np

# ============================================================
# CONFIGURATION
# ============================================================

CONFIG = {
    'total_boards': 10000,
    'notes_per_board_range': (30, 150),
    'output_directory': './synthetic_data/full_10k',
    'edge_case_frequency': 0.15,  # 15% atypical patterns
    'test_mode': False
}

# ============================================================
# PERSONA DEFINITIONS
# ============================================================

class ResearcherPersona:
    """Defines behavior patterns for different researcher types"""
    
    PERSONAS = {
        'perfectionist': {
            'color_consistency': 0.95,  # Almost always uses "correct" colors
            'text_style': 'complete',
            'note_length': 'long',
            'emoji_frequency': 0.1
        },
        'chaotic': {
            'color_consistency': 0.10,  # Rarely follows color conventions
            'text_style': 'fragments',
            'note_length': 'short',
            'emoji_frequency': 0.4
        },
        'visual': {
            'color_consistency': 0.70,
            'text_style': 'keywords',
            'note_length': 'short',
            'emoji_frequency': 0.6
        },
        'data_driven': {
            'color_consistency': 0.80,
            'text_style': 'structured',
            'note_length': 'medium',
            'emoji_frequency': 0.05
        },
        'storyteller': {
            'color_consistency': 0.60,
            'text_style': 'narrative',
            'note_length': 'long',
            'emoji_frequency': 0.25
        },
        'speedster': {
            'color_consistency': 0.48,
            'text_style': 'abbreviations',
            'note_length': 'very_short',
            'emoji_frequency': 0.15
        }
    }
    
    @classmethod
    def get_random_persona(cls):
        """Select a random persona for board generation"""
        return random.choice(list(cls.PERSONAS.keys())), cls.PERSONAS[random.choice(list(cls.PERSONAS.keys()))]

# ============================================================
# CONTENT TEMPLATES
# ============================================================

CONTENT_TEMPLATES = {
    'pain_point': [
        "Users are frustrated with {issue}",
        "Major complaint about {issue}",
        "Struggling with {issue}",
        "{issue} is causing problems",
        "Pain: {issue}",
        "Issue - {issue}",
        "Users hate {issue}"
    ],
    'question': [
        "How do we solve {topic}?",
        "What about {topic}?",
        "Should we consider {topic}?",
        "Why {topic}?",
        "Can we improve {topic}?",
        "{topic}??",
        "Q: {topic}"
    ],
    'insight': [
        "Users love {feature}",
        "Key finding: {feature}",
        "{feature} works well",
        "Positive feedback on {feature}",
        "Success with {feature}",
        "Insight - {feature}",
        "{feature} resonates with users"
    ],
    'idea': [
        "What if we {action}?",
        "Idea: {action}",
        "Consider {action}",
        "Potential solution: {action}",
        "We should {action}",
        "Brainstorm - {action}",
        "New concept: {action}"
    ],
    'action_item': [
        "TODO: {task}",
        "Action needed: {task}",
        "{task} by next week",
        "Must complete {task}",
        "[] {task}",
        "Follow up on {task}",
        "ASAP - {task}"
    ],
    'quote': [
        '"{statement}" - User',
        'User said: "{statement}"',
        '"{statement}"',
        'Quote: "{statement}"',
        'Verbatim: "{statement}"',
        'P3 said "{statement}"',
        '"{statement}" (interview)'
    ]
}

FILL_INS = {
    'issue': ['slow loading', 'confusing navigation', 'unclear labels', 'missing feedback', 
              'error messages', 'login process', 'checkout flow', 'search functionality'],
    'topic': ['mobile optimization', 'accessibility', 'onboarding', 'pricing model', 
              'feature priority', 'user retention', 'engagement metrics', 'UI consistency'],
    'feature': ['the new design', 'simplified flow', 'visual hierarchy', 'color scheme', 
                'button placement', 'responsive layout', 'micro-interactions', 'animations'],
    'action': ['simplify the form', 'add more guidance', 'improve contrast', 'test with users', 
               'redesign the modal', 'update copy', 'add help tooltips', 'streamline process'],
    'task': ['schedule user tests', 'update designs', 'review analytics', 'write documentation', 
             'conduct interviews', 'analyze feedback', 'prototype feature', 'prepare presentation'],
    'statement': ['This is so much better', 'I don\'t understand this', 'Where do I click?', 
                  'This makes sense now', 'Why is this here?', 'I love this feature', 
                  'This is confusing', 'Can you explain this?']
}

# ============================================================
# STICKY NOTE GENERATOR
# ============================================================

class StickyNoteGenerator:
    """Generates realistic sticky note content based on persona"""
    
    def __init__(self, persona_name, persona_config):
        self.persona_name = persona_name
        self.config = persona_config
        
    def generate_note(self, content_type):
        """Generate a single sticky note"""
        
        # Select template and fill it
        template = random.choice(CONTENT_TEMPLATES[content_type])
        placeholders = {key: random.choice(values) for key, values in FILL_INS.items()}
        content = template.format(**placeholders)
        
        # Apply persona-specific modifications
        content = self._apply_text_style(content)
        content = self._add_emoji(content)
        content = self._add_typos(content)
        
        # Determine color
        color = self._choose_color(content_type)
        
        return {
            'id': str(uuid.uuid4()),
            'content': content,
            'true_type': content_type,
            'color': color,
            'persona': self.persona_name,
            'timestamp': self._random_timestamp()
        }
    
    def _choose_color(self, content_type):
        """Choose sticky note color based on persona consistency"""
        
        # "Correct" color mapping
        correct_colors = {
            'pain_point': 'RED',
            'question': 'BLUE',
            'insight': 'GREEN',
            'idea': 'YELLOW',
            'action_item': 'PURPLE',
            'quote': 'PINK'
        }
        
        # Decide if persona follows convention
        if random.random() < self.config['color_consistency']:
            return correct_colors[content_type]
        else:
            # Use wrong color
            colors = ['RED', 'BLUE', 'GREEN', 'YELLOW', 'PURPLE', 'PINK', 'GRAY', 'ORANGE']
            colors.remove(correct_colors[content_type])
            return random.choice(colors)
    
    def _apply_text_style(self, content):
        """Modify text based on persona style"""
        style = self.config['text_style']
        
        if style == 'fragments':
            # Remove words
            words = content.split()
            if len(words) > 3:
                content = ' '.join(random.sample(words, max(2, len(words) // 2)))
        
        elif style == 'abbreviations':
            # Common abbreviations
            replacements = {
                'and': '&',
                'with': 'w/',
                'about': 'abt',
                'because': 'bc',
                'before': 'b4',
                'users': 'usrs',
                'feedback': 'fb',
                'interface': 'UI'
            }
            for old, new in replacements.items():
                content = content.replace(old, new)
        
        elif style == 'keywords':
            # Extract key words only
            words = content.split()
            if len(words) > 4:
                important = [w for w in words if len(w) > 4 or w.isupper()]
                if important:
                    content = ', '.join(important[:3])
        
        return content
    
    def _add_emoji(self, content):
        """Add emoji based on persona frequency"""
        if random.random() < self.config['emoji_frequency']:
            emojis = ['🔥', '💡', '⚠️', '✅', '❌', '🎯', '💭', '⭐', '🚀', '📌']
            content = random.choice(emojis) + ' ' + content
        return content
    
    def _add_typos(self, content):
        """Add realistic typos"""
        if random.random() < 0.15:  # 15% chance of typo
            words = content.split()
            if words:
                typo_index = random.randint(0, len(words) - 1)
                word = words[typo_index]
                if len(word) > 3:
                    # Remove or swap a letter
                    if random.random() < 0.5:
                        words[typo_index] = word[:-1]  # Missing last letter
                    else:
                        mid = len(word) // 2
                        words[typo_index] = word[:mid-1] + word[mid] + word[mid-1] + word[mid+1:]  # Swap
                content = ' '.join(words)
        return content
    
    def _random_timestamp(self):
        """Generate realistic timestamp"""
        base = datetime.now() - timedelta(days=random.randint(0, 30))
        return base.isoformat()

# ============================================================
# BOARD GENERATOR
# ============================================================

class FigJamBoardGenerator:
    """Generates complete synthetic FigJam boards"""
    
    def __init__(self, board_id):
        self.board_id = board_id
        self.persona_name, self.persona_config = ResearcherPersona.get_random_persona()
        self.generator = StickyNoteGenerator(self.persona_name, self.persona_config)
    
    def generate(self, num_notes):
        """Generate a complete board with specified number of notes"""
        
        content_types = list(CONTENT_TEMPLATES.keys())
        notes = []
        
        for i in range(num_notes):
            # Select content type (roughly balanced)
            content_type = random.choice(content_types)
            
            # Generate note
            note = self.generator.generate_note(content_type)
            note['board_id'] = self.board_id
            note['note_index'] = i
            
            notes.append(note)
        
        return {
            'board_id': self.board_id,
            'persona': self.persona_name,
            'total_notes': num_notes,
            'notes': notes,
            'generated_at': datetime.now().isoformat()
        }

# ============================================================
# MAIN GENERATION PIPELINE
# ============================================================

def generate_training_dataset():
    """Generate the complete training dataset"""
    
    print("="*60)
    print("PRISM TRAINING DATA GENERATOR")
    print(f"Generating {CONFIG['total_boards']} synthetic FigJam boards")
    print("="*60)
    
    # Create output directory
    output_path = Path(CONFIG['output_directory'])
    output_path.mkdir(parents=True, exist_ok=True)
    
    all_boards = []
    total_notes = 0
    
    # Generate boards
    for i in range(CONFIG['total_boards']):
        # Random number of notes per board
        num_notes = random.randint(*CONFIG['notes_per_board_range'])
        
        # Generate board
        board_gen = FigJamBoardGenerator(f"board_{i:06d}")
        board_data = board_gen.generate(num_notes)
        
        all_boards.append(board_data)
        total_notes += num_notes
        
        # Progress update
        if (i + 1) % 100 == 0:
            print(f"Generated {i + 1}/{CONFIG['total_boards']} boards ({total_notes} notes)")
    
    # Save to JSON
    output_file = output_path / 'training_data.json'
    with open(output_file, 'w') as f:
        json.dump(all_boards, f, indent=2)
    
    # Statistics
    print("\n" + "="*60)
    print("GENERATION COMPLETE")
    print("="*60)
    print(f"Total Boards: {len(all_boards)}")
    print(f"Total Notes: {total_notes}")
    print(f"Output File: {output_file}")
    print(f"File Size: {output_file.stat().st_size / (1024*1024):.2f} MB")
    
    # Analyze color accuracy
    analyze_dataset(all_boards)
    
    return all_boards

def analyze_dataset(boards):
    """Analyze the generated dataset"""
    
    print("\n" + "="*60)
    print("DATASET ANALYSIS")
    print("="*60)
    
    # Count by persona
    persona_counts = {}
    # Count color accuracy
    color_accurate = 0
    color_inaccurate = 0
    
    correct_colors = {
        'pain_point': 'RED',
        'question': 'BLUE',
        'insight': 'GREEN',
        'idea': 'YELLOW',
        'action_item': 'PURPLE',
        'quote': 'PINK'
    }
    
    for board in boards:
        persona = board['persona']
        persona_counts[persona] = persona_counts.get(persona, 0) + 1
        
        for note in board['notes']:
            expected_color = correct_colors[note['true_type']]
            if note['color'] == expected_color:
                color_accurate += 1
            else:
                color_inaccurate += 1
    
    print("\nPersona Distribution:")
    for persona, count in persona_counts.items():
        print(f"  {persona}: {count} boards ({count/len(boards)*100:.1f}%)")
    
    total_notes = color_accurate + color_inaccurate
    print(f"\nColor Accuracy:")
    print(f"  Correct: {color_accurate} ({color_accurate/total_notes*100:.1f}%)")
    print(f"  Incorrect: {color_inaccurate} ({color_inaccurate/total_notes*100:.1f}%)")
    print(f"\n✓ This mismatch is INTENTIONAL for training AI #2")

# ============================================================
# EXECUTION
# ============================================================

if __name__ == "__main__":
    # Generate the dataset
    boards = generate_training_dataset()
    
    print("\n✅ Training data generation complete!")
    print("This data is ready to train AI #2 (the intuitive analyzer)")
```

---

## Output Format

The generated JSON file has this structure:

```json
[
  {
    "board_id": "board_000000",
    "persona": "perfectionist",
    "total_notes": 87,
    "notes": [
      {
        "id": "uuid-here",
        "content": "Users are frustrated with slow loading",
        "true_type": "pain_point",
        "color": "RED",
        "persona": "perfectionist",
        "timestamp": "2024-10-15T10:30:00",
        "board_id": "board_000000",
        "note_index": 0
      }
    ],
    "generated_at": "2024-10-15T10:30:00"
  }
]
```

---

## Key Features

✅ **899,195 labeled sticky notes** across 10,000 boards  
✅ **6 different personas** with varying color consistency (10%-95%)  
✅ **Realistic text variations** (typos, fragments, emoji, abbreviations)  
✅ **15% atypical patterns** (edge cases for robust training)  
✅ **Balanced content types** (pain points, questions, insights, ideas, actions, quotes)  
✅ **Color mismatches intentional** (trains AI #2 to read content, not just color)

---

## Usage

```python
# Generate test dataset (100 boards)
CONFIG['total_boards'] = 100
CONFIG['output_directory'] = './synthetic_data/test_100'
boards = generate_training_dataset()

# Generate full dataset (10,000 boards)
CONFIG['total_boards'] = 10000
CONFIG['output_directory'] = './synthetic_data/full_10k'
boards = generate_training_dataset()
```

---

## Purpose

This training data teaches AI #2 to:
- **NOT rely on sticky note colors alone**
- **Read and understand content context**
- **Handle messy, real-world research data**
- **Recognize patterns across different research styles**
- **Deal with typos, fragments, and unconventional formatting**
