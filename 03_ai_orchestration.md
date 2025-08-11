# 03 — AI Orchestration Strategy TravelSense v2

## Dual LLM Architecture

```mermaid
flowchart TD
    subgraph "Input Classification"
        A[Request Analysis]
        B{Task Type?}
        C[Context Requirements]
    end

    subgraph "LLM Cheap (GPT-3.5/Claude Haiku)"
        D[Creative Ideation]
        E[Casual Chat]
        F[General Q&A]
        G[Activity Brainstorming]
    end

    subgraph "LLM Structured (GPT-4/Claude Sonnet)"
        H[Schema Validation]
        I[Constraint Checking]
        J[JSON Output Generation]
        K[Complex Reasoning]
    end

    subgraph "Output Processing"
        L[Response Formatting]
        M[Source Citation]
        N[Confidence Scoring]
    end

    A --> B --> C
    
    B -->|Creative/Chat| D
    B -->|General| E
    B -->|Simple Q&A| F
    B -->|Brainstorming| G
    
    B -->|Needs Structure| H
    B -->|Complex Logic| I
    B -->|API Output| J
    B -->|Critical Decision| K

    D --> L
    E --> L
    F --> L
    G --> L
    H --> M
    I --> M
    J --> N
    K --> N

    L --> O[User Response]
    M --> O
    N --> O

    %% Cost optimization indicators
    classDef cheap fill:#e8f5e8
    classDef expensive fill:#ffebee
    classDef neutral fill:#f5f5f5

    class D,E,F,G cheap
    class H,I,J,K expensive
    class A,B,C,L,M,N,O neutral
```

## Task Routing Logic

## Detailed Task Routing Strategy 

### Cheap LLM Tasks (Cost-Optimized cho high-frequency, creative workflows)
```mermaid
mindmap
  root((Cheap LLM))
    Creative Tasks
      Activity brainstorming
      Local culture insights
      Food recommendations
      Hidden gems suggestions
    Conversational
      Welcome messages
      Clarifying questions
      General travel advice
      FAQ responses
    Simple Processing
      Text summarization
      Basic categorization
      Sentiment analysis
      Language detection
```

### Structured LLM Tasks (Quality-Critical cho business-logic và validation)
```mermaid
mindmap
  root((Structured LLM))
    Data Validation
      JSON schema compliance
      Constraint verification
      Data type checking
      Required field validation
    Complex Reasoning
      Multi-step planning
      Conflict resolution
      Trade-off analysis
      Optimization decisions
    Critical Output
      Final itinerary generation
      Budget calculations
      Safety recommendations
      Legal/visa requirements
```

## AI Orchestrator Implementation

```mermaid
sequenceDiagram
    participant U as User Request
    participant O as AI Orchestrator
    participant C as LLM Cheap
    participant S as LLM Structured
    participant V as Vector Store
    participant Cache as Redis Cache

    U->>O: Request + Context
    O->>O: Classify task type
    O->>Cache: Check cached response
    
    alt Cache Hit
        Cache-->>O: Cached result
        O-->>U: Return cached response
    else Cache Miss - Simple Task
        O->>C: Send to cheap LLM
        C-->>O: Creative response
        O->>Cache: Store result (TTL: 1h)
        O-->>U: Return response
    else Cache Miss - Complex Task
        O->>V: Retrieve relevant context
        V-->>O: Context embeddings
        O->>S: Send to structured LLM
        S-->>O: Validated JSON response
        O->>Cache: Store result (TTL: 24h)
        O-->>U: Return structured response
    end
```

## Prompt Engineering Strategy

### Template System
```mermaid
flowchart LR
    subgraph "Base Templates"
        A[System Prompt]
        B[Task-specific Context]
        C[User Input]
        D[Output Format]
    end

    subgraph "Dynamic Components"
        E[User Profile]
        F[Trip Context]
        G[External Data]
        H[Constraint Rules]
    end

    subgraph "Output Schemas"
        I[Activity Schema]
        J[Itinerary Schema]
        K[Budget Schema]
        L[Validation Schema]
    end

    A --> E
    B --> F
    C --> G
    D --> H

    E --> I
    F --> J
    G --> K
    H --> L
```

### Sample Prompt Templates

#### Cheap LLM - Activity Brainstorming
```
System: You are a creative travel assistant. Generate activity ideas based on user preferences.

Context:
- Destination: {destination}
- Travel Style: {style}
- Group: {group_type}
- Season: {season}

User Input: {user_request}

Output: Provide 5-7 creative activity suggestions with brief descriptions. Be enthusiastic but realistic.
```

#### Structured LLM - Itinerary Validation
```
System: You are a precise travel planning validator. Check itinerary feasibility and return structured JSON.

Schema: {itinerary_schema}

Constraints:
- Opening hours: {opening_hours_data}
- Travel times: {travel_matrix}
- Budget limits: {budget_constraints}
- Group needs: {group_requirements}

Input Itinerary: {draft_itinerary}

Task: Validate the itinerary and return JSON with:
1. is_valid: boolean
2. violations: array of constraint violations
3. suggestions: array of improvements
4. confidence_score: 0-1

Output must be valid JSON only.
```

## Context Management

### Vector Store Integration
```mermaid
graph TB
    subgraph "Embedding Sources"
        A[User Preferences]
        B[POI Descriptions]
        C[Activity Reviews]
        D[Historical Itineraries]
    end

    subgraph "Vector Operations"
        E[Text Embedding]
        F[Similarity Search]
        G[Context Retrieval]
    end

    subgraph "Context Augmentation"
        H[Relevant POIs]
        I[Similar Trips]
        J[User Patterns]
        K[Seasonal Info]
    end

    A --> E
    B --> E
    C --> E
    D --> E

    E --> F --> G

    G --> H
    G --> I
    G --> J
    G --> K

    H --> LLM[LLM Input]
    I --> LLM
    J --> LLM
    K --> LLM
```

### Advanced Context Window Management & Optimization

**TravelSense v2** sử dụng sophisticated context management để maximize hiệu quả của cả hai LLM tiers. **Context Window Allocation** được tối ưu với max 8K tokens cho cheap LLM (đủ cho creative tasks) và 32K tokens cho structured LLM (handle complex reasoning với nhiều constraints). **Dynamic Priority Ranking** áp dụng weighted scoring: User input (weight: 1.0), Current trip context (0.8), Relevant POI data (0.6), Historical patterns (0.4), và General knowledge (0.2). **Intelligent Truncation Strategy** giữ lại most recent user interactions, most relevant context from vector search, và core constraint data, đồng thời compress generic information thành summaries để tiết kiệm tokens mà vẫn preserve essential context.

## Quality Assurance

### Response Validation Pipeline
```mermaid
flowchart TD
    A[LLM Response] --> B{Response Type?}
    
    B -->|JSON| C[Schema Validation]
    B -->|Text| D[Content Quality Check]
    
    C --> E{Valid Schema?}
    E -->|No| F[Regenerate with Error Context]
    E -->|Yes| G[Constraint Validation]
    
    D --> H{Quality Score > Threshold?}
    H -->|No| F
    H -->|Yes| I[Sentiment Analysis]
    
    G --> J{Constraints Met?}
    J -->|No| F
    J -->|Yes| K[Source Citation Check]
    
    I --> L{Appropriate Tone?}
    L -->|No| F
    L -->|Yes| M[Pass to Output]
    
    K --> N{Sources Valid?}
    N -->|No| F
    N -->|Yes| M
    
    F --> O[Retry Count < 3?]
    O -->|Yes| P[Enhanced Prompt]
    O -->|No| Q[Fallback Response]
    
    P --> A
    M --> R[Final Response]
    Q --> R
```

## Cost Optimization

### Token Usage Monitoring
```mermaid
graph LR
    subgraph "Metrics Collection"
        A[Request Tokens]
        B[Response Tokens]
        C[Model Used]
        D[Response Time]
    end

    subgraph "Cost Analysis"
        E[Token Cost Calculation]
        F[Model Efficiency Score]
        G[Cache Hit Rate]
    end

    subgraph "Optimization Actions"
        H[Route to Cheaper Model]
        I[Increase Cache TTL]
        J[Prompt Compression]
        K[Context Pruning]
    end

    A --> E
    B --> E
    C --> F
    D --> F

    E --> H
    F --> I
    G --> J
    F --> K
```

### Cost Targets
- **Average Cost per Trip**: < $0.50
- **Cheap LLM Usage**: 70% of requests
- **Structured LLM Usage**: 30% of requests
- **Cache Hit Rate**: > 60%

## Error Handling & Fallbacks

### Graceful Degradation
```mermaid
stateDiagram-v2
    [*] --> Primary_LLM
    Primary_LLM --> Fallback_LLM: Timeout/Error
    Primary_LLM --> Cache_Response: Rate Limited
    Primary_LLM --> Success: Valid Response
    
    Fallback_LLM --> Cache_Response: Still Failing
    Fallback_LLM --> Success: Recovery
    
    Cache_Response --> Static_Response: Cache Miss
    Cache_Response --> Success: Cache Hit
    
    Static_Response --> Success: Default Message
    
    Success --> [*]
```

### Fallback Strategies
1. **Model Fallback**: GPT-4 → GPT-3.5 → Claude → Static response
2. **Cache Fallback**: Recent response → Similar request → Generic template
3. **Feature Fallback**: Full itinerary → Basic suggestions → Error message
4. **Timeout Handling**: Async processing → Progress updates → Partial results

## AI Ethics & Safety

### Content Filtering
- **Input Sanitization**: Prevent prompt injection
- **Output Monitoring**: Check for harmful content
- **Bias Detection**: Monitor for cultural/demographic bias
- **Privacy Protection**: No personal data in training

### Transparency
- **Source Attribution**: All suggestions cite sources
- **Confidence Scores**: Indicate AI certainty level
- **Human Override**: Allow manual corrections
- **Explainability**: Show reasoning for suggestions
