# EchoLearn Architecture Summary

## Overview

This document summarizes the simplified, hackathon-optimized architecture for EchoLearn.

## Architecture Philosophy

**MVP-First Approach**: Designed for rapid development and demonstration, using 6 core AWS services instead of a complex 12+ service architecture. The system can be extended post-hackathon without changing the product design.

## Core Technology Stack

### Frontend
- **Browser Extension**: React-based UI with Manifest V3
- **Components**: Floating notepad, screenshot tool, diagram canvas, chat interface
- **Storage**: Local storage for UI state and chat history

### Backend (AWS)
1. **API Gateway**: RESTful API endpoints
2. **Lambda**: Node.js 18.x serverless functions
3. **Amazon Bedrock**: AI embeddings and LLM
4. **DynamoDB**: Notes, metadata, and embedding vectors
5. **S3**: Screenshots and diagram images
6. **Cognito** (Optional): User authentication

## Key Architectural Decisions

### 1. Vector Storage in DynamoDB (Not OpenSearch)

**Decision**: Store embedding vectors directly in DynamoDB as number arrays.

**Rationale**:
- Simpler setup and management
- Lower cost for MVP scale
- Fast enough for hundreds/thousands of notes
- In-memory cosine similarity is acceptable for demo
- Can migrate to OpenSearch later without product changes

**Trade-off**: Slower than dedicated vector DB, but acceptable for hackathon.

### 2. AWS Bedrock (Not Local Models)

**Decision**: Use Amazon Bedrock for embeddings and LLM instead of browser-based models (Transformers.js).

**Rationale**:
- More powerful models than browser-compatible alternatives
- Better scalability and reliability
- Easier to add advanced features (summarization, chat)
- Native AWS integration with IAM and networking

**Trade-off**: Requires internet connection, but acceptable for MVP.

### 3. Serverless Architecture

**Decision**: Use Lambda + API Gateway instead of EC2 or containers.

**Rationale**:
- Zero server management
- Auto-scaling built-in
- Pay only for actual usage
- Perfect for hackathon demo with variable traffic

**Trade-off**: Cold start latency, but mitigated with provisioned concurrency if needed.

## Data Flow

### Note Creation
```
Extension → API Gateway → Lambda → Bedrock (embedding) → DynamoDB
```

### Context Matching
```
Extension → API Gateway → Lambda → Bedrock (embedding) → DynamoDB (fetch notes)
                                 → Compute similarity → Return top matches
```

### Screenshot Upload
```
Extension → API Gateway → Lambda → S3 (store image) → DynamoDB (metadata)
```

### Chat
```
Extension → API Gateway → Lambda → DynamoDB (fetch context) → Bedrock LLM → Response
```

## Database Schema

### DynamoDB - Notes Table
```
PK: USER#{userId}
SK: NOTE#{noteId}

Attributes:
- noteId, userId, content, title, summary
- embedding: number[] (stored as List)
- tags, topics, sourceUrl, sourceTitle
- mediaUrls, createdAt, updatedAt

GSI1: USER#{userId} + CREATED#{timestamp}
```

### DynamoDB - Media Table
```
PK: USER#{userId}
SK: MEDIA#{mediaId}

Attributes:
- mediaId, userId, noteId
- mediaType, s3Bucket, s3Key, s3Url
- createdAt
```

## API Endpoints

```
POST   /notes              Create note
GET    /notes              List notes
GET    /notes/{id}         Get note
PUT    /notes/{id}         Update note
DELETE /notes/{id}         Delete note

POST   /match              Find relevant notes
POST   /media              Upload media
GET    /media/{id}         Get media
POST   /chat               Chat message
```

## Cost Estimation (1000 users/month)

- Lambda: ~$20
- API Gateway: ~$3.50
- DynamoDB: ~$25
- S3: ~$5
- Bedrock: ~$50
- **Total: ~$103.50/month**

## Performance Targets

- Content analysis: < 2 seconds
- Note retrieval: < 1 second
- Page load overhead: < 100ms
- API response: < 3 seconds

## Future Enhancements (Post-Hackathon)

### Vector Database Migration
- Migrate from DynamoDB to OpenSearch or Pinecone
- Improves similarity search performance
- Architecture designed for easy swap

### Advanced Features
- Flowchart drawing canvas
- Spaced repetition algorithm
- Quiz generation
- Team collaboration
- Mobile app

## Why This Architecture Works for Hackathons

1. **Simple**: 6 services vs 12+ in production systems
2. **Fast**: Can be built in 2 days
3. **Scalable**: Auto-scales with AWS serverless
4. **Cost-effective**: Pay only for usage
5. **Extensible**: Can add features without redesign
6. **Demo-ready**: Works reliably for presentations

## Comparison: Original vs Simplified

| Aspect | Original (Local) | Simplified (AWS) |
|--------|-----------------|------------------|
| AI Processing | Transformers.js | Amazon Bedrock |
| Vector Storage | IndexedDB | DynamoDB |
| Scalability | Browser-limited | Cloud-scale |
| Offline Support | Yes | No (requires internet) |
| Setup Complexity | Medium | Low |
| Performance | Good | Better |
| Cost | Free | ~$100/month |
| Hackathon Fit | Good | Excellent |

## Development Timeline

### Day 1 (8 hours)
- [ ] Extension UI skeleton (2h)
- [ ] Lambda + API Gateway setup (2h)
- [ ] Bedrock integration (2h)
- [ ] DynamoDB schema (1h)
- [ ] Basic note CRUD (1h)

### Day 2 (8 hours)
- [ ] Context matching implementation (3h)
- [ ] Chat assistant (2h)
- [ ] Screenshot upload (2h)
- [ ] Testing and polish (1h)

## Conclusion

This architecture balances simplicity, functionality, and scalability. It's optimized for hackathon success while maintaining a clear path to production-ready features.

**Key Principle**: Start simple, prove the concept, then scale.
