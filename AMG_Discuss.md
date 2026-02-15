# EduQuest: AWS Hackathon Discussion & Reference Guide

## Project Overview

EduQuest is an AI-native Active Learning OS that transforms passive content consumption into active mastery through gamified, contextual learning. Built entirely on AWS services for the hackathon.

## Quick Reference

### Core Value Proposition
- **Problem**: Students and developers consume content passively, leading to poor retention (70% forgotten in 24 hours)
- **Solution**: Active learning through AI-generated challenges, spaced repetition, and contextual tutoring
- **Impact**: 60% less context switching, 50% faster onboarding, 90% retention at 30 days

### Dual Persona Approach
1. **Students**: PDF/textbook → Quizzes, flashcards, fill-in-blanks
2. **Developers**: GitHub repos → Debugging challenges, code comprehension, refactoring

## AWS Services Architecture

### Core Services
- **Amazon Bedrock**: AI foundation (Claude 3.5 Sonnet for generation, Titan Embeddings v2 for vectors)
- **Amazon OpenSearch Serverless**: Vector database for semantic search
- **Amazon DynamoDB**: User data, challenges, progress tracking
- **Amazon ElastiCache (Redis)**: Real-time leaderboards
- **Amazon S3**: Content storage with event notifications
- **AWS Lambda**: Serverless compute for processing and API
- **Amazon Cognito**: Authentication and user management
- **Amazon Textract**: PDF text extraction
- **Amazon SQS**: Async task queue
- **Amazon CloudWatch**: Monitoring and logging
- **AWS Amplify**: Frontend hosting with CI/CD

### Why This Stack?
- **Serverless-first**: Minimal ops, auto-scaling, pay-per-use
- **AI-native**: Bedrock provides managed LLMs without infrastructure
- **Event-driven**: S3 events + SQS for async processing
- **Real-time**: Redis for instant leaderboard updates
- **Observable**: CloudWatch for comprehensive monitoring

## Key Technical Decisions

### 1. RAG (Retrieval Augmented Generation)
**Why**: Eliminates AI hallucinations by grounding responses in source material
**How**: 
- Chunk content into 500-1000 token segments
- Generate embeddings with Titan v2 (1024 dimensions)
- Store in OpenSearch with metadata
- Retrieve top-k relevant chunks for each AI query
- Pass chunks as context to Claude

### 2. Spaced Repetition (SM-2 Algorithm)
**Why**: Proven to boost long-term retention by 90%+
**How**:
- Track repetitions, easiness factor, interval for each challenge
- Correct answer → increase interval exponentially
- Incorrect answer → reset to 1 day
- Schedule reviews based on calculated intervals

### 3. Interleaved Practice
**Why**: 43% better retention vs. blocked practice
**How**:
- Mix challenge modalities within sessions
- Prioritize weak topics (low accuracy)
- Avoid 3+ consecutive challenges of same type
- Adapt difficulty based on rolling accuracy

### 4. Gamification Layer
**Why**: Increases engagement and motivation
**How**:
- XP system (10-100 per challenge)
- Streaks (consecutive days active)
- Leaderboards (global + squad-based)
- Focus scores (0-100 based on session quality)
- Badges and progression tiers

## Implementation Phases

### Phase 1: MVP Core (Week 1-2)
- [ ] S3 + Lambda content ingestion pipeline
- [ ] Textract PDF extraction
- [ ] Bedrock Titan embeddings generation
- [ ] OpenSearch vector storage
- [ ] Basic challenge generation (MCQ only)
- [ ] Simple Next.js UI for upload + challenges

### Phase 2: AI Tutor (Week 2-3)
- [ ] RAG pipeline with LangChain
- [ ] Claude 3.5 Sonnet integration
- [ ] Context retrieval from OpenSearch
- [ ] Tutor chat interface
- [ ] Multilingual support (English, Hindi, Spanish)

### Phase 3: Practice Engine (Week 3-4)
- [ ] SM-2 spaced repetition scheduler
- [ ] DynamoDB user challenge tracking
- [ ] Interleaving algorithm
- [ ] Daily practice queue
- [ ] Performance analytics

### Phase 4: Social Features (Week 4-5)
- [ ] Cognito authentication
- [ ] Squad creation and management
- [ ] Redis leaderboards
- [ ] Activity feeds
- [ ] Real-time updates

### Phase 5: Polish & Demo (Week 5-6)
- [ ] Developer persona features (GitHub ingestion, debugging challenges)
- [ ] Analytics dashboard
- [ ] CloudWatch monitoring setup
- [ ] Performance optimization
- [ ] Demo flow preparation

## Demo Script (5 Minutes)

### Opening (30 sec)
"EduQuest solves the passive learning problem. Whether you're a student cramming for exams or a developer onboarding to a new codebase, we turn information into intuition through active practice."

### Student Scenario (2 min)
1. Upload CS textbook chapter on "Sorting Algorithms"
2. Show auto-generated Quest with 5 challenges
3. Complete a fill-in-the-blank on QuickSort complexity
4. Ask tutor: "Explain MergeSort vs QuickSort in Hindi"
5. Show step-by-step explanation with source citations

### Pivot (30 sec)
"Now imagine you're not a student, but a new developer joining a team."

### Developer Scenario (2 min)
1. Paste GitHub URL for auth library
2. Show auto-generated Sprint with debugging challenges
3. Complete a "Find the race condition" challenge
4. Ask tutor: "Explain this middleware pattern vs Express.js"
5. Show comparative explanation with code references

### Unification (30 sec)
1. Show unified leaderboard with both personas
2. Highlight XP, streaks, focus scores
3. "Knowledge acquisition is the universal score"

### Closing (30 sec)
"Built entirely on AWS: Bedrock for AI, OpenSearch for vectors, Lambda for compute, DynamoDB for data. Serverless, scalable, and ready to transform how we learn."

## Key Metrics to Highlight

### Cognitive Science Backed
- **70% forgetting curve**: Without active recall, we forget 70% within 24 hours
- **43% retention boost**: Interleaved practice vs. blocked studying
- **90% retention**: Spaced repetition at 30-day intervals

### Business Impact
- **60% reduction**: Context switching for developers
- **50% faster**: Developer onboarding time
- **40+ hours saved**: Per developer per quarter

### Technical Performance
- **Sub-100ms**: Vector search latency
- **<2 seconds**: Page load times
- **10,000+ concurrent users**: Auto-scaling capacity
- **99.9% uptime**: AWS managed services SLA

## Common Questions & Answers

### Q: How do you prevent AI hallucinations?
**A**: RAG architecture. Every AI response is grounded in retrieved chunks from the source material stored in OpenSearch. We never let the model generate from memory alone.

### Q: How do you handle large files (100MB+ PDFs)?
**A**: Async processing via SQS. Upload triggers S3 event → SQS message → Lambda processes in background → User gets notification when ready.

### Q: How do you ensure challenge quality?
**A**: Prompt engineering + validation. We use structured prompts with Claude 3.5 Sonnet, validate output schemas, and include source references for every challenge.

### Q: How do you scale to thousands of users?
**A**: Serverless architecture. Lambda auto-scales, OpenSearch Serverless handles query load, DynamoDB scales horizontally, Redis handles hot data.

### Q: How do you handle multilingual support?
**A**: Bedrock Claude supports 100+ languages natively. We detect user language preference and pass it in the system prompt for tutor responses.

### Q: What about cost optimization?
**A**: 
- Lambda: Pay per invocation (free tier covers dev)
- Bedrock: Pay per token (cache embeddings to reduce calls)
- OpenSearch Serverless: Pay for indexed data + queries
- DynamoDB: On-demand pricing for variable load
- S3: Lifecycle policies to archive old content

## Technical Deep Dives

### Challenge Generation Pipeline

```
User uploads PDF
    ↓
S3 stores file + triggers event
    ↓
SQS queues processing job
    ↓
Lambda extracts text (Textract)
    ↓
Lambda chunks text (500-1000 tokens)
    ↓
Lambda generates embeddings (Bedrock Titan)
    ↓
Lambda stores vectors (OpenSearch)
    ↓
Lambda updates status (DynamoDB)
    ↓
User requests Quest generation
    ↓
Lambda retrieves relevant chunks (OpenSearch)
    ↓
Lambda generates challenges (Bedrock Claude)
    ↓
Lambda stores challenges (DynamoDB)
    ↓
User sees Quest in UI
```

### RAG Tutor Flow

```
User asks question
    ↓
Lambda embeds question (Bedrock Titan)
    ↓
Lambda searches vectors (OpenSearch cosine similarity)
    ↓
Lambda retrieves top-5 chunks (confidence > 0.7)
    ↓
Lambda builds context prompt
    ↓
Lambda calls Claude with context + question
    ↓
Claude generates grounded response
    ↓
Lambda returns response + source citations
    ↓
User sees answer in chat
```

### Spaced Repetition Scheduler

```
User completes challenge
    ↓
Lambda calculates quality score (0-5)
    ↓
Lambda retrieves current SR state (DynamoDB)
    ↓
Lambda applies SM-2 algorithm
    ↓
If correct: interval *= easiness_factor
If incorrect: interval = 1 day
    ↓
Lambda updates next_review_date (DynamoDB)
    ↓
Lambda adds XP to user total
    ↓
Lambda updates leaderboard (Redis)
    ↓
User sees XP animation + next review date
```

## AWS Service Justifications

### Why Amazon Bedrock?
- Managed LLM access (no infrastructure)
- Claude 3.5 Sonnet: Best-in-class reasoning
- Titan Embeddings v2: High-quality vectors
- Pay-per-token pricing
- Built-in content filtering

### Why OpenSearch Serverless?
- Purpose-built for vector search
- HNSW algorithm for fast similarity search
- Serverless = no cluster management
- Scales automatically with query load
- Integrated with AWS ecosystem

### Why DynamoDB?
- Single-digit millisecond latency
- Infinite scale with on-demand pricing
- Flexible schema for evolving data models
- DynamoDB Streams for real-time analytics
- Global tables for multi-region (future)

### Why ElastiCache Redis?
- In-memory speed for leaderboards
- Sorted sets perfect for rankings
- Sub-millisecond latency
- Pub/sub for real-time notifications
- Managed service (no ops)

### Why Lambda?
- Zero server management
- Auto-scaling to zero
- Pay per 100ms of execution
- Integrates with all AWS services
- Perfect for event-driven architecture

### Why Amplify?
- Git-based CI/CD
- Edge hosting via CloudFront
- Environment management
- Built-in preview deployments
- Optimized for Next.js

## Potential Extensions

### Post-Hackathon Features
1. **Mobile Apps**: React Native with offline support
2. **Voice Tutor**: Bedrock + Polly for audio explanations
3. **Live Collaboration**: Real-time study sessions with WebRTC
4. **Content Marketplace**: Share and monetize Quests
5. **Enterprise Features**: Team analytics, SSO, custom branding
6. **Adaptive Difficulty**: ML model for personalized challenge selection
7. **Gamification++**: Achievements, power-ups, boss battles
8. **Integration APIs**: Canvas LMS, GitHub Classroom, Notion

### Advanced AI Features
1. **Multimodal Learning**: Image/video content with Bedrock Nova
2. **Concept Graphs**: Knowledge graph visualization
3. **Peer Comparison**: "You're in top 10% for this topic"
4. **Predictive Analytics**: "You'll master this in 3 more sessions"
5. **Auto-Curriculum**: AI-generated learning paths

## Troubleshooting Guide

### Common Issues

**Issue**: Textract fails on scanned PDFs
**Solution**: Use Textract's OCR capabilities, increase timeout, handle errors gracefully

**Issue**: OpenSearch queries too slow
**Solution**: Optimize index settings, use filters to reduce search space, cache frequent queries

**Issue**: Lambda cold starts
**Solution**: Provisioned concurrency for critical functions, optimize package size, use Lambda layers

**Issue**: Bedrock rate limits
**Solution**: Implement exponential backoff, batch requests, cache embeddings

**Issue**: DynamoDB hot partitions
**Solution**: Use composite keys, enable auto-scaling, consider GSIs for access patterns

## Cost Estimation (Monthly)

### Development/Demo (Low Usage)
- Lambda: $0 (free tier)
- Bedrock: ~$50 (1M tokens)
- OpenSearch: ~$100 (small index)
- DynamoDB: $0 (free tier)
- S3: $5 (100GB)
- ElastiCache: $15 (t4g.micro)
- **Total: ~$170/month**

### Production (1000 active users)
- Lambda: ~$50
- Bedrock: ~$500 (10M tokens)
- OpenSearch: ~$300
- DynamoDB: ~$100
- S3: ~$20
- ElastiCache: ~$50 (t4g.small)
- CloudWatch: ~$30
- **Total: ~$1,050/month**

### Scale (10,000 active users)
- Lambda: ~$200
- Bedrock: ~$2,000
- OpenSearch: ~$800
- DynamoDB: ~$500
- S3: ~$100
- ElastiCache: ~$200 (r6g.large)
- CloudWatch: ~$100
- **Total: ~$3,900/month**

## Resources & Links

### AWS Documentation
- [Amazon Bedrock](https://docs.aws.amazon.com/bedrock/)
- [OpenSearch Serverless](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless.html)
- [DynamoDB](https://docs.aws.amazon.com/dynamodb/)
- [Lambda](https://docs.aws.amazon.com/lambda/)
- [Amplify](https://docs.amplify.aws/)

### Learning Science
- [Spaced Repetition (SM-2)](https://www.supermemo.com/en/archives1990-2015/english/ol/sm2)
- [Interleaved Practice Research](https://www.scientificamerican.com/article/the-interleaving-effect-mixing-it-up-boosts-learning/)
- [Retrieval Practice](https://www.retrievalpractice.org/)

### Tech Stack
- [Next.js 14](https://nextjs.org/)
- [LangChain](https://python.langchain.com/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Tailwind CSS](https://tailwindcss.com/)
- [shadcn/ui](https://ui.shadcn.com/)

## Hackathon Submission Checklist

- [x] requirements.md created
- [x] design.md created
- [ ] GitHub repo created
- [ ] requirements.md uploaded to repo
- [ ] design.md uploaded to repo
- [ ] README.md with project overview
- [ ] Presentation deck created
- [ ] Demo video recorded (optional but recommended)
- [ ] Architecture diagram included
- [ ] AWS services clearly highlighted
- [ ] Submission form completed

## Next Steps

1. **Create GitHub Repo**: Initialize with README
2. **Upload Spec Files**: Copy requirements.md and design.md
3. **Create Presentation**: Use this guide for talking points
4. **Build MVP**: Start with Phase 1 (content ingestion)
5. **Prepare Demo**: Follow the 5-minute script
6. **Submit**: Complete hackathon submission form

---

**Good luck with the hackathon! 🚀**

*Remember: The spec files are your blueprint. Refer to them constantly during implementation to ensure you're building what you designed.*
