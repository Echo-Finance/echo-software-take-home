# Senior Software Engineer - Take-Home Exercise: FireGo Wallet Service

Welcome to the Echo Senior Software Engineer take-home exercise! We're excited to see your skills in action.

## 🚀 About Echo

We are an early-stage company building innovative solutions in the digital asset space. We value autonomy, technical excellence, and a proactive approach to problem-solving.

## 🎯 The Challenge

Your task is to build a backend service in Go, named "FireGo Wallet Service," that interacts with the Fireblocks API to perform basic cryptocurrency wallet operations. This service should expose a set of RESTful API endpoints.

The goal is to assess your technical proficiency in Go, your understanding of cryptocurrency concepts (particularly around custody and transactions using services like Fireblocks), your ability to work with databases like PostgreSQL using GORM, and your communication skills. We're looking for engineers who can take a high-level task and independently drive it towards a production-quality solution.

### Core Functionality:

Your service must implement the following features:

1.  **Create Wallet:**
    * **Endpoint:** `POST /wallets`
    * **Request Body:** `{ "name": "My Personal Wallet" }` (or similar user-friendly name)
    * **Action:** Create a new Fireblocks Vault Account. Store the `vaultAccountId` returned by Fireblocks along with the user-friendly name in your PostgreSQL database.
    * **Response:** Details of the created wallet, including its local ID, name, and Fireblocks Vault ID.

2.  **Get Wallet Balance:**
    * **Endpoint:** `GET /wallets/{walletId}/assets/{assetId}/balance`
    * **Path Parameters:**
        * `walletId`: The local ID of the wallet stored in your database.
        * `assetId`: The Fireblocks asset ID (e.g., `TEST_BTC`, `TEST_ETH_TEST3` for Goerli ETH).
    * **Action:** Retrieve the balance for the specified `assetId` within the corresponding Fireblocks Vault Account.
    * **Response:** The asset balance details.

3.  **Get Deposit Address:**
    * **Endpoint:** `GET /wallets/{walletId}/assets/{assetId}/address`
    * **Path Parameters:** (Same as Get Wallet Balance)
    * **Action:** Retrieve a deposit address for the specified `assetId` within the corresponding Fireblocks Vault Account.
    * **Response:** The deposit address details.

4.  **Initiate Transfer:**
    * **Endpoint:** `POST /wallets/{walletId}/transactions`
    * **Path Parameters:** `walletId` (local ID of the source wallet).
    * **Request Body:**
        ```json
        {
          "assetId": "TEST_BTC", // Or other testnet asset
          "amount": "0.001",
          "destinationAddress": "some_external_testnet_address",
          "note": "Test transfer" // Optional
        }
        ```
    * **Action:** Create and submit a transaction using the Fireblocks API from the specified Vault Account. Store a record of this transaction attempt (including Fireblocks `txId`, amount, asset, destination, and initial status) in your database.
    * **Response:** Details of the initiated transaction, including the Fireblocks `txId`.

5.  **Get Transaction Status:**
    * **Endpoint:** `GET /transactions/{fireblocksTxId}`
    * **Path Parameters:** `fireblocksTxId`: The transaction ID returned by Fireblocks.
    * **Action:** Retrieve the status of the transaction from Fireblocks. Update the status of the transaction in your database.
    * **Response:** The transaction details, including its current status.

### ⭐ Bonus Functionality (Optional):

* **List Supported Assets:** Implement an endpoint `GET /fireblocks/assets` that fetches and returns a list of all supported assets from Fireblocks.
* **Webhook Handler:** Implement a webhook endpoint (`POST /fireblocks/webhook`) to receive transaction status updates from Fireblocks. This would require you to understand Fireblocks webhooks and securely process incoming notifications to update your database. *(This is more advanced and demonstrates a deeper understanding of production systems).*

## 🛠️ Technical Requirements

* **Language:** Go (latest stable version)
* **Database:** PostgreSQL
* **ORM:** GORM (for database interactions and migrations)
* **HTTP Framework:** Standard library (`net/http`) or a lightweight framework of your choice (e.g., Gin, Echo, Chi).
* **Fireblocks API:** You will be provided with a **testnet** API key and secret separately. **Do NOT commit these credentials to your repository.**
* **Code Management:** Git.
* **Testing:**
    * Unit tests for key business logic (e.g., service layer functions, complex transformations).
    * Integration tests for API endpoints are a plus.
* **Configuration:** Application configuration (like database connection strings, Fireblocks API details) should be managed via environment variables. Provide a `.env.example` file.
* **Containerization (Bonus):** Provide a `Dockerfile` and `docker-compose.yml` (if applicable) to build and run your application and its database.

## 🔥 Fireblocks API Interaction

* You will primarily interact with the Fireblocks API. Refer to the [Fireblocks API Documentation](https://developers.fireblocks.com/reference/introduction) for details on authentication, request signing, and specific endpoints.
* Key endpoints you'll likely use:
    * `POST /v1/vault/accounts`
    * `GET /v1/vault/accounts/{vaultAccountId}/{assetId}`
    * `GET /v1/vault/accounts/{vaultAccountId}/{assetId}/addresses_paginated` (or `POST .../addresses` to generate a new one if needed)
    * `POST /v1/transactions`
    * `GET /v1/transactions/{txId}`
    * `GET /v1/assets` (for bonus)
* **Security:** Ensure secure handling of the Fireblocks API key and secret. These should be configurable via environment variables and **never hardcoded or committed.** Implement request signing as required by Fireblocks.
* **Error Handling:** Implement robust error handling and logging for all interactions with the Fireblocks API.

## 💧 Obtaining Testnet Assets

To fully test the wallet service functionality, you'll need testnet cryptocurrency assets in your Fireblocks vault accounts. These can be obtained from public testnet faucets:

* **Testnet Bitcoin (TEST_BTC):** Use Bitcoin testnet faucets to obtain test BTC
* **Testnet Ethereum (TEST_ETH_TEST3):** Use Goerli or other Ethereum testnet faucets to obtain test ETH
* **Other Testnet Assets:** Various blockchain networks provide testnet faucets for their respective test tokens

**Important Notes:**
* Only use testnet assets for this exercise - never use real cryptocurrency
* Testnet assets have no monetary value and are meant solely for development and testing
* You'll need to generate deposit addresses using your wallet service and fund them via these faucets
* Some faucets may have rate limits or require social verification to prevent abuse

Once you have testnet assets in your vault accounts, you can fully test the balance retrieval and transfer functionality of your wallet service.

## 🗄️ Database Schema (Suggestion)

You should define your database schema using GORM migrations. Here's a suggested starting point:

* **`wallets` table:**
    * `id` (Primary Key, e.g., UUID or auto-incrementing int)
    * `name` (string, user-friendly name)
    * `fireblocks_vault_id` (string, unique, indexed)
    * `created_at` (timestamp)
    * `updated_at` (timestamp)

* **`transactions` table:**
    * `id` (Primary Key)
    * `wallet_id` (Foreign Key to `wallets` table)
    * `fireblocks_tx_id` (string, unique, indexed)
    * `asset_id` (string)
    * `amount` (string or numeric, ensure precision)
    * `destination_address` (string)
    * `status` (string, e.g., `SUBMITTED`, `PENDING_SIGNATURE`, `BROADCASTING`, `CONFIRMING`, `COMPLETED`, `FAILED`, `CANCELLED`)
    * `note` (string, optional)
    * `created_at` (timestamp)
    * `updated_at` (timestamp)

Feel free to adapt or extend this schema as you see fit.

## 📁 Suggested Project Structure

While you have flexibility, a typical Go project structure might look like this:


firego-wallet-service/
├── cmd/
│   └── api/
│       └── main.go         # Main application entry point
├── internal/
│   ├── api/                # HTTP handlers, routes, request/response structs, middleware
│   │   ├── handlers/
│   │   └── router.go
│   ├── config/             # Configuration loading (e.g., from .env)
│   │   └── config.go
│   ├── database/           # GORM models, migration logic, repository/DAL functions
│   │   ├── models/
│   │   └── repository.go
│   ├── fireblocks/         # Client/service for interacting with Fireblocks API
│   │   ├── client.go       # Low-level API client (signing, requests)
│   │   └── service.go      # Higher-level business logic for Fireblocks ops
│   └── core/               # Core domain logic/services (if needed, distinct from handlers)
│       └── wallet_service.go
│       └── transaction_service.go
├── migrations/             # SQL or GORM migration files
├── pkg/                    # Shared libraries (if any, less common for small projects)
├── test/                   # Test files, potentially mirroring internal structure
├── .env.example            # Example environment file
├── .gitignore
├── Dockerfile              # Optional
├── docker-compose.yml      # Optional
├── go.mod
├── go.sum
└── README.md               # This file!


##  deliverables

Please provide the following:

1.  **A link to a private GitHub repository** containing your complete solution.
2.  **A comprehensive `README.md` file** (this file, but updated by you) that includes:
    * A brief overview of your solution, any assumptions made, and key design choices.
    * Clear instructions on how to set up the development environment (dependencies, Go version, database setup, etc.).
    * Instructions on how to configure the application (especially Fireblocks API key/secret via environment variables).
    * Step-by-step instructions on how to build and run the application.
    * Instructions on how to run any tests you've written.
    * API documentation for the endpoints you've created. This can be a simple list with request/response examples, a Postman collection, or a Swagger/OpenAPI specification.
3.  **(Optional but highly recommended for communication assessment):** A short video (5-10 minutes, e.g., via Loom) where you:
    * Briefly walk through your code structure and key components.
    * Explain your design decisions and any trade-offs you made.
    * Demonstrate the application's functionality by making a few API calls.

## ⚖️ Evaluation Criteria

Your submission will be evaluated based on the following:

* **Correctness & Completeness:** Does the application meet the specified requirements? Is it fully functional?
* **Technical Proficiency (Go & System Design):**
    * Code quality, clarity, organization, and adherence to Go best practices.
    * Effective use of Go's concurrency features (if applicable and appropriate).
    * Sensible project structure and separation of concerns.
    * Robust error handling and logging.
* **Fireblocks Integration:**
    * Correct and secure usage of the Fireblocks API (authentication, request signing).
    * Understanding of relevant Fireblocks concepts (vaults, assets, transactions).
* **Database Interaction (Postgres & GORM):**
    * Effective use of GORM for schema definition (migrations) and data manipulation.
    * Sensible database schema design.
    * Efficient and correct database queries.
* **Testing:**
    * Quality and coverage of unit tests.
    * Presence and quality of integration tests (if included).
* **Problem Solving & Independence:**
    * Ability to understand the requirements and implement a working solution with minimal guidance.
    * Thoughtfulness in design and implementation choices.
* **Communication:**
    * Clarity and completeness of the `README.md` (setup, usage, API docs).
    * Quality of code comments and commit messages.
    * (If submitted) Clarity and effectiveness of the optional video walkthrough.
* **Production Readiness Considerations:**
    * Attention to aspects like configuration management, logging, security, and overall robustness.
    * (If submitted) Quality of Docker setup.

## 🤝 Collaboration & Support

We encourage open communication and collaboration throughout this exercise! You're not expected to work in isolation:

* **Dedicated Slack Channel:** You'll be invited to a private Slack channel where you can communicate directly with our team members during the exercise.
* **Synchronous Support:** Feel free to schedule calls with our team if you need real-time discussion about technical challenges, requirements clarification, or design decisions.
* **Asynchronous Communication:** We support various communication methods that work best for you:
    * **Slack:** For quick questions, updates, or informal discussions
    * **Loom Videos:** Share screen recordings to explain challenges or demonstrate progress
    * **GitHub:** Use issues, discussions, or commit comments for code-related questions
    * **Email:** For any formal communication needs

**When to Reach Out:**
* Clarifying requirements or technical specifications
* Discussing architectural decisions or trade-offs
* Getting unstuck on technical challenges
* Seeking feedback on your approach or progress
* Any questions about Fireblocks API integration

We value engineers who know when and how to collaborate effectively, so don't hesitate to engage with our team!

## ⏰ Submission Guidelines

* Please aim to complete this exercise within **[Specify timeframe, e.g., 3-5 days]**. We understand you have other commitments; this is a guideline, not a strict deadline. Focus on quality.
* Share your private GitHub repository with the following GitHub username(s): **[Your GitHub Username(s) for Review]**.
* Submit your solution by **[Date and Time, Timezone, if you have a hard deadline]**.
* **Important:** Ensure your Fireblocks API key and secret are **NOT** committed to the repository. Your `README.md` should explain how to configure these (e.g., via environment variables in a `.env` file that is gitignored).

## ❓ Questions

If you have any questions about the assignment, please don't hesitate to reach out to us via Slack. Feel free to ask any clarifying questions by 24-48 hours after receiving the test to ensure you have enough time to work on the solution.

---

Good luck! We look forward to seeing your solution.
