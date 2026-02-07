
# Public Help Platform – System Design Document

## 1. System Overview

### 1.1 Introduction
The Public Help Platform is a web-based guidance system that assists users by analyzing searched or uploaded government documents and providing language-specific video guidance. The platform is designed with a mobile-first, accessibility-focused approach to serve rural and low-literacy users across India.

### 1.2 Design Philosophy
- **Simplicity First**: Minimize cognitive load with clear visual hierarchy
- **Accessibility**: Support for multiple languages, voice input, and low-bandwidth scenarios
- **Privacy by Design**: No permanent storage of user documents
- **Progressive Enhancement**: Core functionality works on basic devices, enhanced features on modern devices

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Web App    │  │  Mobile Web  │  │   PWA        │      │
│  │  (Desktop)   │  │  (Responsive)│  │  (Future)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway / CDN                       │
│              (Rate Limiting, Caching, SSL)                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Search     │  │   Upload     │  │   Video      │      │
│  │   Service    │  │   Service    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Document    │  │    Voice     │  │  Analytics   │      │
│  │  Classifier  │  │    Service   │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Document   │  │    Video     │  │   Analytics  │      │
│  │   Database   │  │     CDN      │  │   Database   │      │
│  │  (Metadata)  │  │   (Storage)  │  │   (Logs)     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    External Services                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │     OCR      │  │  Speech-to-  │  │   Malware    │      │
│  │   Service    │  │     Text     │  │   Scanner    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

**Frontend**:
- HTML5, CSS3 (with CSS Grid/Flexbox)
- Vanilla JavaScript (ES6+) or lightweight framework (React/Vue)
- Responsive design framework (Bootstrap/Tailwind)
- Progressive Web App capabilities

**Backend**:
- Node.js with Express.js OR Python with FastAPI
- RESTful API architecture
- JWT for session management (if needed)

**Database**:
- PostgreSQL for document metadata and analytics
- Redis for caching and session storage

**File Processing**:
- Tesseract OCR for text extraction from images
- PDF.js or PyPDF2 for PDF text extraction
- Sharp or Pillow for image optimization

**AI/ML**:
- TF-IDF or BERT for document classification
- Keyword matching with fuzzy search (Levenshtein distance)
- Pre-trained models for multilingual text processing

**Voice Processing**:
- Web Speech API (browser-native) for client-side processing
- Google Cloud Speech-to-Text or Azure Speech Services (fallback)

**Video Delivery**:
- AWS S3 + CloudFront OR Azure Blob + CDN
- HLS (HTTP Live Streaming) for adaptive bitrate
- Video compression: H.264 codec, multiple quality levels

**Infrastructure**:
- Docker containers for microservices
- Kubernetes or AWS ECS for orchestration
- Nginx as reverse proxy and load balancer
- CloudWatch/Prometheus for monitoring

## 3. Detailed Component Design

### 3.1 Frontend Architecture

#### 3.1.1 Page Structure
```
/
├── index.html (Homepage)
│ ├── Language selector
│ ├── Search bar (text + voice)
│ ├── Upload button
│ ├── Popular documents grid
│ └── Disclaimer banner
│
├── search-results.html
│ ├── Search query display
│ ├── Results list
│ └── Refinement options
│
├── document-details.html
│ ├── Document name and category
│ ├── Video player
│ ├── Step-by-step instructions
│ ├── Required documents checklist
│ └── Official portal links
│
└── upload-result.html
├── Identified document (or top 3 matches)
├── Confidence indicator
└── Proceed to guidance button
```

#### 3.1.2 UI Components
- **Language Selector**: Dropdown with flag icons, persistent selection
- **Search Bar**: Autocomplete, voice button, clear button
- **Document Card**: Icon, name, category badge, tap target
- **Video Player**: Custom controls, subtitle toggle, quality selector
- **Upload Zone**: Drag-and-drop, file browser, format indicators
- **Loading States**: Skeleton screens, progress bars, spinners
- **Error Messages**: Friendly language, retry options, help links

#### 3.1.3 Responsive Breakpoints
- Mobile: 320px - 767px (single column)
- Tablet: 768px - 1023px (two columns)
- Desktop: 1024px+ (three columns with sidebar)

### 3.2 Backend API Design

#### 3.2.1 API Endpoints

**Search API**:
```
GET /api/v1/search?q={query}&lang={language}&limit={number}
Response: {
"results": [
{
"id": "doc_123",
"name": "PAN Card Application",
"name_local": "పాన్ కార్డ్ దరఖాస్తు",
"category": "identity",
"keywords": ["pan", "permanent account number"],
"relevance_score": 0.95
}
],
"total": 1,
"query_time_ms": 45
}
```

**Voice Search API**:
```
POST /api/v1/voice-search
Body: {
"audio": "base64_encoded_audio",
"language": "te-IN"
}
Response: {
"transcription": "పాన్ కార్డ్",
"confidence": 0.92,
"search_results": [...]
}
```

**Upload API**:
```
POST /api/v1/upload
Body: multipart/form-data with file
Response: {
"upload_id": "temp_xyz789",
"file_size": 2048576,
"file_type": "application/pdf",
"status": "processing"
}
```

**Document Identification API**:
```
POST /api/v1/identify
Body: {
"upload_id": "temp_xyz789"
}
Response: {
"identified": true,
"confidence": 0.87,
"document": {
"id": "doc_123",
"name": "PAN Card Application",
"name_local": "పాన్ కార్డ్ దరఖాస్తు"
},
"alternatives": [
{"id": "doc_124", "name": "PAN Card Correction", "confidence": 0.65}
]
}
```

**Document Details API**:
```
GET /api/v1/documents/{doc_id}?lang={language}
Response: {
"id": "doc_123",
"name": "PAN Card Application",
"name_local": "పాన్ కార్డ్ దరఖాస్తు",
"category": "identity",
"description": "Apply for new PAN card...",
"video_url": "https://cdn.example.com/videos/pan_te.m3u8",
"video_duration": 285,
"instructions": [
{"step": 1, "text": "Fill Form 49A..."},
{"step": 2, "text": "Attach proof of identity..."}
],
"required_documents": ["Aadhaar", "Address Proof"],
"official_portal": "https://www.incometax.gov.in/",
"estimated_time": "15 minutes"
}
```

**Popular Documents API**:
```
GET /api/v1/documents/popular?lang={language}&limit=10
Response: {
"documents": [
{
"id": "doc_123",
"name": "PAN Card Application",
"name_local": "పాన్ కార్డ్ దరఖాస్తు",
"category": "identity",
"view_count": 15420
}
]
}
```

**Analytics API** (Internal):
```
POST /api/v1/analytics/event
Body: {
"event_type": "video_view",
"document_id": "doc_123",
"language": "te",
"timestamp": "2026-02-07T10:30:00Z",
"session_id": "sess_abc123"
}
```

### 3.3 Document Classification System

#### 3.3.1 Classification Pipeline
```
Input Document (PDF/Image)
↓
Text Extraction (OCR/PDF Parser)
↓
Preprocessing (Lowercase, Remove Special Chars)
↓
Keyword Extraction (TF-IDF)
↓
Fuzzy Matching Against Document Database
↓
Confidence Scoring
↓
Return Top Match(es)
```

#### 3.3.2 Document Database Schema
```sql
CREATE TABLE documents (
id VARCHAR(50) PRIMARY KEY,
name_en VARCHAR(255) NOT NULL,
name_hi VARCHAR(255),
name_te VARCHAR(255),
category VARCHAR(50),
department VARCHAR(100),
keywords TEXT[], -- Array of keywords
description_en TEXT,
description_hi TEXT,
description_te TEXT,
official_url VARCHAR(500),
created_at TIMESTAMP,
updated_at TIMESTAMP
);

CREATE TABLE document_videos (
id SERIAL PRIMARY KEY,
document_id VARCHAR(50) REFERENCES documents(id),
language VARCHAR(5), -- 'en', 'hi', 'te'
video_url VARCHAR(500),
subtitle_url VARCHAR(500),
duration_seconds INT,
thumbnail_url VARCHAR(500),
created_at TIMESTAMP
);

CREATE TABLE document_instructions (
id SERIAL PRIMARY KEY,
document_id VARCHAR(50) REFERENCES documents(id),
language VARCHAR(5),
step_number INT,
instruction_text TEXT,
created_at TIMESTAMP
);
```

#### 3.3.3 Keyword Matching Algorithm
```python
def identify_document(extracted_text, documents_db):
# Preprocess extracted text
text_clean = preprocess(extracted_text)

# Extract keywords using TF-IDF
keywords = extract_keywords(text_clean)

# Calculate similarity scores
scores = []
for doc in documents_db:
similarity = calculate_similarity(keywords, doc.keywords)
scores.append((doc.id, similarity))

# Sort by score
scores.sort(key=lambda x: x[1], reverse=True)

# Return top match if confidence > threshold
if scores[0][1] > 0.70:
return {
"document_id": scores[0][0],
"confidence": scores[0][1],
"alternatives": scores[1:4] # Top 3 alternatives
}
else:
return {
"document_id": None,
"confidence": scores[0][1],
"alternatives": scores[0:3]
}
```

### 3.4 Voice Search Implementation

#### 3.4.1 Client-Side (Web Speech API)
```javascript
// Browser-native speech recognition
const recognition = new webkitSpeechRecognition();
recognition.lang = 'te-IN'; // Telugu
recognition.continuous = false;
recognition.interimResults = false;

recognition.onresult = (event) => {
const transcript = event.results[0][0].transcript;
const confidence = event.results[0][0].confidence;

// Send to search API
searchDocuments(transcript);
};

recognition.start();
```

#### 3.4.2 Server-Side Fallback
For browsers without Web Speech API support:
- Record audio on client (Web Audio API)
- Send audio blob to server
- Use Google Cloud Speech-to-Text API
- Return transcription to client

### 3.5 Video Delivery System

#### 3.5.1 Video Encoding Strategy
Each video is encoded in multiple qualities:
- **Low**: 360p, 500 kbps (for 2G/3G)
- **Medium**: 480p, 1 Mbps (for 4G)
- **High**: 720p, 2.5 Mbps (for WiFi/5G)

#### 3.5.2 Adaptive Streaming (HLS)
```
video_pan_te/
├── master.m3u8 (playlist)
├── 360p.m3u8
├── 480p.m3u8
├── 720p.m3u8
└── segments/
├── 360p_001.ts
├── 360p_002.ts
└── ...
```

#### 3.5.3 Video Player Implementation
```javascript
// Using HLS.js for adaptive streaming
const video = document.getElementById('video-player');
const hls = new Hls({
startLevel: -1, // Auto-select quality
maxBufferLength: 10, // 10 seconds buffer
maxMaxBufferLength: 20
});

hls.loadSource(videoUrl);
hls.attachMedia(video);

// Track playback for analytics
video.addEventListener('ended', () => {
sendAnalytics('video_completed', documentId);
});
```

### 3.6 Multilingual Support

#### 3.6.1 Translation Strategy
- All UI strings stored in JSON files per language
- Server-side rendering with language parameter
- Client-side language switching without page reload

```javascript
// i18n structure
{
"en": {
"search_placeholder": "Search for government documents...",
"upload_button": "Upload Document",
"popular_documents": "Popular Documents"
},
"te": {
"search_placeholder": "ప్రభుత్వ పత్రాలను శోధించండి...",
"upload_button": "పత్రాన్ని అప్‌లోడ్ చేయండి",
"popular_documents": "ప్రసిద్ధ పత్రాలు"
},
"hi": {
"search_placeholder": "सरकारी दस्तावेज़ खोजें...",
"upload_button": "दस्तावेज़ अपलोड करें",
"popular_documents": "लोकप्रिय दस्तावेज़"
}
}
```

## 4. User Interaction Flows

### 4.1 Search-Based Flow (Detailed)
```
1. User lands on homepage
↓
2. System detects browser language → Sets default language
↓
3. User sees search bar + popular documents
↓
4. User types "pan card" OR clicks microphone for voice
↓
5. [If voice] System converts speech to text → Displays transcription
↓
6. System searches database with fuzzy matching
↓
7. Results displayed within 2 seconds
↓
8. User clicks on "PAN Card Application"
↓
9. System loads document details page
↓
10. Video player loads with selected language
↓
11. User watches video + reads instructions
↓
12. User clicks "Go to Official Portal" button
↓
13. User redirected to government website
```

### 4.2 Upload-Based Flow (Detailed)
```
1. User lands on homepage
↓
2. User clicks "Upload Document" button
↓
3. File picker opens (or drag-and-drop zone)
↓
4. User selects PDF/image file
↓
5. System validates file (size, format)
↓
6. [If invalid] Error message + retry option
↓
7. [If valid] Upload progress bar shown
↓
8. File uploaded to server (temp storage)
↓
9. System extracts text (OCR for images, parser for PDF)
↓
10. System runs classification algorithm
↓
11. [If confidence > 70%] Single match shown
[If confidence < 70%] Top 3 matches shown for user selection
↓
12. User confirms document OR selects from alternatives
↓
13. User selects preferred language
↓
14. System loads document details page with video
↓
15. Uploaded file deleted from server immediately
```

### 4.3 Error Handling Flows

**Network Error**:
```
User action → Network request fails
↓
Show friendly error: "Connection issue. Please check your internet."
↓
Provide "Retry" button
↓
Log error for monitoring
```

**Document Not Identified**:
```
Upload → Classification confidence < 50%
↓
Show message: "We couldn't identify this document. Try searching by name."
↓
Provide search bar
↓
Show popular documents as alternatives
```

**Video Load Failure**:
```
Video player fails to load
↓
Show error: "Video unavailable. Please try again later."
↓
Display text instructions as fallback
↓
Provide "Report Issue" button
```

## 5. Security & Privacy Design

### 5.1 Document Upload Security
- **File Validation**: Check file type, size, and magic bytes
- **Malware Scanning**: Scan uploaded files with ClamAV or similar
- **Sandboxed Processing**: Process files in isolated containers
- **Immediate Deletion**: Delete files after identification (max 5 minutes retention)
- **No Logging**: Do not log file contents or extracted text

### 5.2 Data Privacy
- **No User Accounts**: No registration required, no user data collected
- **Anonymous Analytics**: Track usage patterns without PII
- **Session IDs**: Temporary, non-identifiable session tokens
- **HTTPS Only**: All traffic encrypted with TLS 1.3
- **No Third-Party Tracking**: No Google Analytics, Facebook Pixel, etc.

### 5.3 Rate Limiting
```
Per IP Address:
- Search: 100 requests per minute
- Upload: 10 requests per hour
- Voice Search: 20 requests per hour
- API calls: 200 requests per minute
```

### 5.4 Content Security Policy
```
Content-Security-Policy:
default-src 'self';
script-src 'self' 'unsafe-inline';
style-src 'self' 'unsafe-inline';
img-src 'self' data: https://cdn.example.com;
media-src 'self' https://cdn.example.com;
connect-src 'self' https://api.example.com;
```

## 6. Performance Optimization

### 6.1 Frontend Optimization
- **Code Splitting**: Load JavaScript modules on-demand
- **Image Optimization**: WebP format with fallbacks, lazy loading
- **CSS Minification**: Compress and bundle CSS
- **Caching Strategy**: Service Worker for offline capability
- **Font Optimization**: Subset fonts, use system fonts as fallback

### 6.2 Backend Optimization
- **Database Indexing**: Index on document keywords, category, language
- **Query Caching**: Redis cache for popular searches (TTL: 1 hour)
- **Connection Pooling**: Reuse database connections
- **Async Processing**: Non-blocking I/O for file uploads

### 6.3 CDN Strategy
- **Static Assets**: Serve CSS, JS, images from CDN
- **Video Delivery**: Use CDN with edge locations across India
- **Cache Headers**: Set appropriate cache-control headers
- **Compression**: Enable Gzip/Brotli compression

### 6.4 Low-Bandwidth Optimization
- **Progressive Loading**: Load critical content first
- **Image Placeholders**: Show low-res placeholders while loading
- **Video Quality Selection**: Default to low quality on slow connections
- **Reduced Animations**: Minimize animations on slow devices

## 7. Analytics & Monitoring

### 7.1 User Analytics (Anonymous)
- Search queries (anonymized)
- Popular documents by language
- Video completion rates
- Upload success/failure rates
- Average session duration
- Device and browser distribution

### 7.2 System Monitoring
- API response times
- Error rates by endpoint
- Server CPU/memory usage
- Database query performance
- CDN cache hit rates
- Upload processing times

### 7.3 Alerting
- API response time > 5 seconds
- Error rate > 5%
- Server CPU > 80%
- Database connection pool exhausted
- CDN failures

## 8. Deployment Architecture

### 8.1 Environment Strategy
- **Development**: Local development with Docker Compose
- **Staging**: Replica of production for testing
- **Production**: Multi-region deployment for reliability

### 8.2 CI/CD Pipeline
```
Code Commit (Git)
↓
Automated Tests (Unit + Integration)
↓
Build Docker Images
↓
Push to Container Registry
↓
Deploy to Staging
↓
Automated E2E Tests
↓
Manual Approval
↓
Deploy to Production (Blue-Green)
↓
Health Checks
↓
Route Traffic to New Version
```

### 8.3 Scaling Strategy
- **Horizontal Scaling**: Add more application servers based on load
- **Database Replication**: Read replicas for search queries
- **CDN Scaling**: Automatic scaling via CDN provider
- **Auto-scaling Rules**: Scale up at 70% CPU, scale down at 30%

## 9. Admin Panel Design

### 9.1 Admin Features
- Add/edit/delete documents
- Upload videos for documents
- View analytics dashboard
- Manage translations
- Monitor system health
- View error logs

### 9.2 Admin Authentication
- Password-protected admin panel
- Two-factor authentication
- Role-based access control (Admin, Editor, Viewer)
- Audit logs for all admin actions

## 10. Testing Strategy

### 10.1 Unit Tests
- Test individual functions and components
- Mock external dependencies
- Target: 80% code coverage

### 10.2 Integration Tests
- Test API endpoints
- Test database operations
- Test file upload and processing

### 10.3 E2E Tests
- Test complete user flows (search, upload, video playback)
- Test on multiple browsers and devices
- Test with slow network conditions

### 10.4 Accessibility Tests
- Screen reader compatibility
- Keyboard navigation
- Color contrast validation
- WCAG 2.1 AA compliance

### 10.5 Performance Tests
- Load testing (1000 concurrent users)
- Stress testing (find breaking point)
- Video streaming performance
- Mobile device performance

## 11. Future Enhancements

### 11.1 Phase 2 Features
- **Smart Form Filling**: OCR + AI to pre-fill form fields
- **WhatsApp Bot**: Send document guidance via WhatsApp
- **Offline Mode**: PWA with offline video caching
- **More Languages**: Tamil, Kannada, Malayalam, Bengali

### 11.2 Phase 3 Features
- **User Accounts**: Save favorite documents, track applications
- **Document Checklist**: Personalized checklist for required documents
- **Community Forum**: Q&A section for users to help each other
- **AI Chatbot**: Instant answers to common questions

### 11.3 Advanced Features
- **DigiLocker Integration**: Fetch documents from DigiLocker
- **Government API Integration**: Check application status
- **SMS Notifications**: Send reminders and updates via SMS
- **Regional Customization**: State-specific documents and processes