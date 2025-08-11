# 02 — Luồng dữ liệu chi tiết TravelSense v2

## Luồng dữ liệu end-to-end

```mermaid
flowchart TD
    subgraph "Input Phase"
        A[User Input<br/>Origin, Destinations, Dates, Group Size, Style]
        B[Profile Analysis<br/>Past trips, Preferences, Constraints]
        C[Missing Info Detection<br/>Smart Questions, Optional Fields]
    end

    subgraph "Data Enrichment"
        D[External APIs<br/>Places, Weather, Accommodation]
        E[Vector Search<br/>Similar trips, POI recommendations]
        F[LLM Ideation<br/>Activity suggestions based on profile]
    end

    subgraph "Draft Generation"
        G[Constraint Modeling<br/>Opening hours, Travel time, Budget]
        H[Initial Route<br/>Nearest neighbor + local optimization]
        I[Draft Itinerary<br/>Day-by-day schedule with gaps]
    end

    subgraph "User Interaction"
        J[User Selections<br/>Accommodation, POIs, Preferences]
        K[Real-time Updates<br/>Cost calculation, Time adjustments]
        L[Conflict Detection<br/>Impossible schedules, Budget overrun]
    end

    subgraph "Optimization"
        M[Route Optimization<br/>TSP/VRP with time windows]
        N[LLM Validation<br/>Check constraints, Suggest alternatives]
        O[Final Itinerary<br/>Optimized schedule with sources]
    end

    subgraph "Export & Share"
        P[Version Management<br/>Snapshots, Diff tracking]
        Q[Multi-format Export<br/>PDF, ICS, JSON, Share links]
        R[Feedback Loop<br/>User ratings, Trip completion data]
    end

    A --> B --> C
    C --> D
    C --> E
    C --> F
    D --> G
    E --> G
    F --> G
    G --> H --> I
    I --> J --> K --> L
    L -->|Issues| C
    L -->|OK| M
    J --> M
    M --> N --> O
    O --> P --> Q
    Q --> R
    R -->|Learn| E

    %% Styling
    classDef input fill:#e3f2fd
    classDef enrich fill:#f1f8e9
    classDef draft fill:#fff3e0
    classDef interact fill:#fce4ec
    classDef optimize fill:#e8f5e8
    classDef export fill:#f3e5f5

    class A,B,C input
    class D,E,F enrich
    class G,H,I draft
    class J,K,L interact
    class M,N,O optimize
    class P,Q,R export
```

## Chi tiết từng giai đoạn

### 1. Input Phase - Thu thập dữ liệu thông minh

```mermaid
sequenceDiagram
    participant U as User
    participant UI as Frontend
    participant API as Backend
    participant ML as Missing Info Detector
    participant DB as Database

    U->>UI: Nhập thông tin cơ bản
    UI->>API: POST /trips/create
    API->>ML: Analyze missing fields
    ML->>DB: Check user history
    DB-->>ML: Past preferences
    ML->>ML: Generate smart questions
    ML-->>API: Priority question list
    API-->>UI: Basic trip + questions
    UI-->>U: Show form with smart defaults

    loop Optional Questions
        U->>UI: Answer/Skip question
        UI->>API: PATCH /trips/{id}/info
        API->>ML: Update completeness score
        ML-->>API: Next priority question
        API-->>UI: Updated questions
    end
```

**Thông tin cơ bản thu thập:**
- **Bắt buộc**: Origin, Destinations, Date range, Group size
- **Tùy chọn thông minh**: Budget range, Travel style, Special needs
- **Suy luận từ profile**: Past destinations, Preferred activities, Spending patterns

### 2. Data Enrichment - Làm giàu dữ liệu

```mermaid
graph LR
    subgraph "Vector Search"
        VS1[Embedding user preferences]
        VS2[Search similar trips]
        VS3[Find relevant POIs]
    end

    subgraph "External APIs"
        API1[Places API<br/>POI details, reviews]
        API2[Weather API<br/>Forecast, climate]
        API3[Maps API<br/>Distance matrix]
        API4[Accommodation<br/>Availability, pricing]
    end

    subgraph "LLM Processing"
        LLM1[Analyze user persona]
        LLM2[Generate activity ideas]
        LLM3[Suggest hidden gems]
    end

    VS1 --> VS2 --> VS3
    API1 --> LLM1
    API2 --> LLM1
    API3 --> LLM2
    API4 --> LLM3

    VS3 --> LLM1
    LLM1 --> LLM2 --> LLM3
```

**Dữ liệu được làm giàu:**
- **POI Data**: Tên, tọa độ, giờ mở cửa, giá vé, reviews
- **Weather Context**: Thời tiết dự báo, mùa vụ, điều kiện đặc biệt
- **Accommodation Options**: Khách sạn/homestay theo budget và vị trí
- **Activity Suggestions**: Dựa trên profile và POI similarity

### 3. Draft Generation - Tạo nháp lịch trình

```mermaid
flowchart TD
    A[Constraint Modeling] --> B[Time Windows]
    A --> C[Budget Limits]
    A --> D[Group Preferences]
    
    B --> E[Opening Hours Matrix]
    C --> F[Cost Estimates]
    D --> G[Activity Filters]
    
    E --> H[Initial Route Planning]
    F --> H
    G --> H
    
    H --> I[Nearest Neighbor TSP]
    I --> J[Local 2-opt Optimization]
    J --> K[Time Feasibility Check]
    
    K -->|Infeasible| L[Adjust Activities]
    K -->|Feasible| M[Draft Itinerary]
    
    L --> I
    M --> N[Gap Detection]
    N --> O[Highlight Missing Info]
```

**Constraint Types:**
- **Time Constraints**: POI opening hours, minimum visit duration
- **Budget Constraints**: Per-category spending limits
- **Physical Constraints**: Walking distance, transportation time
- **Group Constraints**: Kids-friendly, accessibility needs

### 4. User Interaction - Tương tác thời gian thực

```mermaid
stateDiagram-v2
    [*] --> DraftShown
    DraftShown --> SelectingPOI: Click on suggestion
    DraftShown --> SelectingAccommodation: Choose lodging
    DraftShown --> AdjustingTime: Drag timeline
    DraftShown --> ModifyingBudget: Update budget

    SelectingPOI --> RecalculatingRoute
    SelectingAccommodation --> RecalculatingRoute
    AdjustingTime --> RecalculatingCosts
    ModifyingBudget --> RecalculatingOptions

    RecalculatingRoute --> DraftUpdated
    RecalculatingCosts --> DraftUpdated
    RecalculatingOptions --> DraftUpdated

    DraftUpdated --> DraftShown: Show changes
    DraftUpdated --> ConflictDetected: Conflicts found

    ConflictDetected --> ShowAlternatives
    ShowAlternatives --> DraftShown: User resolves
```

**Real-time Calculations:**
- **Route Updates**: Khi thêm/bớt POI, tính lại optimal path
- **Cost Changes**: Update budget breakdown theo real-time
- **Time Conflicts**: Detect scheduling impossibilities
- **Alternative Suggestions**: Provide fallback options

### 5. Optimization - Tối ưu hóa cuối cùng

```mermaid
flowchart LR
    subgraph "Multi-objective Optimization"
        A[Minimize Travel Time]
        B[Maximize POI Quality Score]
        C[Stay Within Budget]
        D[Respect Time Constraints]
    end

    subgraph "Algorithm Stack"
        E[Genetic Algorithm<br/>Global search]
        F[Simulated Annealing<br/>Local optimization]
        G[Constraint Programming<br/>Feasibility check]
    end

    subgraph "Validation"
        H[LLM Constraint Check]
        I[Rule-based Validation]
        J[User Preference Scoring]
    end

    A --> E
    B --> E
    C --> F
    D --> G

    E --> H
    F --> I
    G --> J

    H --> K[Final Itinerary]
    I --> K
    J --> K
```

**Optimization Objectives:**
- **Primary**: Feasibility (tất cả constraints thỏa mãn)
- **Secondary**: User satisfaction score
- **Tertiary**: Cost efficiency
- **Quaternary**: Travel time minimization

### 6. Export & Feedback - Xuất và học hỏi

```mermaid
graph TB
    subgraph "Version Management"
        V1[Snapshot Current State]
        V2[Calculate Diff from Previous]
        V3[Store Change History]
    end

    subgraph "Export Formats"
        E1[PDF with Maps]
        E2[ICS Calendar Events]
        E3[JSON API Response]
        E4[Shareable Web Link]
    end

    subgraph "Feedback Collection"
        F1[Trip Completion Rate]
        F2[User Satisfaction Survey]
        F3[Actual vs Planned Comparison]
        F4[POI Rating Updates]
    end

    V1 --> V2 --> V3
    V3 --> E1
    V3 --> E2
    V3 --> E3
    V3 --> E4

    E1 --> F1
    E2 --> F2
    E3 --> F3
    E4 --> F4

    F1 --> ML[Machine Learning<br/>Improve Suggestions]
    F2 --> ML
    F3 --> ML
    F4 --> ML
```

## Data Quality & Reliability

### Source Citation System
```mermaid
erDiagram
    ACTIVITY ||--o{ SOURCE_CITATION : cites
    SOURCE_CITATION ||--|| EXTERNAL_SOURCE : references
    EXTERNAL_SOURCE ||--o{ RELIABILITY_SCORE : has

    ACTIVITY {
        uuid id
        string name
        timestamp start_time
        interval duration
    }

    SOURCE_CITATION {
        uuid id
        uuid activity_id
        uuid source_id
        timestamp checked_at
        float confidence_score
    }

    EXTERNAL_SOURCE {
        uuid id
        string provider
        string url
        timestamp last_updated
        string data_type
    }

    RELIABILITY_SCORE {
        uuid id
        uuid source_id
        float accuracy_score
        int total_verifications
        timestamp last_scored
    }
```

### Cache Strategy
- **L1 Cache**: Browser localStorage (user preferences, recent searches)
- **L2 Cache**: Redis (API responses, computed routes)
- **L3 Cache**: PostgreSQL materialized views (aggregated data)
- **TTL Strategy**: Dynamic TTL based on data volatility (weather: 1h, POI: 24h, maps: 1 week)
