# Cataseek — Architecture & Engineering Case Study

> **Commercial Disclaimer:** Cataseek is a proprietary B2B commercial SaaS product engineered at **Gofenice Technologies**. This repository serves as an independent **System Architecture, Engineering Case Study, and Technical Documentation** authored by **Anoop Muhammed** (Lead Developer). No proprietary source code, credentials, or private API keys are shared.

[![Live Platform](https://img.shields.io/badge/Live_Platform-cataseek.com-00C853?style=flat-square&logo=googlechrome&logoColor=white)](https://cataseek.com)
[![Console](https://img.shields.io/badge/Merchant_Console-console.cataseek.com-blue?style=flat-square)](https://console.cataseek.com)
[![Tech Stack](https://img.shields.io/badge/Stack-Node.js_•_Express_•_Meilisearch_•_MariaDB_•_React-orange?style=flat-square)](#technology-stack)

---

## 📌 Executive Overview

**Cataseek** is a hosted, multi-tenant eCommerce search service designed for small to mid-sized online stores running on **PrestaShop** and **WooCommerce**. 

Traditional eCommerce search engines query MySQL databases directly with strict `LIKE '%query%'` queries, resulting in high server load, zero typo tolerance, and lost sales when shoppers misspell keywords. Cataseek solves this by offloading search traffic from the merchant's server to a dedicated, high-speed indexing engine that delivers sub-25ms typo-tolerant search results directly in the shopper's browser.

---

## 🏗️ System Architecture & Data Flow

The platform is designed around a single-process TypeScript monolith running under **PM2**, managing API endpoints, background sweeps, in-memory search metering buffers, and database connections.

```mermaid
flowchart TD
    subgraph ClientLayer ["1. Client & Storefront Layer"]
        A[Shopper Browser + Widget]
        B[Merchant Storefront - PrestaShop / WooCommerce]
        C[Merchant Console SPA - React]
    end

    subgraph EdgeLayer ["2. Edge & Ingress Layer"]
        D[Cloudflare Proxy - TLS, DNS, CF-IPCountry]
        E[OpenLiteSpeed / CyberPanel - Reverse Proxy :3000]
    end

    subgraph AppLayer ["3. Application Backend (Node.js / Express)"]
        F["Express REST API (/api)"]
        G["Auth Guard (JWT, API Key, Domain)"]
        H["Smart Search Metering Engine (In-Memory Buffer, 5s Flush)"]
        I["Subscription Lifecycle & Razorpay Webhook Handler"]
    end

    subgraph DataLayer ["4. Data & Search Storage"]
        J[("MariaDB (System of Record: Accounts, Billing, Products)")]
        K[("Meilisearch (Read Copy: store_{id}_products Index)")]
    end

    A -- "Public Search (150ms debounce)" --> D
    B -- "Catalog Sync (50 batch)" --> D
    C -- "JWT Session Requests" --> D
    D --> E --> F
    F --> G
    G --> H
    F --> J
    F --> K
