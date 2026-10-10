# Software Requirements Specification
## Sentiment Analysis Web App (ML + Web)

---

## 1. Project Overview

**Project Name:** Sentiment Analyzer  
**Repository Name:** `nlp-sentiment-analyzer`  
**Purpose:** Bridge ML and web development by creating a deployable AI-powered application  
**Target Users:** Recruiters seeking AI/ML + web integration skills  
**Estimated Development Time:** 5-6 hours

---

## 2. Functional Requirements

### 2.1 Core Features

- **FR1:** User can input text via web interface (textarea)
- **FR2:** Backend API analyzes sentiment using NLP model
- **FR3:** Display sentiment classification (positive/negative/neutral)
- **FR4:** Show confidence score as percentage
- **FR5:** Provide detailed sentiment scores breakdown
- **FR6:** Display visual indicator (color-coded badge)
- **FR7:** Handle edge cases (empty input, very long text)
- **FR8:** Provide example sentences for quick testing
- **FR9:** Show loading state during analysis
- **FR10:** Display error messages for invalid input

### 2.2 User Stories

**As a user, I want to:**
- Paste a product review and instantly see if it's positive or negative
- Understand how confident the model is in its prediction
- Try different sentences without refreshing the page
- See example texts to understand how the tool works
- Get clear feedback when something goes wrong

### 2.3 User Flow

```
1. User opens web app
2. User sees input area with placeholder text
3. User types or pastes text (e.g., "This movie was amazing!")
4. User clicks "Analyze Sentiment" button
5. Loading spinner appears briefly
6. Result displays:
   - Sentiment label with colored badge (e.g., "Positive 😊")
   - Confidence percentage (e.g., "92% confident")
   - Detailed score breakdown (expandable)
7. User can clear input and try another text
8. User can click example buttons to test quickly
```

---

## 3. Technical Requirements

### 3.1 Technology Stack

**Backend:**
- **Python:** 3.10+
- **Framework:** Flask 3.x (lightweight, simple API)
- **NLP Library:** NLTK with VADER (for MVP) OR Hugging Face Transformers
- **CORS:** flask-cors for cross-origin requests
- **Validation:** Built-in Flask validators

**Frontend:**
- **Option A:** Vanilla HTML/CSS/JavaScript (simplest)
- **Option B:** React (consistency with other portfolio projects)
- **Styling:** Tailwind CSS or custom CSS
- **HTTP Client:** Fetch API or Axios

**Deployment:**
- **Backend:** Railway, Render, or Fly.io
- **Frontend:** Vercel or Netlify
- **Environment:** Separate deployments with CORS configured

---

### 3.2 ML Model Selection

#### 3.2.1 Model Comparison

| Model | Pros | Cons | Training Required | Accuracy | Decision |
|-------|------|------|-------------------|----------|----------|
| **VADER (NLTK)** | No training, fast, works well for social media | Less accurate than transformers | ❌ No | ~70% | ✅ MVP |
| **DistilBERT** | High accuracy, pre-trained | Larger, slower, requires transformers | ❌ No (use pre-trained) | ~90% | 🔄 Future |
| **Custom LSTM** | Full control | Requires dataset, training time | ✅ Yes | Variable | ❌ Not MVP |

**Selected for MVP:** VADER (Valence Aware Dictionary and sEntiment Reasoner)

**Rationale:**
- No training required (rule-based + lexicon)
- Fast inference (< 50ms)
- Good performance on general text
- Easy to understand and explain
- Perfect for portfolio demonstration

**Upgrade Path:** Mention DistilBERT in README as future enhancement

---

### 3.3 API Specification

#### 3.3.1 POST `/api/analyze`

**Description:** Analyze sentiment of input text

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "text": "I absolutely loved this product! Best purchase ever!"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "sentiment": "positive",
    "confidence": 0.92,
    "scores": {
      "positive": 0.567,
      "neutral": 0.243,
      "negative": 0.190,
      "compound": 0.8439
    },
    "text_length": 52,
    "processing_time_ms": 15
  }
}
```

**Error Response (400 Bad Request):**
```json
{
  "success": false,
  "error": "Text is required and must be between 1 and 5000 characters",
  "code": "INVALID_INPUT"
}
```

**Error Response (429 Too Many Requests):**
```json
{
  "success": false,
  "error": "Rate limit exceeded. Please try again in 60 seconds.",
  "code": "RATE_LIMIT_EXCEEDED"
}
```

#### 3.3.2 GET `/api/health`

**Description:** Health check endpoint

**Response (200 OK):**
```json
{
  "status": "healthy",
  "model": "VADER",
  "version": "1.0.0"
}
```

#### 3.3.3 GET `/api/examples`

**Description:** Get example sentences for testing

**Response (200 OK):**
```json
{
  "examples": [
    {
      "text": "This is the best day ever!",
      "expected_sentiment": "positive"
    },
    {
      "text": "I'm so disappointed with this service.",
      "expected_sentiment": "negative"
    },
    {
      "text": "The weather is cloudy today.",
      "expected_sentiment": "neutral"
    }
  ]
}
```

---

### 3.4 Sentiment Classification Logic

#### 3.4.1 VADER Implementation

```python
from nltk.sentiment import SentimentIntensityAnalyzer
import nltk

# Download VADER lexicon (one-time)
nltk.download('vader_lexicon')

class SentimentAnalyzer:
    def __init__(self):
        self.analyzer = SentimentIntensityAnalyzer()
    
    def analyze(self, text: str) -> dict:
        """
        Analyze sentiment of text using VADER
        
        Args:
            text: Input text to analyze
            
        Returns:
            Dictionary with sentiment, confidence, and scores
        """
        # Get polarity scores
        scores = self.analyzer.polarity_scores(text)
        
        # Compound score ranges from -1 (most negative) to +1 (most positive)
        compound = scores['compound']
        
        # Classify sentiment based on compound score
        if compound >= 0.05:
            sentiment = 'positive'
            confidence = compound  # 0.05 to 1.0
        elif compound <= -0.05:
            sentiment = 'negative'
            confidence = abs(compound)  # 0.05 to 1.0
        else:
            sentiment = 'neutral'
            confidence = 1.0 - abs(compound)  # Higher when closer to 0
        
        return {
            'sentiment': sentiment,
            'confidence': round(confidence, 4),
            'scores': {
                'positive': round(scores['pos'], 4),
                'neutral': round(scores['neu'], 4),
                'negative': round(scores['neg'], 4),
                'compound': round(scores['compound'], 4)
            }
        }
```

#### 3.4.2 Classification Thresholds

| Compound Score | Sentiment | Reasoning |
|----------------|-----------|-----------|
| >= 0.05 | Positive | Clear positive sentiment |
| -0.05 to 0.05 | Neutral | Ambiguous or balanced |
| <= -0.05 | Negative | Clear negative sentiment |

**Source:** VADER original paper recommendations

---

### 3.5 Input Validation

#### 3.5.1 Validation Rules

```python
def validate_input(text: str) -> tuple[bool, str]:
    """
    Validate input text
    
    Returns:
        (is_valid, error_message)
    """
    if not text:
        return False, "Text is required"
    
    if not isinstance(text, str):
        return False, "Text must be a string"
    
    text = text.strip()
    
    if len(text) == 0:
        return False, "Text cannot be empty or whitespace only"
    
    if len(text) > 5000:
        return False, "Text must be 5000 characters or less"
    
    return True, ""
```

#### 3.5.2 Text Preprocessing

```python
def preprocess_text(text: str) -> str:
    """Clean and prepare text for analysis"""
    # Remove excessive whitespace
    text = ' '.join(text.split())
    
    # VADER handles emojis, punctuation, capitalization automatically
    # No need to remove or normalize
    
    return text
```

---

## 4. Backend Implementation

### 4.1 Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── routes.py              # API endpoints
│   ├── sentiment_model.py     # ML model wrapper
│   ├── validators.py          # Input validation
│   └── config.py              # Configuration
├── tests/
│   ├── test_api.py
│   └── test_sentiment.py
├── requirements.txt
├── .env
├── .env.example
└── run.py                     # Application entry point
```

### 4.2 Flask Application Setup

```python
# app/__init__.py
from flask import Flask
from flask_cors import CORS

def create_app():
    app = Flask(__name__)
    
    # Load configuration
    app.config.from_object('app.config.Config')
    
    # Enable CORS
    CORS(app, resources={
        r"/api/*": {
            "origins": app.config['ALLOWED_ORIGINS'],
            "methods": ["GET", "POST"],
            "allow_headers": ["Content-Type"]
        }
    })
    
    # Register blueprints
    from app.routes import api_bp
    app.register_blueprint(api_bp, url_prefix='/api')
    
    return app
```

### 4.3 API Routes

```python
# app/routes.py
from flask import Blueprint, request, jsonify
from app.sentiment_model import SentimentAnalyzer
from app.validators import validate_input
import time

api_bp = Blueprint('api', __name__)
analyzer = SentimentAnalyzer()

@api_bp.route('/analyze', methods=['POST'])
def analyze_sentiment():
    """Analyze sentiment of input text"""
    start_time = time.time()
    
    # Get request data
    data = request.get_json()
    text = data.get('text', '').strip()
    
    # Validate input
    is_valid, error_msg = validate_input(text)
    if not is_valid:
        return jsonify({
            'success': False,
            'error': error_msg,
            'code': 'INVALID_INPUT'
        }), 400
    
    # Analyze sentiment
    try:
        result = analyzer.analyze(text)
        processing_time = (time.time() - start_time) * 1000  # Convert to ms
        
        return jsonify({
            'success': True,
            'data': {
                **result,
                'text_length': len(text),
                'processing_time_ms': round(processing_time, 2)
            }
        }), 200
        
    except Exception as e:
        return jsonify({
            'success': False,
            'error': 'An error occurred during analysis',
            'code': 'ANALYSIS_ERROR'
        }), 500

@api_bp.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint"""
    return jsonify({
        'status': 'healthy',
        'model': 'VADER',
        'version': '1.0.0'
    }), 200

@api_bp.route('/examples', methods=['GET'])
def get_examples():
    """Get example sentences"""
    examples = [
        {"text": "This is the best day ever!", "expected": "positive"},
        {"text": "I absolutely loved this product! 😍", "expected": "positive"},
        {"text": "Great service and friendly staff.", "expected": "positive"},
        {"text": "I'm so disappointed with this.", "expected": "negative"},
        {"text": "Terrible experience, would not recommend.", "expected": "negative"},
        {"text": "This is the worst thing I've ever bought.", "expected": "negative"},
        {"text": "The weather is cloudy today.", "expected": "neutral"},
        {"text": "The meeting starts at 3pm.", "expected": "neutral"},
        {"text": "It's okay, nothing special.", "expected": "neutral"}
    ]
    
    return jsonify({'examples': examples}), 200
```

### 4.4 Rate Limiting

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["60 per minute"]
)

@api_bp.route('/analyze', methods=['POST'])
@limiter.limit("10 per minute")  # Stricter limit for analysis
def analyze_sentiment():
    # ... implementation
```

---

## 5. Frontend Implementation

### 5.1 Project Structure (Vanilla JS)

```
frontend/
├── index.html
├── css/
│   └── styles.css
├── js/
│   ├── app.js
│   ├── api.js
│   └── ui.js
└── assets/
    └── (images if needed)
```

### 5.2 HTML Structure

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sentiment Analyzer</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>😊 Sentiment Analyzer</h1>
            <p>Analyze the sentiment of any text using AI</p>
        </header>

        <main>
            <!-- Input Section -->
            <div class="input-section">
                <textarea 
                    id="textInput" 
                    placeholder="Enter text to analyze sentiment... (e.g., 'This movie was absolutely amazing!')"
                    rows="6"
                    maxlength="5000"
                ></textarea>
                <div class="character-count">
                    <span id="charCount">0</span> / 5000 characters
                </div>
                <button id="analyzeBtn" class="btn-primary">Analyze Sentiment</button>
            </div>

            <!-- Examples Section -->
            <div class="examples-section">
                <h3>Try these examples:</h3>
                <div class="example-buttons" id="exampleButtons">
                    <!-- Populated by JavaScript -->
                </div>
            </div>

            <!-- Results Section -->
            <div id="results" class="results-section hidden">
                <h3>Analysis Result</h3>
                <div class="sentiment-badge" id="sentimentBadge">
                    <!-- Populated by JavaScript -->
                </div>
                <div class="confidence-bar">
                    <label>Confidence</label>
                    <div class="progress-bar">
                        <div id="confidenceBar" class="progress-fill"></div>
                    </div>
                    <span id="confidenceText">0%</span>
                </div>
                <details class="score-details">
                    <summary>Detailed Scores</summary>
                    <div id="detailedScores">
                        <!-- Populated by JavaScript -->
                    </div>
                </details>
            </div>

            <!-- Loading State -->
            <div id="loading" class="loading hidden">
                <div class="spinner"></div>
                <p>Analyzing sentiment...</p>
            </div>

            <!-- Error Message -->
            <div id="error" class="error-message hidden"></div>
        </main>

        <footer>
            <p>Built with Python (Flask) + VADER NLP</p>
        </footer>
    </div>

    <script src="js/api.js"></script>
    <script src="js/ui.js"></script>
    <script src="js/app.js"></script>
</body>
</html>
```

### 5.3 JavaScript Implementation

```javascript
// js/api.js
const API_BASE_URL = 'http://localhost:5000/api';  // Change for production

async function analyzeSentiment(text) {
    const response = await fetch(`${API_BASE_URL}/analyze`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ text })
    });
    
    const data = await response.json();
    
    if (!response.ok) {
        throw new Error(data.error || 'Analysis failed');
    }
    
    return data.data;
}

async function getExamples() {
    const response = await fetch(`${API_BASE_URL}/examples`);
    const data = await response.json();
    return data.examples;
}
```

```javascript
// js/app.js
document.addEventListener('DOMContentLoaded', async () => {
    const textInput = document.getElementById('textInput');
    const analyzeBtn = document.getElementById('analyzeBtn');
    const charCount = document.getElementById('charCount');
    
    // Character counter
    textInput.addEventListener('input', () => {
        charCount.textContent = textInput.value.length;
    });
    
    // Analyze button
    analyzeBtn.addEventListener('click', async () => {
        const text = textInput.value.trim();
        
        if (!text) {
            showError('Please enter some text to analyze');
            return;
        }
        
        showLoading();
        hideError();
        hideResults();
        
        try {
            const result = await analyzeSentiment(text);
            showResults(result);
        } catch (error) {
            showError(error.message);
        } finally {
            hideLoading();
        }
    });
    
    // Load examples
    try {
        const examples = await getExamples();
        displayExamples(examples);
    } catch (error) {
        console.error('Failed to load examples:', error);
    }
});
```

---

## 6. Styling & UI Design

### 6.1 Color Scheme

```css
:root {
    /* Sentiment Colors */
    --positive: #10B981;      /* Green */
    --negative: #EF4444;      /* Red */
    --neutral: #6B7280;       /* Gray */
    
    /* UI Colors */
    --primary: #3B82F6;       /* Blue */
    --background: #F9FAFB;
    --card-bg: #FFFFFF;
    --text-primary: #111827;
    --text-secondary: #6B7280;
    --border: #E5E7EB;
}
```

### 6.2 Responsive Design

```css
/* Mobile First */
.container {
    max-width: 100%;
    padding: 1rem;
}

/* Tablet */
@media (min-width: 768px) {
    .container {
        max-width: 720px;
        margin: 0 auto;
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .container {
        max-width: 900px;
    }
}
```

---

## 7. Non-Functional Requirements

### 7.1 Performance

- **API Response Time:** < 200ms for typical sentences
- **UI Responsiveness:** Immediate feedback on button clicks
- **Text Limit:** 5000 characters (reasonable for analysis)
- **Rate Limiting:** 10 requests/minute per IP (prevent abuse)

### 7.2 Usability

- **Mobile Responsive:** Works on 320px+ screens
- **Clear Feedback:** Loading states, success/error messages
- **Visual Indicators:** Color-coded sentiment badges
- **Accessibility:** 
  - Keyboard navigation
  - ARIA labels
  - Sufficient contrast
  - Screen reader friendly

### 7.3 Security

- **Input Validation:** Both frontend and backend
- **XSS Prevention:** Sanitize user input before display
- **CORS:** Configured for specific origins in production
- **Rate Limiting:** Prevent API abuse
- **No Sensitive Data:** No user tracking or data storage

---

## 8. Testing Strategy

### 8.1 Backend Unit Tests

```python
# tests/test_sentiment.py
import unittest
from app.sentiment_model import SentimentAnalyzer

class TestSentimentAnalyzer(unittest.TestCase):
    def setUp(self):
        self.analyzer = SentimentAnalyzer()
    
    def test_positive_sentiment(self):
        result = self.analyzer.analyze("I love this!")
        self.assertEqual(result['sentiment'], 'positive')
        self.assertGreater(result['confidence'], 0.5)
    
    def test_negative_sentiment(self):
        result = self.analyzer.analyze("This is terrible.")
        self.assertEqual(result['sentiment'], 'negative')
        self.assertGreater(result['confidence'], 0.5)
    
    def test_neutral_sentiment(self):
        result = self.analyzer.analyze("The sky is blue.")
        self.assertEqual(result['sentiment'], 'neutral')
    
    def test_emoji_handling(self):
        result = self.analyzer.analyze("Great product! 😊")
        self.assertEqual(result['sentiment'], 'positive')
```

### 8.2 API Integration Tests

```python
# tests/test_api.py
import unittest
from app import create_app

class TestAPI(unittest.TestCase):
    def setUp(self):
        self.app = create_app()
        self.client = self.app.test_client()
    
    def test_analyze_endpoint(self):
        response = self.client.post('/api/analyze',
            json={'text': 'I love this!'},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
        self.assertTrue(data['success'])
        self.assertIn('sentiment', data['data'])
    
    def test_empty_text_validation(self):
        response = self.client.post('/api/analyze',
            json={'text': ''},
            content_type='application/json'
        )
        self.assertEqual(response.status_code, 400)
    
    def test_health_endpoint(self):
        response = self.client.get('/api/health')
        self.assertEqual(response.status_code, 200)
```

### 8.3 Manual Testing Checklist

**Functionality:**
- [ ] Positive text correctly identified
- [ ] Negative text correctly identified
- [ ] Neutral text correctly identified
- [ ] Emojis handled properly
- [ ] Long text (5000 chars) works
- [ ] Empty text shows error
- [ ] Example buttons populate text
- [ ] Confidence score displays correctly

**UI/UX:**
- [ ] Responsive on mobile (375px)
- [ ] Responsive on tablet (768px)
- [ ] Responsive on desktop (1920px)
- [ ] Loading spinner shows during analysis
- [ ] Error messages clear and helpful
- [ ] Color coding intuitive (green=positive, red=negative)
- [ ] Keyboard navigation works

**Edge Cases:**
- [ ] Very short text (1 word)
- [ ] Text with only emojis
- [ ] Text with special characters
- [ ] Mixed sentiment text
- [ ] Rate limit triggers correctly

---

## 9. Deployment

### 9.1 Backend Deployment (Railway)

```bash
# 1. Install Railway CLI
npm install -g @railway/cli

# 2. Login
railway login

# 3. Initialize project
railway init

# 4. Deploy
railway up

# 5. Set environment variables in Railway dashboard
# FLASK_ENV=production
# ALLOWED_ORIGINS=https://your-frontend.vercel.app
```

**Procfile:**
```
web: gunicorn run:app
```

**requirements.txt:**
```
Flask==3.0.0
flask-cors==4.0.0
flask-limiter==3.5.0
nltk==3.8.1
gunicorn==21.2.0
```

### 9.2 Frontend Deployment (Vercel)

```bash
# 1. Install Vercel CLI
npm install -g vercel

# 2. Deploy
vercel --prod

# 3. Set environment variable
# API_BASE_URL=https://your-backend.railway.app/api
```

---

## 10. Out of Scope (Future Enhancements)

- ❌ User accounts and history
- ❌ Batch processing (analyze multiple texts)
- ❌ Advanced NLP models (DistilBERT, RoBERTa)
- ❌ Multi-language support
- ❌ Aspect-based sentiment (e.g., "food good, service bad")
- ❌ Emotion detection (joy, anger, sadness)
- ❌ Sarcasm detection
- ❌ Sentiment trends over time
- ❌ API authentication
- ❌ Export results to CSV

---

## 11. Success Criteria

✅ API correctly classifies 85%+ of test sentences  
✅ Frontend is clean and responsive  
✅ Error handling prevents crashes  
✅ Confidence scores are intuitive  
✅ Deployed and accessible via public URL  
✅ README clearly explains ML approach  
✅ Code demonstrates ML + web integration  
✅ CORS configured correctly for production

---

## 12. README Requirements

1. **Project Overview:** Problem and solution
2. **Live Demo:** Links to deployed app
3. **Screenshots:** Desktop and mobile views
4. **Tech Stack:** Both frontend and backend
5. **ML Approach:** Why VADER, how it works
6. **Features:** What users can do
7. **Architecture:** Diagram showing flow
8. **Setup Instructions:**
   - Backend setup
   - Frontend setup
   - Environment variables
9. **API Documentation:** Endpoints with examples
10. **Testing:** How to run tests
11. **Deployment:** How it's deployed
12. **Future Improvements:** DistilBERT upgrade, etc.

---

## 13. Portfolio Value

**Signal This Sends:**
> "I can integrate machine learning models into production web applications, handle real-time predictions, and create user-friendly AI interfaces."

**Key Talking Points for Interviews:**
- Why you chose VADER over transformer models
- How you handle API rate limiting
- Your approach to error handling across the stack
- CORS configuration and security
- How you would improve accuracy (upgrade to DistilBERT)
- Scaling considerations (caching, model serving)
- Real-time vs batch processing trade-offs
- Handling model updates without downtime

---

## 14. Example API Usage

### cURL

```bash
# Analyze sentiment
curl -X POST http://localhost:5000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"text": "I love this product!"}'

# Health check
curl http://localhost:5000/api/health

# Get examples
curl http://localhost:5000/api/examples
```

### JavaScript

```javascript
const response = await fetch('http://localhost:5000/api/analyze', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ text: 'I love this product!' })
});

const data = await response.json();
console.log(data.data.sentiment);  // "positive"
console.log(data.data.confidence); // 0.92
```

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Status:** Ready for Implementation
