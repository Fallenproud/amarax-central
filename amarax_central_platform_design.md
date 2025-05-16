# Amarax Central Platform - Detailed Design Document

## 0. Document Version

-   **Version:** 1.1 (Incorporating Amarax Central Branding and New Features)
-   **Date:** May 14, 2025
-   **Status:** Draft

## 1. Introduction

This document provides the detailed design for the **Amarax Central Platform**, a sophisticated web application developed by **Amarax Productions™️©️**. It builds upon initial concepts and the selected architectural approach (Flask backend, React/Next.js frontend). The goal is to create a comprehensive platform that includes AI chat, RAG-powered specialized agent creation, an **Agent Market**, an **Agent Flow Builder**, a downloadable desktop client, subscription management, and web roaming capabilities. The user interface will be heavily inspired by the **Thrae IDE layout** (referencing user-provided image `Skjermbilde 2025-02-20 104826.png`) and will adopt color schemes derived from the latest set of visual references provided by the user.

## 2. Overall System Architecture (Recap and Elaboration)

The platform will consist of several key interacting components:

1.  **Web Frontend (React/Next.js):** The primary user interface, providing the Thrae IDE-style experience. It will handle user interactions, display information, and communicate with the backend via APIs. Visual design will adhere to Amarax Central branding and color schemes from provided images.
2.  **Web Backend (Flask):** The core server-side application. It will handle:
    *   User authentication and authorization.
    *   Business logic for all features (chat, RAG agents, subscriptions, Agent Market, Agent Flow Builder).
    *   AI processing (interfacing with LLMs, RAG components, flow execution).
    *   Database interactions.
    *   Serving APIs for the frontend and the downloadable client.
    *   Managing web roaming tasks.
3.  **Database (MySQL recommended by Flask template, or PostgreSQL):** Stores user data, subscription information, RAG configurations, agent definitions, **published agent market data, agent flow definitions,** and potentially cached data.
4.  **Downloadable Desktop Client (Python with CustomTkinter or a web-wrapper like Electron):** A client application branded for Amarax Central, connecting to the Flask backend.
5.  **RAG Sub-system (Integrated into Flask Backend):** As previously defined, supporting specialized agent creation.
6.  **LLM Interface (Integrated into Flask Backend):** As previously defined.
7.  **Subscription Management Module (Integrated into Flask Backend):** As previously defined.
8.  **Web Roaming Module (Integrated into Flask Backend):** As previously defined.
9.  **Agent Market Module (New - Integrated into Flask Backend):** Manages the publishing, discovery, and acquisition of user-created agents.
10. **Agent Flow Builder Module (New - Integrated into Flask Backend):** Provides tools and services for users to visually design and implement complex agent behaviors and workflows.

### 2.1. Architectural Diagram (Conceptual Refinement for Amarax Central)

```mermaid
graph TD
    User --> WebFrontend["Amarax Central Web Frontend (React/Next.js, Thrae IDE-Style UI)"];
    User --> DownloadableClient["Amarax Central Downloadable Client"];

    WebFrontend -- HTTPS (API Calls) --> FlaskBackend;
    DownloadableClient -- HTTPS (API Calls) --> FlaskBackend;

    FlaskBackend["Flask Backend (Python) - Amarax Central Core"] -- Manages --> UserAuthModule[User Authentication Module];
    FlaskBackend -- Manages --> APIServer[API Server (RESTful Endpoints)];
    FlaskBackend -- Manages --> ChatLogicModule[Chat Logic Module];
    FlaskBackend -- Manages --> RAGSubsystem[RAG Sub-system];
    FlaskBackend -- Manages --> LLMInterfaceModule[LLM Interface Module];
    FlaskBackend -- Manages --> SubscriptionModule[Subscription Management Module];
    FlaskBackend -- Manages --> WebRoamingModule[Web Roaming Module];
    FlaskBackend -- Manages --> AgentMarketModule[Agent Market Module];
    FlaskBackend -- Manages --> AgentFlowBuilderModule[Agent Flow Builder Module];
    FlaskBackend -- CRUD Operations --> Database[(Database: MySQL/PostgreSQL)];

    RAGSubsystem -- Manages --> DataIngestionPipeline[Data Ingestion Pipeline];
    RAGSubsystem -- Manages --> VectorStore[Vector Store (ChromaDB/FAISS)];
    RAGSubsystem -- Uses --> LLMInterfaceModule;

    AgentMarketModule -- Interacts --> Database;
    AgentFlowBuilderModule -- Interacts --> Database;
    AgentFlowBuilderModule -- Uses --> LLMInterfaceModule;
    AgentFlowBuilderModule -- May Use --> RAGSubsystem;
    AgentFlowBuilderModule -- May Use --> WebRoamingModule;

    LLMInterfaceModule -- Interacts --> LLMs[LLMs (Local/API-based)];

    WebRoamingModule -- Controls --> HeadlessBrowsers[Headless Browsers (Playwright/Selenium)];
    HeadlessBrowsers -- Interacts --> Internet[Internet];

    SubscriptionModule -- Integrates (Placeholder) --> PaymentGateway[Payment Gateway (e.g., Stripe)];

    style User fill:#f9f,stroke:#333,stroke-width:2px;
    style WebFrontend fill:#ccf,stroke:#333,stroke-width:2px;
    style DownloadableClient fill:#ccf,stroke:#333,stroke-width:2px;
    style FlaskBackend fill:#fcc,stroke:#333,stroke-width:2px;
    style Database fill:#dbf,stroke:#333,stroke-width:2px;
    style Internet fill:#9cf,stroke:#333,stroke-width:2px;
```

(The rest of the document will be updated to reflect Amarax Central branding and new features in subsequent steps.)




## 3. Database Schema (Updated for Amarax Central)

We will use a relational database (MySQL or PostgreSQL). The following tables are envisioned (existing tables are retained and new ones for Agent Market and Flow Builder are added):

*   **Users:** (As previously defined)
    *   `user_id` (PK, INT, AUTO_INCREMENT)
    *   `username` (VARCHAR, UNIQUE, NOT NULL)
    *   `email` (VARCHAR, UNIQUE, NOT NULL)
    *   `password_hash` (VARCHAR, NOT NULL)
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `subscription_id` (FK, INT, NULLABLE) - Links to Subscriptions table
    *   `api_key` (VARCHAR, UNIQUE, NULLABLE) - For programmatic access if needed

*   **Subscriptions:** (As previously defined)
    *   `subscription_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - Links to Users table
    *   `tier_id` (FK, INT, NOT NULL) - Links to SubscriptionTiers table
    *   `start_date` (DATETIME, NOT NULL)
    *   `end_date` (DATETIME, NULLABLE)
    *   `status` (VARCHAR - e.g., "active", "cancelled", "expired", "pending_payment")
    *   `payment_gateway_subscription_id` (VARCHAR, NULLABLE) - ID from Stripe, etc.

*   **SubscriptionTiers:** (As previously defined)
    *   `tier_id` (PK, INT, AUTO_INCREMENT)
    *   `name` (VARCHAR, UNIQUE, NOT NULL - e.g., "Free", "Basic", "Premium")
    *   `price_monthly` (DECIMAL, NULLABLE)
    *   `price_annually` (DECIMAL, NULLABLE)
    *   `features` (JSON or TEXT - e.g., RAG agent limits, agent market access, flow builder features)

*   **RagAgents:** (As previously defined - these are the core building blocks, can be private or published to market)
    *   `agent_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - Owner of the agent
    *   `agent_name` (VARCHAR, NOT NULL)
    *   `description` (TEXT, NULLABLE)
    *   `system_prompt` (TEXT, NULLABLE)
    *   `knowledge_base_id` (FK, INT, NOT NULL) - Links to KnowledgeBases table
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `is_public` (BOOLEAN, DEFAULT FALSE) - Indicates if user intends to share/sell, distinct from market listing status.

*   **KnowledgeBases:** (As previously defined)
    *   `kb_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - Owner of the KB
    *   `kb_name` (VARCHAR, NOT NULL)
    *   `vector_store_namespace` (VARCHAR, UNIQUE, NOT NULL)
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

*   **KnowledgeBaseDocuments:** (As previously defined)
    *   `doc_id` (PK, INT, AUTO_INCREMENT)
    *   `kb_id` (FK, INT, NOT NULL)
    *   `file_name` (VARCHAR, NOT NULL)
    *   `file_path_on_server` (VARCHAR, NOT NULL)
    *   `status` (VARCHAR - e.g., "pending_processing", "processing", "processed", "error")
    *   `uploaded_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `metadata` (JSON, NULLABLE)

*   **ChatHistories:** (As previously defined)
    *   `chat_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL)
    *   `agent_id` (FK, INT, NULLABLE) - If chatting with a specific RAG agent
    *   `flow_id` (FK, INT, NULLABLE) - If chatting via an Agent Flow
    *   `session_id` (VARCHAR, NOT NULL, INDEX)
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

*   **ChatMessages:** (As previously defined)
    *   `message_id` (PK, BIGINT, AUTO_INCREMENT)
    *   `chat_id` (FK, INT, NOT NULL)
    *   `sender` (VARCHAR - "user" or "amarax_central_agent" or "flow_step_name")
    *   `message_content` (TEXT, NOT NULL)
    *   `timestamp` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `metadata` (JSON, NULLABLE - e.g., sources for RAG, tool calls, flow step info)

*   **WebRoamingTasks:** (As previously defined)
    *   `task_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL)
    *   `prompt` (TEXT, NOT NULL)
    *   `status` (VARCHAR - "pending", "running", "completed", "failed")
    *   `result` (TEXT, NULLABLE)
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)

*   **AgentMarketListings (New):**
    *   `listing_id` (PK, INT, AUTO_INCREMENT)
    *   `agent_id` (FK, INT, NOT NULL) - The RAG Agent being listed
    *   `publisher_user_id` (FK, INT, NOT NULL) - The user who published the agent
    *   `title` (VARCHAR, NOT NULL)
    *   `market_description` (TEXT, NOT NULL)
    *   `tags` (JSON or TEXT - comma-separated tags for searchability)
    *   `price` (DECIMAL, NULLABLE - 0 for free, or a price)
    *   `version` (VARCHAR, NOT NULL - e.g., "1.0.0")
    *   `status` (VARCHAR - e.g., "published", "unpublished", "under_review")
    *   `published_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `average_rating` (FLOAT, NULLABLE)
    *   `download_count` (INT, DEFAULT 0)

*   **UserAcquiredAgents (New):**
    *   `acquisition_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - The user who acquired the agent
    *   `listing_id` (FK, INT, NOT NULL) - The market listing acquired
    *   `acquired_agent_id` (FK, INT, NOT NULL) - Reference to the actual RagAgent instance (could be a copy or shared with permissions)
    *   `acquired_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `purchase_price` (DECIMAL, NULLABLE)

*   **AgentFlows (New):**
    *   `flow_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - Owner of the flow
    *   `flow_name` (VARCHAR, NOT NULL)
    *   `flow_description` (TEXT, NULLABLE)
    *   `flow_definition` (JSON, NOT NULL) - Stores the graph structure, nodes, connections, configurations of the flow
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `is_public` (BOOLEAN, DEFAULT FALSE) - If user intends to share/sell this flow (could also be listed on market)

*   **AgentFlowExecutions (New - Optional, for logging/tracking flow runs):**
    *   `execution_id` (PK, INT, AUTO_INCREMENT)
    *   `flow_id` (FK, INT, NOT NULL)
    *   `user_id` (FK, INT, NOT NULL)
    *   `start_time` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `end_time` (TIMESTAMP, NULLABLE)
    *   `status` (VARCHAR - "running", "completed", "failed", "paused")
    *   `input_data` (JSON, NULLABLE)
    *   `output_data` (JSON, NULLABLE)
    *   `current_step_id` (VARCHAR, NULLABLE) - If flows can be paused/resumed

## 4. API Endpoints (Flask Backend - RESTful - Updated for Amarax Central)

Base URL: `/api/v1`

**Authentication (`/auth`):** (As previously defined)
*   `POST /auth/register`
*   `POST /auth/login`
*   `POST /auth/logout`
*   `GET /auth/me`

**User Profile (`/users`):** (As previously defined)
*   `GET /users/me`
*   `PUT /users/me`
*   `GET /users/me/subscription`

**Chat (`/chat`):** (Updated to potentially include `flow_id`)
*   `POST /chat/send_message`
    *   Request: `{ "message": "...", "agent_id": null_or_int, "flow_id": null_or_int, "session_id": "..." }`
    *   Response: `{ "reply": "...", "sources": [], "session_id": "...", "flow_step_info": {} }`
*   `GET /chat/history/{session_id}`
*   `GET /chat/sessions`

**RAG Agents (`/rag_agents`):** (As previously defined - these are the building blocks)
*   `POST /rag_agents`
*   `GET /rag_agents` (user's own agents)
*   `GET /rag_agents/{agent_id}`
*   `PUT /rag_agents/{agent_id}`
*   `DELETE /rag_agents/{agent_id}`
*   `POST /rag_agents/{agent_id}/kb/upload_doc`
*   `GET /rag_agents/{agent_id}/kb/documents`

**Agent Market (`/market/agents` - New):**
*   `POST /market/agents/{agent_id}/publish`: Publish a user's RAG agent to the market (creates a listing).
    *   Request: `{ "title": "...", "market_description": "...", "tags": ["tag1", "tag2"], "price": 0.00, "version": "1.0.0" }`
*   `GET /market/agents`: List all published agents (searchable, filterable).
*   `GET /market/agents/{listing_id}`: Get details of a specific market listing.
*   `PUT /market/agents/{listing_id}`: Update a market listing (publisher only).
*   `POST /market/agents/{listing_id}/acquire`: Acquire/purchase an agent from the market.
*   `GET /market/agents/my_published`: List agents published by the current user.
*   `GET /market/agents/my_acquired`: List agents acquired by the current user.

**Agent Flows (`/flows` - New):**
*   `POST /flows`: Create a new agent flow.
    *   Request: `{ "flow_name": "...", "flow_description": "...", "flow_definition": { ... } }`
*   `GET /flows`: List user's agent flows.
*   `GET /flows/{flow_id}`: Get details of a specific agent flow.
*   `PUT /flows/{flow_id}`: Update an agent flow definition.
*   `DELETE /flows/{flow_id}`: Delete an agent flow.
*   `POST /flows/{flow_id}/execute`: Execute an agent flow (could be long-running, might return a task ID for status check, or interact via chat endpoint).
*   `GET /flows/executions/{execution_id}`: Get status/result of a flow execution (if applicable).

**Knowledge Bases (`/knowledge_bases`):** (As previously defined)

**Subscriptions (`/subscriptions`):** (As previously defined)

**Downloadable Client (`/client`):** (As previously defined)

**Web Roaming (`/web_roam`):** (As previously defined)

(Further sections will detail UI/UX, Downloadable Client, RAG, Web Roaming, Subscriptions, Implementation Roadmap, all updated for Amarax Central)




## 5. UI/UX Design (IDE-Style - Updated for Amarax Central)

The user interface will be heavily inspired by the **Thrae IDE layout** (user-provided image `Skjermbilde 2025-02-20 104826.png`) and will embody the **Amarax Central** branding. Color schemes will be derived from the new set of visual references provided by the user, aiming for a modern, professional, and potentially customizable (dark/light themes) look and feel. Development will use React or Next.js.

### 5.1. Main Layout Components (Thrae IDE Inspired):

*   **Top Menu Bar:** Standard application menu (File, Edit, View, Go, Run, Terminal, Help - contextualized for Amarax Central). Will display "Amarax Central by Amarax Productions™️©️" branding subtly.
*   **Left Sidebar (Icon-based Navigation):** Icons for switching between major views/tools:
    *   **Explorer/Projects:** Manage user projects, RAG knowledge bases, agent configurations, and flow definitions.
    *   **Search:** Global search across projects, agents, market, documentation.
    *   **Source Control (Future):** Placeholder for potential Git integration for managing agent code/configurations.
    *   **Chat/Interact:** Main interaction panel for chatting with general Amarax agents or specific user-created/acquired agents/flows.
    *   **Agent Builder (RAG & Basic):** Interface for creating and configuring individual RAG agents.
    *   **Flow Builder:** Visual interface for designing complex agent workflows (connecting multiple agents, tools, logic blocks).
    *   **Agent Market:** Browse, search, publish, and acquire agents.
    *   **Training Management (Future/Advanced):** Interface for fine-tuning models or managing specialized training data (as per original Sophie plan).
    *   **Settings:** User account, platform preferences, API keys, subscription management.
*   **Primary Content Area:** Dynamically changes based on the selected view. This is the main workspace.
    *   **Welcome/Dashboard View:** Customizable dashboard. Quick access to recent projects, create new agent/flow, browse market, tutorials.
    *   **Chat View:** Large panel for chat. Input area supporting rich text, file uploads (for context), and potentially voice input. Conversation history with clear distinction between user and agent messages. Support for rendering rich outputs (tables, charts, images) from agents.
    *   **Agent Builder View:** Form-based and potentially guided interface for defining RAG agent properties (name, prompt, KB linking). UI for managing KB documents (upload, process, delete).
    *   **Flow Builder View:** A canvas-based visual editor. Drag-and-drop nodes representing agents, tools (e.g., web search, code execution), logic (conditions, loops), and data transformations. Connectors to define the flow of execution and data. Properties panel for configuring selected nodes.
    *   **Agent Market View:** Grid or list view of available agents. Filtering and sorting options. Detailed view for each agent (description, publisher, version, price, reviews, usage examples). Interface for publishing user-created agents.
    *   **"Amarax Terminal" View:** An embedded terminal-like interface for interacting with sandboxed command-line tools available to agents or for advanced users to manage certain aspects of their environment (within strict security boundaries).
*   **Right Sidebar/Panel (Contextual):** Provides details, configuration options, or help related to the active item in the primary content area (e.g., selected agent details, flow node properties, document preview).
*   **Status Bar (Bottom):** Displays current active agent/flow, LLM model in use, background task status (e.g., KB indexing, flow execution), notifications, and potentially a quick access to system logs or console.

### 5.2. Key Screens and Workflows (Updated for Amarax Central):

*   **User Registration/Login:** Branded for Amarax Central.
*   **Dashboard:** Amarax Central branded. Personalized with user activity.
*   **Chat Interface:** As described above, using Amarax color schemes.
*   **RAG Agent Creation Workflow:** As previously defined, styled for Amarax Central.
*   **Agent Flow Creation Workflow:**
    1.  User selects "Create New Flow" from Explorer or Dashboard.
    2.  Enters flow name and description.
    3.  Opens the Flow Builder canvas.
    4.  User drags agents (own, acquired from market), tools, and logic blocks onto the canvas.
    5.  User connects nodes to define execution order and data flow.
    6.  User configures individual nodes (e.g., prompts for agents, parameters for tools).
    7.  Save and (optionally) test the flow within the builder.
*   **Agent Market Workflow:**
    *   **Browsing/Searching:** Users can search by keyword, tags, publisher, rating. Filters for free/paid.
    *   **Viewing Listing:** Detailed page with agent description, capabilities, version history, reviews, publisher info, and how to use/integrate.
    *   **Acquiring:** Simple one-click for free agents. Integration with subscription/payment for paid agents.
    *   **Publishing:** User selects one of their RAG agents or Flows. Fills out a form for market listing (title, description, tags, price, version notes). Submits for review (if applicable) or publishes directly.
*   **Subscription Management Page:** Branded for Amarax Central. Clearly outlines features available at each tier, especially regarding Agent Market access (e.g., number of agents to publish/acquire, access to premium agents) and Flow Builder capabilities.
*   **Downloadable Client Page:** Branded for Amarax Central. Highlights features and provides download links.

### 5.3. Color Schemes and Visual Style:

*   The UI will primarily use the dark, modern, and professional color palettes evident in the newly provided "Sophie" images, adapted for the "Amarax Central" branding. This typically involves dark backgrounds (deep blues, grays, blacks) with vibrant accent colors (e.g., electric blues, purples, oranges, greens as seen in images) for interactive elements, highlights, and branding. Specific hex codes will be extracted from the reference images or a complementary palette will be designed.
*   Typography will be clean and modern, prioritizing readability.
*   Iconography will be consistent (e.g., using a single library like Lucide Icons, customized as needed).

## 6. Downloadable Desktop Client Design (Updated for Amarax Central)

*   **Branding:** Will carry full Amarax Central branding and visual style, consistent with the web platform.
*   **Technology:** Decision between Python/CustomTkinter and Electron remains, but with the increased complexity and desire for a UI consistent with the web (Thrae IDE style), **Electron might be favored** if development resources allow for building a consistent React/Next.js based shell.
*   **Functionality:** As previously defined, but with access to the Agent Market (viewing, potentially acquiring if API allows) and ability to run/interact with acquired agents and flows. The primary interface will mirror the web platform's chat and agent interaction views.

(Sections 7-9 on RAG, Web Roaming, Subscriptions will be reviewed for consistency with Amarax Central but core technical details likely remain similar. Section 10 on Implementation Roadmap will need significant updates.)




## 10. Implementation Roadmap (Phased Approach - Updated for Amarax Central)

Given the expanded scope including the Agent Market and Flow Builder, a carefully phased implementation is crucial.

**Phase 1: Core Platform & Chat (MVP for Amarax Central)**
1.  **Branding & Basic Structure:**
    *   Setup Flask backend with Amarax Central configurations.
    *   Setup Next.js frontend with basic Thrae IDE layout and Amarax Central branding/colors.
    *   Implement user registration/login.
2.  **Core Chat Functionality:**
    *   Basic database schema for users, chat history/messages.
    *   Integrate core chat logic (LLM interface) into the backend.
    *   Implement basic chat UI in the frontend.
3.  **Initial Deployment (Internal/Staging):** Deploy this core MVP for early testing.

**Phase 2: RAG Agent Creation & Basic Subscription Framework**
1.  **RAG Implementation:**
    *   Implement RAG sub-system (data ingestion, vector store, retrieval).
    *   Develop UI for creating RAG agents and managing their knowledge bases.
    *   Integrate RAG agents into the chat interface.
2.  **Subscription Foundation:**
    *   Implement basic subscription tiers in the database.
    *   Develop UI for users to see tiers (no payment processing yet).
    *   Basic access control placeholders based on tiers (e.g., limit on number of RAG agents).

**Phase 3: Agent Flow Builder (Core Functionality)**
1.  **Flow Engine Backend:**
    *   Design and implement database models for Agent Flows and their definitions.
    *   Develop backend logic for parsing and executing simple flow definitions (e.g., sequential agent calls).
2.  **Visual Flow Builder UI (Basic):**
    *   Develop the canvas-based UI for dragging/dropping nodes (representing RAG agents initially).
    *   Implement functionality to connect nodes and save flow definitions.
    *   Allow execution of simple flows via the UI, with output displayed in chat or a dedicated panel.
3.  **Integration:** Allow flows to be selected and interacted with via the main chat interface.

**Phase 4: Agent Market (Core Functionality)**
1.  **Market Backend:**
    *   Implement database models for Agent Market Listings and User Acquired Agents.
    *   Develop APIs for publishing RAG agents (and later, flows) to the market.
    *   Develop APIs for browsing, searching, and acquiring agents from the market.
2.  **Market UI:**
    *   Develop UI for browsing and searching market listings.
    *   Develop UI for viewing agent details and acquiring them.
    *   Develop UI for users to manage their published and acquired agents.
3.  **Initial Publishing:** Allow users to publish their created RAG agents.

**Phase 5: Downloadable Client & Full Subscription Integration**
1.  **Downloadable Client:**
    *   Develop the downloadable desktop client (Electron preferred for UI consistency) with Amarax Central branding.
    *   Client connects to backend for chat, RAG agent use, flow execution, and market browsing.
2.  **Full Subscription & Payment:**
    *   Integrate with a payment gateway (e.g., Stripe).
    *   Implement full subscription lifecycle management (subscribe, upgrade, cancel).
    *   Enforce feature gating based on active subscription tiers (e.g., advanced Flow Builder features, number of market publishes/acquires).

**Phase 6: Advanced Features & Polish**
1.  **Web Roaming Module:** Implement as previously designed.
2.  **Advanced Flow Builder Features:** Add more node types (tools, logic blocks, data transformations), versioning, sharing.
3.  **Advanced Agent Market Features:** Reviews, ratings, more complex search filters, analytics for publishers.
4.  **Enhanced Training Environment:** If model fine-tuning is pursued.
5.  **UI/UX Refinements:** Based on user feedback, fully realize the Thrae IDE vision with Amarax color schemes.
6.  **Security Hardening & Scalability Optimizations.**

**Phase 7: Public Launch & Ongoing Maintenance**
1.  Final testing and quality assurance.
2.  Public deployment.
3.  Monitoring, user support, and iterative updates.

This roadmap prioritizes getting a core, usable platform with the new branding and key differentiators (Flow Builder, Market) available progressively.

## 11. Next Steps (Post-Design Document v1.1)

-   Proceed to **Step 4 (004): Detailed Design of Web Interface and Backend for Amarax Central**, which involves creating detailed mockups, fully specifying all API request/response payloads, and finalizing all technical choices for each module.
-   Begin setting up project repositories and development environments.

---
*Document End*




## 5. UI/UX Design (IDE-Style - Updated for Amarax Central)

...(Existing content from previous update)...

### 5.4. Detailed UI Component Descriptions & Visual Styling (Amarax Central)

This section details specific UI components, drawing inspiration from the user-provided images (especially the latest batch showing dark themes, vibrant accents, and specific UI elements like chat windows and icon styles).

**Branding Elements:**
*   **Logo:** The "Amarax" flame logo and "Amarax Central" logotype (as seen in `863efae0-021a-47cf-a3b9-c39375228785.png`, `fb1f5a88-37f9-473d-acf4-fbb103cef293.png`, `f1ad7a55-e0ed-4d3d-8cdd-aab229c4d665.png`) will be prominently displayed in the login screen, top menu bar (subtly), and potentially as a loading animation.
*   **Copyright/Ownership:** "Amarax Productions™️©️" will be included in the footer or an "About" section.

**Color Palette (Derived from latest images like `83f64728-df1c-474e-b043-b1fd2d0a95f0.png`, `c1f383b7-5391-4da2-8176-5c85eb96a1f0.png`, `8a704f48-43ab-491a-98a0-9992e94eca75.png`):
*   **Primary Backgrounds:** Dark grays, deep blues, near-blacks (e.g., `#1A1A2E`, `#16213E`, `#0F3460`).
*   **Secondary Backgrounds/Panels:** Slightly lighter shades of dark gray or blue for contrast within panels (e.g., `#2D3748`, `#1E293B`).
*   **Accent Colors (Vibrant):**
    *   Orange/Red-Orange (e.g., `#FF6700`, `#FF4500` - for primary call-to-actions, highlights, icons as seen in `de0f18d0-b5e2-4fdf-a6d4-d2dfcf344549.png`, `64010c19-3247-48ff-be74-76af6d859ad8.png`).
    *   Bright Blue/Cyan (e.g., `#00BFFF`, `#50C878` - for secondary actions, information, links, some icons as seen in `8a704f48-43ab-491a-98a0-9992e94eca75.png` chat bubbles).
    *   Potentially other vibrant colors (purples, greens) for specific states or elements if they fit the overall aesthetic.
*   **Text Colors:** Light grays, off-whites for primary text on dark backgrounds (e.g., `#E0E0E0`, `#F5F5F5`). Accent colors for headings or important text snippets.
*   **Borders/Dividers:** Subtle darker or slightly lighter shades of the background, or a thin accent color line.

**Typography:**
*   **Primary Font:** A clean, modern sans-serif font (e.g., Inter, Roboto, Open Sans) for readability.
*   **Headings:** Slightly bolder weight, potentially using an accent color.
*   **Code/Terminal Font:** Monospaced font (e.g., Fira Code, Source Code Pro).

**Specific UI Components (Referencing provided images):

*   **Chat Interface (`c1f383b7-5391-4da2-8176-5c85eb96a1f0.png`, `8a704f48-43ab-491a-98a0-9992e94eca75.png`):**
    *   **Message Bubbles:** User messages on one side (e.g., right, with a blue accent as in `8a704f48-43ab-491a-98a0-9992e94eca75.png`), AI responses on the other (e.g., left, slightly different background shade).
    *   **Input Area:** Clean text input field with a prominent send button (e.g., paper plane icon with accent color).
    *   **Suggested Actions/Buttons:** Below chat messages for quick replies or actions (as in `8a704f48-43ab-491a-98a0-9992e94eca75.png`).
    *   **Header:** Displaying current agent name or "Chat with Amarax". Close/minimize button.
    *   **Scrollbar:** Styled to match the dark theme.

*   **IDE-Style Panels (Inspired by `Skjermbilde 2025-02-20 104826.png` and general dark IDEs):
    *   **File Explorer/Tree View:** Standard tree structure for projects, agents, flows. Icons for different file/item types. Subtle hover effects.
    *   **Editor Area (for Flow Builder, Agent Config):** Main workspace. For Flow Builder, this will be a canvas. For configurations, it will be forms or structured text editors.
    *   **Terminal Panel:** Dark background, monospaced font, standard terminal prompt.

*   **Buttons (Inspired by `40d82cb6-265c-4e92-a0ec-690a8a970b72.png` - UI Design elements):
    *   **Primary Buttons:** Solid fill with accent color (e.g., orange), white/light text. Subtle hover/active states (e.g., slightly darker or brighter).
    *   **Secondary Buttons:** Outlined with accent color, or a less prominent solid fill.
    *   **Icon Buttons:** Clean icons with minimal or no text, clear hover states.

*   **Icons (Inspired by `de0f18d0-b5e2-4fdf-a6d4-d2dfcf344549.png`, `1c6223a5-7e15-44fb-8ab5-c097414aa99c.png`):
    *   Style: Modern, slightly rounded, potentially with a subtle 3D or neumorphic touch if it fits the overall flat design. The orange icons in `de0f18d0-b5e2-4fdf-a6d4-d2dfcf344549.png` are a strong reference.
    *   Usage: For sidebar navigation, action buttons, status indicators.
    *   Consistency: Use a single icon set (e.g., Lucide, Feather Icons, or custom designed based on references).

*   **Forms & Input Fields:**
    *   Clean, minimalist design. Dark background with lighter text.
    *   Accent color for focus state or validation messages.

*   **Cards (for Agent Market, Dashboard items):
    *   Slightly raised panels with subtle shadows or borders.
    *   Space for an icon/image, title, description, and action buttons.

*   **Modals/Pop-ups:** Consistent dark theme, clear call-to-action buttons.

*   **Agent Market & Flow Builder Specifics:**
    *   **Agent Market Cards:** Display agent icon (user-uploaded or generated), name, brief description, publisher, rating, price. Use accent colors for tags or calls to action.
    *   **Flow Builder Nodes:** Rectangular or rounded-rectangular nodes with clear titles, input/output ports. Color-coding for different node types (e.g., RAG agent, tool, logic). Connections as clean lines or curves.

### 5.5. Accessibility Considerations:
*   Ensure sufficient color contrast between text and backgrounds.
*   Support keyboard navigation.
*   Use ARIA attributes where appropriate.
*   Allow for text resizing if possible.

(This section will be further expanded with more detailed mock-up descriptions for each key screen as the design progresses.)

## 12. Detailed API Payload Specifications (Example - to be expanded for all endpoints)

This section will detail the exact JSON request and response structures for each API endpoint defined in Section 4. 

**Example: `POST /api/v1/auth/register`**
*   **Request Body:**
    ```json
    {
      "username": "testuser",
      "email": "test@example.com",
      "password": "SecurePassword123!"
    }
    ```
*   **Success Response (201 Created):**
    ```json
    {
      "message": "User registered successfully. Please check your email to verify your account.",
      "user_id": 123
    }
    ```
*   **Error Response (400 Bad Request - e.g., validation error):**
    ```json
    {
      "error": "Validation Error",
      "details": {
        "email": ["Invalid email format."],
        "password": ["Password must be at least 8 characters long."]
      }
    }
    ```
*   **Error Response (409 Conflict - e.g., username/email exists):**
    ```json
    {
      "error": "User already exists",
      "message": "A user with this username or email already exists."
    }
    ```

**(This will be systematically done for all endpoints: `/auth/login`, `/users/me`, `/chat/send_message`, `/rag_agents`, `/market/agents/*`, `/flows/*`, etc.)**

---
*Further updates to follow in subsequent steps, focusing on implementation details for frontend and backend.*




## 12. Detailed API Payload Specifications (Continued)

**(Continuing from previous example for `/auth/register`)**

**Example: `POST /api/v1/market/agents/{agent_id}/publish`**
*   **Path Parameter:** `agent_id` (integer) - ID of the RAG agent to be published.
*   **Request Body:**
    ```json
    {
      "title": "My Awesome Sales Agent",
      "market_description": "This agent is expertly trained to handle sales inquiries and guide customers through the purchase process.",
      "tags": ["sales", "customer_service", "ecommerce"],
      "price": 19.99, // Nullable, 0 or null for free
      "version": "1.0.0"
    }
    ```
*   **Success Response (201 Created):**
    ```json
    {
      "message": "Agent listing created successfully and is pending review/published.",
      "listing_id": 56,
      "agent_id": 12,
      "title": "My Awesome Sales Agent",
      "status": "pending_review" // or "published"
    }
    ```
*   **Error Response (400 Bad Request - Validation):**
    ```json
    {
      "error": "Validation Error",
      "details": {
        "title": ["Title is required."],
        "price": ["Price must be a non-negative number."]
      }
    }
    ```
*   **Error Response (404 Not Found - Agent not found or not owned by user):**
    ```json
    {
      "error": "Not Found",
      "message": "Agent not found or you do not have permission to publish it."
    }
    ```

**Example: `GET /api/v1/market/agents` (List published agents)**
*   **Query Parameters (Optional):**
    *   `search` (string): Keyword search for title, description, tags.
    *   `tag` (string): Filter by a specific tag.
    *   `free` (boolean): Filter for free agents.
    *   `min_rating` (float): Minimum average rating.
    *   `sort_by` (string): e.g., "published_at", "rating", "download_count".
    *   `order` (string): "asc" or "desc".
    *   `page` (integer): For pagination.
    *   `limit` (integer): Items per page.
*   **Success Response (200 OK):**
    ```json
    {
      "pagination": {
        "total_items": 150,
        "total_pages": 15,
        "current_page": 1,
        "per_page": 10
      },
      "listings": [
        {
          "listing_id": 56,
          "agent_id": 12,
          "publisher_user_id": 1,
          "title": "My Awesome Sales Agent",
          "market_description_snippet": "This agent is expertly trained...",
          "tags": ["sales", "customer_service"],
          "price": 19.99,
          "version": "1.0.0",
          "average_rating": 4.5,
          "download_count": 120,
          "published_at": "2025-05-10T10:00:00Z"
        }
        // ... more listings
      ]
    }
    ```

**Example: `POST /api/v1/flows` (Create a new agent flow)**
*   **Request Body:**
    ```json
    {
      "flow_name": "Customer Onboarding Workflow",
      "flow_description": "A multi-step flow to welcome new users and guide them through setup.",
      "flow_definition": {
        "nodes": [
          {"id": "start", "type": "start_node", "position": {"x": 50, "y": 50}, "data": {}},
          {"id": "welcome_agent", "type": "rag_agent_node", "position": {"x": 50, "y": 200}, "data": {"agent_id": 7, "prompt_template": "Welcome {username}! Ask me anything about getting started."}},
          {"id": "setup_guide_tool", "type": "tool_node", "position": {"x": 300, "y": 200}, "data": {"tool_name": "fetch_setup_guide", "params": {"topic": "initial_config"}}},
          {"id": "conditional_feedback", "type": "condition_node", "position": {"x": 300, "y": 350}, "data": {"condition": "output_of_setup_guide_tool contains \"error\""}},
          {"id": "end_success", "type": "end_node", "position": {"x": 50, "y": 500}, "data": {"status": "success"}},
          {"id": "end_error", "type": "end_node", "position": {"x": 300, "y": 500}, "data": {"status": "error"}}
        ],
        "edges": [
          {"id": "e1", "source": "start", "target": "welcome_agent"},
          {"id": "e2", "source": "welcome_agent", "target": "setup_guide_tool"},
          {"id": "e3", "source": "setup_guide_tool", "target": "conditional_feedback"},
          {"id": "e4", "source": "conditional_feedback", "target": "end_error", "condition_type": "true_branch"},
          {"id": "e5", "source": "conditional_feedback", "target": "end_success", "condition_type": "false_branch"}
        ]
      }
    }
    ```
*   **Success Response (201 Created):**
    ```json
    {
      "message": "Agent flow created successfully.",
      "flow_id": 3,
      "flow_name": "Customer Onboarding Workflow"
    }
    ```

## 13. Key Technology Choices (Summary)

*   **Backend Framework:** Flask (Python)
*   **Frontend Framework/Library:** Next.js (React, TypeScript)
*   **Database:** MySQL (initially, as per Flask template) or PostgreSQL (for more advanced features if needed).
*   **User Interface Styling:** Tailwind CSS (comes with Next.js template), shadcn/ui components, custom CSS for Amarax branding.
*   **Vector Store (for RAG):** ChromaDB or FAISS (to be decided based on ease of integration and performance).
*   **LLM Interaction:** Direct API calls to chosen LLM providers (e.g., OpenAI, Anthropic) or local models via libraries like `transformers`.
*   **Authentication:** Flask-Bcrypt for password hashing, Flask-JWT-Extended for token-based authentication.
*   **Web Roaming:** Playwright (Python library).
*   **Downloadable Client (Option 1 - Web Tech):** Electron (packaging the Next.js frontend).
*   **Downloadable Client (Option 2 - Python Native):** Python with CustomTkinter (if a lighter, more Python-centric client is preferred, though UI consistency with web might be harder).
*   **Deployment (Backend):** Docker containers, potentially managed by Kubernetes or a PaaS (e.g., Heroku, AWS Elastic Beanstalk).
*   **Deployment (Frontend):** Vercel (ideal for Next.js) or static hosting on services like AWS S3/CloudFront, Netlify.

---
*This concludes the primary updates for the detailed design phase based on current information. Further refinements will occur during implementation.*



## 5. UI/UX Design (IDE-Style - Updated for Amarax Central)

...(Existing content from previous update)...

### 5.6. Mascot Integration (Amarax Robots - `9b040cde-2b66-49cf-b5ac-5e9e8b4de165.png`)

*   The Amarax robot mascots (male/blue and female/purple versions) will be used strategically throughout the platform to enhance brand personality and user engagement.
*   **Usage Scenarios:**
    *   **Welcome Screens/Onboarding:** Mascots can guide new users through initial setup or feature tours.
    *   **Loading Screens/Placeholders:** A mascot animation or static image can be used during loading states.
    *   **Help/Support Sections:** Mascots can be part of the visual language for FAQs or support chat.
    *   **Empty States:** When a user has no agents, flows, or projects, a friendly mascot illustration can fill the empty space with a call to action.
    *   **Error Pages:** A sympathetic mascot expression for error states.
    *   **Marketing Materials/Downloadable Client Branding:** Mascots will be key visual elements.
*   **Style:** The 3D rendered style of the mascots should be maintained. If 2D versions are needed for UI elements, they should be high-quality illustrations derived from the 3D models.

### 5.7. Agent Core Diagram Integration (`9d2f6bbf-3426-4f9b-822a-7db1d716e047.png`)

*   The "Agent Core" diagram, illustrating concepts like Memory, Tools, Other Agents, Users, Actions, Environment, Goals, and Planning, can be used in:
    *   **Documentation:** Explaining the capabilities of Amarax Central agents.
    *   **Agent Builder UI:** As a conceptual overview or a high-level map when designing complex agents or flows.
    *   **Tutorials/Guides:** To educate users on agent architecture.
*   The visual style of the diagram (clean icons, purple/blue accents, clear connections) should be maintained and adapted to fit within the platform's UI.

### 5.8. UI Component Styling Refinements (Based on `13ad5dd0-ac3f-44c3-83db-4f14bad3e6e7.png`, `a233cfb9-5d77-4f9c-9f11-14847d1643b5.png`, `14775819-51f6-4070-838f-89d16c8600d7.png`, `140db5a8-b85a-43ec-831e-ab04922a62f9.png`, `0e8607e9-5fe5-4c6b-9578-b40c124e2c0c.png`)

*   **Feature Cards/Sections (`a233cfb9-5d77-4f9c-9f11-14847d1643b5.png`):**
    *   Clean, card-based layout for highlighting features (Agent, Memory, Integration, Live Preview).
    *   Use of toggle switches for enabling/disabling features.
    *   Clear iconography and concise descriptions.
    *   Buttons for actions like "Shared"/"Private" selection.
*   **Navigation Menus (`13ad5dd0-ac3f-44c3-83db-4f14bad3e6e7.png` - API Keys example):
    *   Dark sidebar with clear, well-spaced text labels and icons.
    *   Highlighting for the active menu item (e.g., purple accent for "API Keys").
*   **Dashboard/Deployment Cards (`14775819-51f6-4070-838f-89d16c8600d7.png` - ForgeKit example):
    *   Grid layout for displaying deployable apps/agents/templates.
    *   Each card includes a title, description, author/project, and a clear "Deploy" or action button.
    *   Use of tags or categories (Recommended, Most Popular, All Templates).
*   **Text-to-App Generation UI (`140db5a8-b85a-43ec-831e-ab04922a62f9.png`):
    *   Simple, prominent text input area for describing the application.
    *   Clear call-to-action button ("Start Free").
    *   Visual feedback for processing (e.g., "Fetching code from stardust...").
    *   Pricing tier cards below, clearly outlining features for each package.
*   **Workflow/UI Preview (`0e8607e9-5fe5-4c6b-9578-b40c124e2c0c.png` - Manus AI example):
    *   Excellent reference for the Thrae IDE style, showing integrated chat, workflow visualization, and potentially a live preview of the UI being built/configured.
    *   The multi-panel layout with chat on the left, main content (workflow/preview) in the center, and potentially settings/details on the right is a key inspiration.

## 0. Document Version (Update)

-   **Version:** 1.2 (Incorporating final visual references and mascot integration)
-   **Date:** May 14, 2025
-   **Status:** Final Design - Ready for Implementation

(The rest of the document, including API payloads and technology choices, remains as defined in version 1.1, as these new images primarily impact UI/UX and branding presentation rather than core backend logic or API contracts at this stage. The implementation roadmap also remains valid, with these visual details to be incorporated during frontend development.)

---
*This concludes the detailed design phase. The document is now considered final for guiding implementation.*



## 5. UI/UX Design (IDE-Style - Updated for Amarax Central)

...(Existing content from previous update)...

### 5.9. Advanced Mascot Usage & Animation Ideas (Incorporating latest user feedback and images like `6a143312-fa2c-46eb-9499-ed362b69a6d2.png` to `ff3ae681-98d7-4c91-9848-c46d3f0da263.png`)

*   **Differentiated Agent Avatars:** The various Amarax robot designs (e.g., the blue one with a water drop, the pink one with a heart, the blue one with an infinity symbol) can be used as distinct avatars for different types of pre-built agents or user-created agents within the Agent Market or Flow Builder. This provides immediate visual differentiation.
*   **Animated Branding Elements:**
    *   **Loading Sequences:** Simple animations of the mascots (e.g., the two robots interacting, or one assembling itself) can be used for loading screens. These could be implemented as high-quality GIFs or using libraries like Lottie for SVG animations if feasible.
    *   **Interactive Hover Effects:** Mascots in static locations (e.g., help sections) could have subtle hover animations (e.g., waving, eyes blinking).
    *   **Promotional Banners/Sections:** Animated GIFs or short video snippets (if externally produced and embeddable) featuring the mascots could be used on landing pages or within the platform to highlight new features or promotions.
*   **Mascot States:** Different poses or expressions of the mascots can be used to convey system status or agent mood (e.g., a happy mascot for successful task completion, a thoughtful one for processing, a slightly concerned one for errors).
*   **Consideration for Oldschool Flash/GIF Feel:** While direct Flash implementation is outdated, the *spirit* of engaging, lively animations can be achieved with modern web technologies. Short, looping animations (GIFs, animated WebP, or CSS/JS sprite animations) can evoke this feel.
*   **Technical Implementation for Web:**
    *   **Static Images:** High-quality PNGs (like those provided) for avatars, illustrations.
    *   **GIFs:** For simple, looping animations.
    *   **CSS Animations/Transitions:** For UI element interactions and subtle mascot movements.
    *   **JavaScript Animation Libraries (e.g., GSAP, Framer Motion):** For more complex scripted animations if needed, though this adds development overhead.
    *   **Lottie Animations:** If vector-based animations are created (e.g., in Adobe After Effects with the LottieFiles plugin), they can be rendered efficiently on the web.

(This section further refines mascot integration, ensuring the latest visual assets and animation ideas are captured for frontend development.)



### 5.X. Mascot Integration and Animation Strategy (New)

To enhance the user experience and strengthen the **Amarax Central** brand identity, the platform will incorporate the provided robot mascot images in various engaging ways. The visual style of these mascots, as seen in the latest image uploads (e.g., `00472b76-5594-4937-8fee-8b9a179b24f3.png`, `f1ad7a55-e0ed-4d3d-8cdd-aab229c4d665.png`, `83f64728-df1c-474e-b043-b1fd2d0a95f0.png`, `863efae0-021a-47cf-a3b9-c39375228785.png`, `0e8607e9-5fe5-4c6b-9578-b40c124e2c0c.png`, `140db5a8-b85a-43ec-831e-ab04922a62f9.png`, `14775819-51f6-4070-838f-89d16c8600d7.png`, `a233cfb9-5d77-4f9c-9f11-14847d1643b5.png`, `13ad5dd0-ac3f-44c3-83db-4f14bad3e6e7.png`, `9d2f6bbf-3426-4f9b-822a-7db1d716e047.png`, `9b040cde-2b66-49cf-b5ac-5e9e8b4de165.png` and the set provided on May 14, 2025 04:07:46 UTC), will be consistently applied.

**Key aspects of mascot integration include:**

1.  **Visual Representation of Agents:** Different variations of the robot mascots will be used to visually distinguish between different types of AI agents available on the platform (e.g., RAG agents, specialized agents from the Agent Market, or agents performing specific tasks within the Flow Builder). This will help users quickly identify and understand the nature of an agent.
    *   *Example:* A more robust, analytical-looking robot for data processing agents, a friendly, communicative robot for chat agents, etc.

2.  **Branding and Engagement through Animation:** To create a "living and engaging structure" as requested, simple animations will be explored for the mascots. These will be primarily web-friendly animations:
    *   **GIFs:** Short, looping animations for loading states, success notifications, or as part of static illustrations to add subtle movement.
    *   **CSS/JavaScript Animations:** For more interactive animations, such as hover effects on mascot icons, or simple animated sequences on the landing page or in dashboards to enhance visual appeal.
    *   While complex Flash-style videos are beyond the current scope of direct generation, the *concept* of dynamic, animated mascots will guide the design of these simpler animations.

3.  **Placement:** Mascots will be strategically placed throughout the UI:
    *   On the main landing page to welcome users.
    *   Within the Agent Market to showcase different agent types.
    *   As avatars or icons associated with specific agents or agent functionalities.
    *   In tutorials or help sections to guide users.

4.  **Consistency with Amarax Central Branding:** All mascot usage and animations will be consistent with the overall branding of "Amarax Central" by "Amarax Productions™️©️", adhering to the established color palettes (dark themes with vibrant purple/blue accents) and modern aesthetic.

This strategy aims to make the Amarax Central platform not only powerful and functional but also visually appealing, memorable, and enjoyable for users to interact with.



### Appendix A: Mascot Image References

The following images, located in the `/home/ubuntu/upload/` directory, serve as primary references for the Amarax Central robot mascots and their visual style. These images will guide the design of agent representations and branding animations:

*   `00472b76-5594-4937-8fee-8b9a179b24f3.png`
*   `f1ad7a55-e0ed-4d3d-8cdd-aab229c4d665.png`
*   `83f64728-df1c-474e-b043-b1fd2d0a95f0.png`
*   `863efae0-021a-47cf-a3b9-c39375228785.png`
*   `0e8607e9-5fe5-4c6b-9578-b40c124e2c0c.png`
*   `140db5a8-b85a-43ec-831e-ab04922a62f9.png`
*   `14775819-51f6-4070-838f-89d16c8600d7.png`
*   `a233cfb9-5d77-4f9c-9f11-14847d1643b5.png`
*   `13ad5dd0-ac3f-44c3-83db-4f14bad3e6e7.png`
*   `9d2f6bbf-3426-4f9b-822a-7db1d716e047.png`
*   `9b040cde-2b66-49cf-b5ac-5e9e8b4de165.png`
*   `6a143312-fa2c-46eb-9499-ed362b69a6d2.png` (from user message on May 14 2025 04:07:46 +0000)
*   `d59c3adb-6548-4cff-aa05-a3f3e7eb55ed.png` (from user message on May 14 2025 04:07:46 +0000)
*   `95d9830a-af96-40ca-9f9c-f62f25eaa40b.png` (from user message on May 14 2025 04:07:46 +0000)
*   `170b124d-d58b-41de-9442-9891b647abbb.png` (from user message on May 14 2025 04:07:46 +0000)
*   `82bf1449-0119-4414-8d8c-88c83234c3b5.png` (from user message on May 14 2025 04:07:46 +0000)
*   `ff3ae681-98d7-4c91-9848-c46d3f0da263.png` (from user message on May 14 2025 04:07:46 +0000)

(This list should be cross-referenced with the full set of images provided by the user on May 14, 2025, around 04:07:46 UTC, and any other relevant mascot images from the `/home/ubuntu/upload/` directory.)




### 5.2. Detailed UI/UX for Key Components

This section expands on the UI/UX design for specific components within the Amarax Central platform, focusing on the Thrae IDE layout, Amarax branding, and strategic mascot integration.

#### 5.2.1. Chat Panel (`ChatPanel.tsx`)

*   **Layout:**
    *   Occupies a significant portion of the main content area, typically center or center-right, depending on the Thrae IDE configuration chosen by the user (e.g., if other panels like a terminal or file explorer are open).
    *   **Header:** Displays the name of the current chat session or the active RAG agent. May feature a small, subtle mascot icon representing the agent type (e.g., a friendly communication bot mascot like `mascot_style_1.png` if it's a general chat, or a specialized mascot if a specific agent is active).
    *   **Message Display Area:** A scrollable area showing chat history. User messages are typically right-aligned, and agent messages are left-aligned.
        *   Agent messages can be prefixed with a small avatar of the active agent (e.g., `mascot_style_1.png` or `mascot_style_2.png` depending on the agent's personality/type). These avatars should be consistent with the mascot strategy.
        *   Code blocks, lists, and other formatted content should be rendered clearly.
        *   Sources for RAG-generated responses should be displayed discreetly, perhaps expandable or as tooltips.
    *   **Input Area:** A text input field at the bottom for the user to type messages. Buttons for sending the message, attaching files (if applicable), and potentially accessing quick commands or voice input.
    *   **Contextual Actions:** Buttons or icons for clearing chat, exporting chat, or managing chat session settings.
*   **Mascot Integration:**
    *   **Agent Avatars:** Each agent (general Sophie or specialized RAG agents) should have a distinct mascot avatar displayed next to their messages. This helps in visually differentiating between multiple agents or the user.
    *   **Loading/Thinking Indicators:** When the agent is processing a request, a subtle animation involving a mascot (e.g., a thinking mascot, or a spinning gear with a small mascot) can be displayed near the input area or as a temporary message.
    *   **Feedback/Notifications:** Mascots can be used in toast notifications or alerts within the chat panel (e.g., a happy mascot for successful operations, a concerned mascot for errors).
*   **Interactions:**
    *   Standard chat interactions: typing, sending, scrolling.
    *   Markdown support for user input and agent output.
    *   Clickable links and interactive elements within agent responses.

#### 5.2.2. RAG Agent Management (`RagAgentCreator.tsx` and related views)

*   **Layout (within a dedicated 'Agents' or 'My Agents' view, accessible from Left Sidebar):
    *   **Agent List:** A sortable, filterable list or grid of the user's created RAG agents. Each entry should display:
        *   Agent Name.
        *   A representative mascot icon/avatar for that agent (e.g., user can select from a predefined set of mascots or a default is assigned based on keywords in description, like `mascot_style_2.png` for a more 'technical' agent).
        *   Brief description.
        *   Status (e.g., active, training, error).
        *   Quick action buttons (e.g., Chat, Edit, Publish to Market, Delete).
    *   **Agent Creation/Editing Form:** A multi-step or tabbed interface for creating or modifying RAG agents.
        *   **Basic Info:** Name, description, selection of a primary mascot avatar for this agent.
        *   **System Prompt:** Text area for defining the agent's core instructions.
        *   **Knowledge Base Management:** Interface to create new KBs, link existing KBs, upload documents (PDF, TXT, MD), and monitor document processing status. Each document could have a small, generic document icon, but the overall KB could be associated with a 'knowledge' themed mascot variant.
        *   **Configuration:** Advanced settings (e.g., LLM model selection, retrieval parameters).
    *   **Agent Detail View:** When an agent is selected, this view (possibly in the Right Sidebar or a main panel) shows comprehensive details, including its mascot, full description, associated KBs, and performance metrics.
*   **Mascot Integration:**
    *   **Agent Identity:** Each RAG agent should be strongly associated with a chosen mascot, acting as its visual identifier across the platform (list, chat, market).
    *   **Guidance/Tutorials:** Helper mascots could appear in tooltips or short guides during agent creation, explaining different fields or steps.
    *   **Status Indicators:** Animated mascots could indicate the status of KB document processing (e.g., a busy mascot for 'processing', a happy one for 'completed').
*   **Interactions:**
    *   CRUD operations for agents and knowledge bases.
    *   Drag-and-drop for document uploads.
    *   Interactive forms and real-time validation.

#### 5.2.3. Agent Flow Builder (`FlowBuilderCanvas.tsx` and related views)

*   **Layout (within a dedicated 'Flow Builder' view):
    *   **Canvas:** The main area where users visually construct agent flows. Uses a node-based interface.
    *   **Node Palette:** A sidebar or panel containing available node types (e.g., Start, End, LLM Call, RAG Query, Code Execution, API Call, User Input, Conditional Logic). Each node type could have a distinct icon, potentially a highly simplified or abstract mascot element related to its function.
    *   **Properties Panel:** When a node is selected on the canvas, this panel (often in the Right Sidebar) displays its configurable properties.
    *   **Toolbar:** Contains actions like Save Flow, Run Flow, Debug Flow, Zoom In/Out, Auto-layout.
*   **Mascot Integration:**
    *   **Node Icons:** While full mascots might be too detailed for small nodes, abstract elements or color-coding derived from the mascot palette can be used for node types. For example, an 'LLM Call' node might use a brain-like icon with Amarax Purple.
    *   **Helper Mascot for Onboarding:** A friendly mascot could guide new users through their first flow creation with interactive tutorial overlays.
    *   **Execution/Debugging Feedback:** Animated mascots could indicate the flow's progress during execution (e.g., a mascot 'walking' along the connections) or highlight errors on specific nodes (a warning mascot appearing on a failed node).
*   **Interactions:**
    *   Drag-and-drop nodes from the palette to the canvas.
    *   Connect nodes by dragging edges between ports.
    *   Configure node properties in the properties panel.
    *   Pan and zoom the canvas.

#### 5.2.4. Agent Market (`AgentMarketplace.tsx` - conceptual name)

*   **Layout (within a dedicated 'Marketplace' view):
    *   **Browse/Search View:**
        *   Search bar with filtering options (categories, tags, price, rating).
        *   Grid or list display of available agents. Each agent card should prominently feature:
            *   The agent's chosen **mascot avatar** (e.g., `mascot_style_1.png` or a custom one uploaded by publisher).
            *   Agent Title.
            *   Publisher Name.
            *   Short Description.
            *   Rating (e.g., star system).
            *   Price (or "Free").
            *   Tags.
    *   **Agent Detail Page:** Accessed by clicking on an agent card.
        *   Larger display of the agent's mascot.
        *   Comprehensive description, features, version history.
        *   Publisher information (with link to publisher profile, potentially featuring their own chosen mascot/avatar).
        *   User reviews and ratings.
        *   Screenshots or demo (if applicable).
        *   "Acquire" or "Add to My Agents" button.
    *   **My Published Agents View:** For users to manage agents they have published to the market.
    *   **My Acquired Agents View:** For users to see agents they have obtained from the market.
*   **Mascot Integration:**
    *   **Primary Agent Visual:** The mascot is key to an agent's identity and appeal in the marketplace. High-quality, expressive mascots will be encouraged.
    *   **Category Icons:** Categories within the marketplace could use thematic mascot icons (e.g., a sales-bot mascot for the 'Sales' category).
    *   **Promotional Banners/Featured Agents:** Mascots can be used extensively in promotional graphics within the marketplace.
    *   **Rating/Review Visuals:** Small mascot expressions could accompany ratings (e.g., happy for 5 stars, neutral for 3 stars).
*   **Interactions:**
    *   Searching, filtering, sorting agents.
    *   Viewing agent details.
    *   Acquiring/purchasing agents.
    *   Leaving reviews and ratings.
    *   Publishing and managing owned agent listings.

This detailed UI/UX section will be further refined with more specific wireframes or mockups as development progresses, but this provides a solid foundation for incorporating the Thrae IDE style, Amarax branding, and the engaging use of mascots throughout the platform's core features.




*   **RagAgents:** (Refined - added `mascot_avatar_url`)
    *   `agent_id` (PK, INT, AUTO_INCREMENT)
    *   `user_id` (FK, INT, NOT NULL) - Owner of the agent
    *   `agent_name` (VARCHAR, NOT NULL)
    *   `description` (TEXT, NULLABLE)
    *   `system_prompt` (TEXT, NULLABLE)
    *   `knowledge_base_id` (FK, INT, NOT NULL) - Links to KnowledgeBases table
    *   `mascot_avatar_url` (VARCHAR, NULLABLE) - Path/URL to the agent's chosen mascot image (e.g., '/mascots/style1.png' or a full URL if user-uploaded/external)
    *   `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `is_public` (BOOLEAN, DEFAULT FALSE) - Indicates if user intends to share/sell, distinct from market listing status.

*   **AgentMarketListings (New - Refined - added `mascot_display_url` and `category`):
    *   `listing_id` (PK, INT, AUTO_INCREMENT)
    *   `agent_id` (FK, INT, NOT NULL) - The RAG Agent being listed
    *   `publisher_user_id` (FK, INT, NOT NULL) - The user who published the agent
    *   `title` (VARCHAR, NOT NULL)
    *   `market_description` (TEXT, NOT NULL)
    *   `category` (VARCHAR, NULLABLE) - e.g., "Productivity", "Sales", "Customer Support", "Fun"
    *   `tags` (JSON or TEXT - comma-separated tags for searchability)
    *   `price` (DECIMAL, NULLABLE - 0 for free, or a price)
    *   `version` (VARCHAR, NOT NULL - e.g., "1.0.0")
    *   `mascot_display_url` (VARCHAR, NULLABLE) - Path/URL to the mascot image specifically for market display. Defaults to RagAgent's mascot if not set.
    *   `status` (VARCHAR - e.g., "published", "unpublished", "under_review")
    *   `published_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
    *   `updated_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
    *   `average_rating` (FLOAT, NULLABLE)
    *   `download_count` (INT, DEFAULT 0)

(Other tables like Users, Subscriptions, KnowledgeBases, ChatHistories, ChatMessages, WebRoamingTasks, UserAcquiredAgents, AgentFlows, AgentFlowExecutions remain largely the same as previously defined, unless a specific UI/UX change implies a direct schema modification not yet caught. For now, the primary changes are for agent identity and market presentation.)
