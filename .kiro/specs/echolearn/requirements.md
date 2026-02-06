# Requirements Document

## Introduction

EchoLearn (RecallAI) is an AI-powered browser extension that transforms passive note-taking into an active learning companion. The system proactively surfaces relevant notes, diagrams, and content when users browse related material online, creating contextual connections that enhance learning and productivity.

## Glossary

- **Extension**: The browser extension component of EchoLearn (React-based UI)
- **Notepad**: The floating, resizable interface for creating and viewing notes
- **Backend**: AWS Lambda functions handling API requests and AI processing
- **Bedrock**: Amazon's AI service for embeddings generation and LLM responses
- **Note**: Any text content, screenshot, or diagram stored by the user
- **Embedding**: Vector representation of text content for semantic similarity
- **DynamoDB**: NoSQL database storing notes, metadata, and embedding vectors
- **S3**: Object storage for screenshots and diagram images
- **API_Gateway**: RESTful API layer between extension and backend
- **Matching_Service**: Lambda service that computes similarity between embeddings
- **Chat_Assistant**: AI-powered conversational interface using Bedrock LLM

## Requirements

### Requirement 1: Core Notepad Interface

**User Story:** As a learner, I want a floating notepad interface in my browser, so that I can quickly capture notes without leaving my current webpage.

#### Acceptance Criteria

1. WHEN a user activates the extension, THE Extension SHALL display a floating notepad interface
2. WHEN a user interacts with the notepad, THE Notepad SHALL remain accessible while browsing other content
3. WHEN a user resizes the notepad, THE Notepad SHALL maintain the new dimensions across browser sessions
4. THE Notepad SHALL support text input and paste operations for content capture
5. WHEN the notepad is minimized, THE Extension SHALL provide a visible indicator for quick access

### Requirement 2: Note Creation and Storage

**User Story:** As a user, I want to create and store notes with AI semantic understanding, so that my content can be intelligently retrieved later.

#### Acceptance Criteria

1. WHEN a user types or pastes content into the notepad, THE Backend SHALL send the content to Bedrock for embedding generation
2. WHEN embeddings are generated, THE Backend SHALL store the note content, embeddings, and metadata in DynamoDB
3. WHEN a note is saved, THE Backend SHALL assign a unique identifier and timestamp to the note
4. THE Backend SHALL persist all notes in DynamoDB for cross-device access
5. WHEN storing notes, THE Backend SHALL use Bedrock to generate summaries and extract key topics for enhanced retrieval

### Requirement 3: Context-Aware Note Retrieval

**User Story:** As a learner, I want relevant notes to automatically appear when I browse related content, so that I can make connections between current and previous learning.

#### Acceptance Criteria

1. WHEN a user navigates to a webpage, THE Extension SHALL extract page content and send it to the Backend via API_Gateway
2. WHEN page content is received, THE Backend SHALL generate embeddings using Bedrock and fetch user notes from DynamoDB
3. WHEN notes are fetched, THE Matching_Service SHALL compute cosine similarity between page embeddings and note embeddings
4. THE Matching_Service SHALL rank retrieved notes by similarity score and recency, returning top matches
5. WHEN no relevant notes exist (similarity < threshold), THE Extension SHALL indicate an empty state and suggest creating new notes

### Requirement 4: AI Chat Assistant

**User Story:** As a user, I want access to an AI chat assistant, so that I can quickly clarify doubts and get explanations about my notes or current content.

#### Acceptance Criteria

1. WHEN a user activates the chat feature, THE Extension SHALL provide a conversational interface
2. WHEN a user asks a question, THE Backend SHALL send the message to Bedrock LLM with relevant context
3. THE Backend SHALL fetch relevant notes from DynamoDB and include them in the LLM prompt for context-aware responses
4. WHEN discussing current webpage content, THE Backend SHALL incorporate page context into the LLM prompt
5. THE Extension SHALL maintain chat history in local storage within the current browser session

### Requirement 5: Cross-Domain Functionality

**User Story:** As a user, I want the extension to work across all websites, so that I can capture and retrieve notes regardless of the domain I'm browsing.

#### Acceptance Criteria

1. THE Extension SHALL function on all HTTP and HTTPS websites
2. WHEN switching between different domains, THE Extension SHALL maintain note retrieval functionality
3. THE Extension SHALL respect website security policies while maintaining core functionality
4. WHEN encountering restricted content, THE Extension SHALL gracefully handle access limitations
5. THE Context_Analyzer SHALL work across different content types including articles, documentation, and research papers

### Requirement 6: Performance and Responsiveness

**User Story:** As a user, I want the extension to be fast and responsive, so that it doesn't interfere with my browsing experience.

#### Acceptance Criteria

1. WHEN analyzing webpage content, THE Backend SHALL complete embedding generation and matching within 2 seconds
2. WHEN retrieving relevant notes, THE API_Gateway SHALL return results within 1 second
3. THE Extension SHALL not significantly impact page load times (less than 100ms overhead)
4. WHEN processing large amounts of text, THE Backend SHALL chunk content and handle it efficiently without blocking the UI
5. THE Extension SHALL limit memory usage and use efficient API communication to prevent browser performance degradation

### Requirement 7: Data Privacy and Security

**User Story:** As a user, I want my notes and browsing data to be secure and private, so that I can trust the extension with sensitive information.

#### Acceptance Criteria

1. THE Backend SHALL store all user data in AWS with encryption at rest (DynamoDB, S3)
2. WHEN processing content, THE Backend SHALL use HTTPS/TLS for all API communication and AWS IAM for service authentication
3. THE Extension SHALL provide clear privacy controls and data management options
4. WHEN a user requests data deletion, THE Backend SHALL remove all notes, embeddings, and media from DynamoDB and S3
5. THE Backend SHALL encrypt stored embeddings and note content using AWS managed encryption keys

### Requirement 8: Browser Extension Integration

**User Story:** As a user, I want seamless browser integration, so that the extension feels like a natural part of my browsing experience.

#### Acceptance Criteria

1. THE Extension SHALL integrate with browser extension APIs for proper installation and management
2. WHEN installed, THE Extension SHALL provide appropriate permissions requests with clear explanations
3. THE Extension SHALL support both Chrome and Firefox browser platforms
4. WHEN browser updates occur, THE Extension SHALL maintain compatibility with current browser versions
5. THE Extension SHALL provide standard extension controls through browser extension management interfaces

### Requirement 9: Note Organization and Management

**User Story:** As a user, I want to organize and manage my notes effectively, so that I can maintain a structured knowledge base.

#### Acceptance Criteria

1. WHEN viewing notes, THE Extension SHALL display creation timestamps and last modified dates
2. THE Extension SHALL provide search functionality that queries the Backend API for notes matching search terms
3. WHEN managing notes, THE Extension SHALL support note editing and deletion operations via API calls
4. THE Extension SHALL allow users to manually tag or categorize notes, stored in DynamoDB
5. WHEN exporting data, THE Backend SHALL provide notes in standard formats (JSON) via export API endpoint

### Requirement 11: Screenshot and Media Storage

**User Story:** As a user, I want to capture screenshots and diagrams, so that I can store visual content alongside my text notes.

#### Acceptance Criteria

1. WHEN a user captures a screenshot, THE Extension SHALL send the image data to the Backend via API_Gateway
2. WHEN image data is received, THE Backend SHALL upload the image to S3 and store metadata in DynamoDB
3. THE Backend SHALL generate a unique media ID and return the S3 URL to the Extension
4. WHEN displaying notes with media, THE Extension SHALL fetch and display images from S3 URLs
5. WHEN a note with media is deleted, THE Backend SHALL remove both the DynamoDB record and the S3 object

### Requirement 10: AI Processing and Embeddings

**User Story:** As a developer, I want robust AI processing capabilities, so that the semantic understanding and retrieval system works accurately across diverse content types.

#### Acceptance Criteria

1. THE Backend SHALL use Amazon Bedrock to generate consistent embeddings for similar content across different phrasings
2. WHEN processing technical content, THE Bedrock embedding model SHALL maintain accuracy for domain-specific terminology
3. THE Bedrock embedding model SHALL handle multiple languages for international users
4. WHEN content contains code snippets, THE Backend SHALL preserve technical context in embeddings
5. THE Matching_Service SHALL use cosine similarity for semantic matching with minimum 0.7 accuracy threshold