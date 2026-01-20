# Telegra Commerce Architecture

## Overview

The Telegra Commerce platform is a multi-layered, cloud-based e-commerce solution designed to support affiliates and healthcare providers. The architecture consists of three primary layers: Management, Infrastructure, and Commerce, with seamless integration between provisioning, store management, and customer-facing applications.

## Architecture Diagram

```mermaid
architecture-beta
    group mgt(cloud)[Management]
    group infra(cloud)[Infrastructure]
    group shop(cloud)[Commerce]
        
    service tam(server)[TAM] in mgt
    service portal(internet)[Affiliate Admin Portal] in mgt
            
    service docker(mdi:docker)[Docker] in infra
    service prov(mdi:cog)[Provisioning] in infra
            
    service wp(disk)[WP Store] in shop
    service tcs(server)[TCS] in shop
    service sf(internet)[Shop Frontend] in shop
        
    %% Infrastructure Path (straight lines)
    tam:R -- L:portal
    portal:R -- L:prov
    prov:R -- L:docker
    docker:R -- L:wp
        
    %% Data & Journey Logic (avoid Docker overlap)
    wp:R -- L:tcs
    portal:L -- L:tcs
        
    %% Final Journey Connection
    portal:R -- L:sf
```

## System Flow - Sequence Diagram

This diagram illustrates the complete flow from admin provisioning through customer journey implementation:

```mermaid
sequenceDiagram
    autonumber
    participant Admin as Telegra Admin Manager (TAM)
    participant Portal as Affiliate Admin Portal (AA)
    participant Prov as Provisioning Service
    participant Docker as Docker Runtime
    participant WP as WordPress Store
    participant TCS as Telegra Commerce Server
    participant SF as Shop Frontend

    Note over Admin, Portal: Step 1: Provisioning
    Admin->>Portal: API Call
    Portal->>Prov: Provision Affiliate
    Prov->>Docker: Run Dockerfile
    Docker->>WP: Create Store instance
    
    Note over Prov, TCS: Step 2: Registration & Sync
    Prov->>TCS: Register Affiliate
    TCS->>WP: REST API Call (JSON)
    WP-->>TCS: Return Products & Metadata

    Note over Portal, SF: Step 3: Customer Journey
    Portal->>SF: Define Journeys & Shop URL
    SF-->>Portal: Redirect after Journey
```

## Component Interactions - Flowchart

Detailed view of the relationships between all system components:

```mermaid
flowchart TB
    %% Group Definitions
    subgraph admin [Admin Layer]
        TAM(Telegra Admin Manager)
        AA(Affiliate Admin Portal)
    end
    
    subgraph platform [Telegra Platform]
        PROV(Provisioning Service)
        TCS(Telegra Commerce Server)
        SF(Shop Frontend)
    end
    
    subgraph infra [Infrastructure]
        DOCKER(Docker Runtime)
        WP(WordPress Store)
    end
    
    %% Relationships
    TAM -->|API| AA
    AA -->|Provision Affiliate| PROV
    PROV -->|Run Dockerfile| DOCKER
    DOCKER -->|Create Store| WP
    
    PROV -->|Register Affiliate| TCS
    TCS -->|REST APIs| WP
    WP -->|Products, Collections, Metadata| TCS
    
    AA -->|Define Journeys| SF
    AA -->|Shop URL| SF
    SF -->|Redirect after Journey| AA
    
    %% Styling for clarity
    style admin fill:#f9f,stroke:#333,stroke-width:2px
    style platform fill:#bbf,stroke:#333,stroke-width:2px
    style infra fill:#dfd,stroke:#333,stroke-width:2px
```

## Core Components

### Management Layer

- **Telegra Admin Manager (TAM)**: Central administration interface for managing the entire platform
- **Affiliate Admin Portal (AA)**: Portal for affiliate partners to manage their stores, products, and customer journeys

### Infrastructure Layer

- **Provisioning Service**: Orchestrates the creation and configuration of new affiliate stores
- **Docker Runtime**: Container orchestration platform for isolated store instances

### Commerce Layer

- **WordPress Store (WP)**: E-commerce backend managing products, collections, inventory, and metadata
- **Telegra Commerce Server (TCS)**: Core REST API backend handling business logic, orders, payments, and integrations
- **Shop Frontend (SF)**: Customer-facing React application for browsing and purchasing products

## Key Integration Points

### 1. Provisioning Workflow

- Admin initiates affiliate provisioning through the TAM
- Portal submits provisioning request to the Provisioning Service
- Provisioning Service spawns isolated Docker containers running WordPress instances
- Each store instance is registered with the TCS

### 2. Data Synchronization

- TCS maintains synchronization with WP stores via REST APIs
- Product catalogs, collections, and metadata are bidirectionally synced
- Order and transaction data flows from TCS to WP for inventory management

### 3. Customer Journey Management

- Affiliates define customer journeys and funnels through the Admin Portal
- Journey configurations are deployed to the Shop Frontend
- Customer actions trigger data collection and redirects back to the Portal

## Data Flow

1. **Store Creation**: TAM → Portal → Provisioning → Docker → WP Store
2. **Affiliate Registration**: Provisioning → TCS
3. **Product Sync**: WP Store ↔ TCS (bidirectional REST APIs)
4. **Journey Definition**: Portal → Shop Frontend
5. **Customer Interaction**: Customer → Shop Frontend → Portal

## Technology Stack

- **Backend**: Node.js, Express.js, MongoDB
- **Frontend**: React, Next.js
- **Infrastructure**: Docker, Docker Compose, Nginx
- **Databases**: MongoDB (primary data store)
- **APIs**: RESTful architecture with JSON payloads
- **Provisioning**: Automated container orchestration

## Deployment Architecture

The platform supports multi-tenant deployment where:

- Each affiliate gets an isolated WordPress store instance via Docker
- All stores connect to a centralized TCS for core business logic
- Admin Portal and Shop Frontend are shared multi-tenant applications
- Data isolation is maintained at both container and database levels

---

Previous: [Installing and Running](installing-and-running.md)

Next: [Command Line Interface](cli.md)
