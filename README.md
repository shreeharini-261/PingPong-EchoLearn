# EchoLearn - Your Knowledge, Right When You Need It

> An AI-powered browser extension that transforms passive note-taking into an active learning companion through semantic understanding and contextual retrieval.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![AWS](https://img.shields.io/badge/AWS-Powered-orange.svg)](https://aws.amazon.com/)
[![Bedrock](https://img.shields.io/badge/Amazon-Bedrock-blue.svg)](https://aws.amazon.com/bedrock/)

## 🎯 Problem Statement

Students and developers take large volumes of notes, screenshots, and diagrams while learning. However:

- Notes are rarely revised
- Important connections between topics are forgotten
- Learners must manually search old notes
- Context switching reduces productivity
- Existing note tools are passive storage systems, not active learning companions

## 💡 Solution

EchoLearn is an AI-powered browser extension notepad that:

- Lets users write notes, paste text, add screenshots, and create diagrams
- **Understands the meaning** of stored content using AI embeddings
- **Automatically surfaces relevant notes** when users browse related content
- Acts as a floating right-side learning companion
- Provides AI chat for quick doubt clarification

The system transforms notes from static storage into **context-aware, proactive revision tools**.

## ✨ Key Features

### MVP (Hackathon Version)
- ✅ **Floating Notepad**: Resizable, always-accessible note interface
- ✅ **Smart Note Storage**: AI-powered semantic understanding of content
- ✅ **Context-Aware Retrieval**: Automatically shows relevant notes based on current webpage
- ✅ **AI Chat Assistant**: Quick doubt clarification and explanations
- ✅ **Screenshot Support**: Capture and store visual content
- ✅ **Cross-Domain**: Works on all websites

### Future Features (Phase 2)
- 📊 Flowchart drawing canvas
- 🎨 Color-coded notes and tags
- 🧠 Quiz generation from studied topics
- ⏱️ Pomodoro timer integration
- 🔄 Spaced repetition reminders
- 👥 Team collaboration

## 🏗️ Architecture

### High-Level Overview

```
Browser Extension → API Gateway → Lambda → Amazon Bedrock
                                    ↓
                              DynamoDB + S3
```

### Technology Stack

**Frontend:**
- Browser Extension (React)
- Manifest V3
- Chrome & Firefox compatible

**Backend:**
- AWS Lambda (Node.js 18.x)
- API Gateway (REST API)
- Amazon Bedrock (Embeddings + LLM)
- DynamoDB (Notes + Embeddings storage)
- S3 (Screenshots + Diagrams)
- Cognito (Optional authentication)

**AI Models:**
- Amazon Titan Embeddings (semantic understanding)
- Anthropic Claude (chat assistant)

### Why This Stack?

**MVP-First Approach**: This architecture is optimized for hackathon demonstration with 6 core AWS services instead of 12+. It's simple, fast to develop, and can be extended post-hackathon without changing the product design.

**Vector Storage**: Embeddings are stored directly in DynamoDB (as arrays) for MVP scale. This is acceptable for hundreds/thousands of notes and avoids the complexity of dedicated vector databases like OpenSearch. Similarity matching is done in-memory using cosine similarity.

**Scalability**: The architecture is designed so we can later plug in a dedicated vector database (OpenSearch, Pinecone) without changing the product design.

## 🚀 Getting Started

### Prerequisites

- Node.js 18.x or higher
- AWS Account with Bedrock access
- AWS CLI configured
- Chrome or Firefox browser

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/echolearn.git
cd echolearn
```

2. **Install dependencies**
```bash
# Backend
cd backend
npm install

# Extension
cd ../extension
npm install
```

3. **Configure AWS**
```bash
# Set up AWS credentials
aws configure

# Deploy infrastructure (using SAM or Terraform)
cd backend
sam deploy --guided
```

4. **Build extension**
```bash
cd extension
npm run build
```

5. **Load extension in browser**
- Open Chrome/Firefox
- Navigate to `chrome://extensions` or `about:debugging`
- Enable "Developer mode"
- Click "Load unpacked"
- Select the `extension/dist` folder

### Environment Variables

Create a `.env` file in the backend directory:

```env
AWS_REGION=us-east-1
DYNAMODB_TABLE_NAME=echolearn-notes
S3_BUCKET_NAME=echolearn-media
BEDROCK_EMBEDDING_MODEL=amazon.titan-embed-text-v1
BEDROCK_LLM_MODEL=anthropic.claude-v2
```

## 📖 Usage

### Creating Notes

1. Click the EchoLearn icon in your browser
2. Type or paste content into the floating notepad
3. AI automatically processes and stores the semantic meaning
4. Notes are saved with timestamps and metadata

### Automatic Recall

1. Browse to any webpage
2. EchoLearn analyzes the page content
3. Relevant notes automatically appear in the notepad
4. Notes are ranked by semantic similarity

### AI Chat

1. Click the chat icon in the notepad
2. Ask questions about your notes or current content
3. AI provides contextually relevant responses
4. Chat has access to your stored notes for better answers

### Screenshots

1. Click the screenshot icon
2. Select area to capture
3. Screenshot is stored in S3 and linked to notes
4. Appears automatically when browsing related content

## 🧪 Example Use Cases

### Student Studying ML
- Reads CNN article → EchoLearn shows previous CNN architecture notes
- Compares current content with past learning
- Asks AI to explain differences

### Developer Coding
- Browsing authentication tutorial → EchoLearn shows JWT flowchart created earlier
- Quick reference without leaving the page
- AI chat for quick clarifications

### Product Research
- Comparing phones → EchoLearn shows saved spec comparison notes
- Makes informed decisions faster
- All research in one place

## 🏆 Innovation / Wow Factor

- **Proactive Learning**: Notes come to you, not the other way around
- **Real-Time Semantic Recall**: AI understands meaning, not just keywords
- **Floating AI Companion**: Always accessible, never intrusive
- **Cross-Domain Usage**: Learning, coding, research, shopping - all in one tool

## 📊 API Documentation

### Endpoints

**Note Operations**
```
POST   /notes              Create new note
GET    /notes              List all notes
GET    /notes/{id}         Get specific note
PUT    /notes/{id}         Update note
DELETE /notes/{id}         Delete note
```

**Context Matching**
```
POST   /match              Find relevant notes for current page
```

**Media Operations**
```
POST   /media              Upload screenshot/diagram
GET    /media/{id}         Retrieve media
```

**Chat Operations**
```
POST   /chat               Send chat message
```

### Example Request

**Create Note**
```bash
curl -X POST https://api.echolearn.com/notes \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Convolutional Neural Networks use filters to detect features in images",
    "title": "CNN Basics",
    "tags": ["machine-learning", "deep-learning"]
  }'
```

**Response**
```json
{
  "noteId": "note_123abc",
  "embedding": [0.123, 0.456, ...],
  "summary": "Overview of CNN architecture and feature detection",
  "topics": ["CNN", "neural networks", "computer vision"],
  "createdAt": "2026-02-06T10:30:00Z"
}
```

## 🧪 Testing

```bash
# Run unit tests
npm test

# Run integration tests
npm run test:integration

# Run property-based tests
npm run test:properties

# Run all tests
npm run test:all
```

## 📈 Performance

- **Content Analysis**: < 2 seconds
- **Note Retrieval**: < 1 second
- **Page Load Overhead**: < 100ms
- **API Response Time**: < 3 seconds

## 💰 Cost Estimation

**Monthly costs for 1000 active users:**
- Lambda: ~$20
- API Gateway: ~$3.50
- DynamoDB: ~$25
- S3: ~$5
- Bedrock: ~$50
- **Total: ~$103.50/month**

## 🔒 Security & Privacy

- All data encrypted at rest and in transit
- AWS IAM for service authentication
- Optional user authentication via Cognito
- No sensitive data logged
- GDPR compliant with data deletion on request

## 🗺️ Roadmap

### Phase 1: MVP (Hackathon - 2 days) ✅
- Browser extension UI
- Note creation and storage
- Context matching
- AI chat assistant

### Phase 2: Core Features (Week 1-2)
- Screenshot capture
- Diagram canvas
- Search functionality
- Tag management

### Phase 3: Advanced Features (Week 3-4)
- User authentication
- Multi-device sync
- Quiz generation
- Spaced repetition

### Phase 4: Production Ready (Week 5-6)
- Comprehensive testing
- Security audit
- Chrome Web Store submission
- User onboarding

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Team

- **Your Name** - [GitHub](https://github.com/yourusername)

## 🙏 Acknowledgments

- Amazon Bedrock for AI capabilities
- AWS for cloud infrastructure
- The open-source community

## 📞 Contact

- Email: your.email@example.com
- Twitter: [@yourhandle](https://twitter.com/yourhandle)
- Project Link: [https://github.com/yourusername/echolearn](https://github.com/yourusername/echolearn)

---

**Built with ❤️ for learners everywhere**

*EchoLearn - Because your past knowledge should inform your present learning*
