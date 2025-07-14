# Contributions

This document outlines my **technical** and **non-technical** contributions to the **PlanTopia graduation project**. From backend development and cloud integration to machine learning deployment and team collaboration, I played a key role in ensuring a robust and scalable solution.

---

## 📚 Table of Contents

- [Technical Contributions](#technical-contributions)
  - [Deployment & Hosting](#deployment--hosting)
  - [Backend System (ASP.NET Core)](#backend-system-aspnet-core)
    - [Authentication & Authorization](#authentication--authorization)
    - [User Operations](#user-operations)
    - [Favorites Feature](#favorites-feature)
    - [Ordering System](#ordering-system)
    - [AWS Integration](#aws-integration)
  - [Machine Learning Integration](#machine-learning-integration)
    - [Dataset & Model Selection](#dataset--model-selection)
    - [Model Fine-Tuning & Hosting](#model-fine-tuning--hosting)
    - [Backend Integration](#backend-integration)
  - [Project Structure & Design Patterns](#project-structure--design-patterns)
- [Non-Technical Contributions](#non-technical-contributions)

---


##  Technical Contributions

###  Deployment & Hosting

- Responsible for **deploying and hosting** the entire web application using **Monster ASP.NET**.
  - Chose Monster ASP.NET as a reliable and free hosting option suitable for our needs as students.
  - It offered the flexibility and resources required to deploy a real-world ASP.NET Core project without incurring additional costs.
- Handled **remote database deployment** and configuration for cloud access.

---

###  Backend System (ASP.NET Core)

####  Authentication & Authorization

- Fully implemented **authentication and authorization** using **JWT tokens**.
- Supported role-based access control across the API for Admins, Workers, and Customers.

####  User Operations

- Developed endpoints for:
  - **User registration**
  - **Profile update**
  - **User deletion** (including role-specific logic)
- Used role resolution helper methods to correctly access sub-user tables (Admin, Worker, etc.)

####  Favorites Feature

- Implemented the **Favorites system**:
  - Linked customers with their favorite plants via a many-to-many relationship.
  - Handled adding, removing, and retrieving favorite items.

####  Ordering System

- Developed the full **Cart service**:
  - Add/view items with quantity and subtotal logic.
  - Created helper methods for user validation and cart initialization.
- Built the **Order service**:
  - Place, retrieve, and cancel orders.
  - Automatically move items from cart to orders and clear the cart after checkout.
  - Handled cancellation logic (only allowed for pending orders).

####  AWS Integration

- Integrated **AWS S3** for uploading and retrieving **user profile and plant images**.
- Handled all related code logic to ensure robust error handling and path consistency.

---

###  Machine Learning Integration

####  Dataset & Model Selection

- Conducted extensive research to select a suitable base model for plant disease classification.
  - Initially explored [this Hugging Face model](https://huggingface.co/linkanjarad/mobilenet_v2_1.0_224-plant-disease-identification), which is based on a **CNN architecture** and already trained on real-world images of plant diseases.
    - We chose it as a starting point due to its lightweight design (**MobileNetV2**) and good performance on limited datasets.
    - However, it only supported **38 disease classes**, which was insufficient for our project’s broader scope.
  
- To expand the model's classification capability, we searched for a larger, more comprehensive dataset and found a suitable one on Kaggle:
  -  [Plant Disease Classification - Merged Dataset](https://www.kaggle.com/datasets/alinedobrovsky/plant-disease-classification-merged-dataset/data)
    - This dataset contains approximately **90 disease classes**, covering a wider variety of plant species.
    - It offered sufficient diversity and volume to fine-tune the base model and better align with the real-world needs of our system.

####  Model Fine-Tuning & Hosting

- Fine-tuned the **MobileNetV2 model** using the selected Kaggle dataset to significantly expand its classification capabilities.
  - The fine-tuning process was time-consuming and required multiple iterations, but we successfully completed it — *Alhamdulillah*.
  - We used **Lightning AI** to train the model, as it offers strong performance, efficient training pipelines, and provides **free GPU access for students**, which was ideal for our needs.

- After training, we deployed the updated model to **Hugging Face Spaces**, creating a user-friendly interface where users can upload plant images and receive real-time predictions.
  -  [PlanTopia Disease Detector on Hugging Face](https://huggingface.co/spaces/reyad7/PlanTopia-diseases-detector)

#### Backend Integration

- Developed a dedicated **.NET API endpoint** to integrate the deployed ML model into our backend.
  - The endpoint accepts **image uploads from users** (e.g., photos of plants).
  - It forwards the image to the **Hugging Face Space** hosting the fine-tuned model.
  - The model processes the image and returns the predicted **plant disease name**.
  - The result is then sent back through the API and displayed to the user within the application.

- Ensured robust error handling and response formatting to provide a seamless experience between the backend and ML service.

---

###  Project Structure & Design Patterns

To ensure **scalability**, **maintainability**, and **testability**, our backend was designed using a **clean, layered architecture**. We adopted a **union architecture** that combines the **Service-Repository pattern**, enforcing a clear **separation of concerns** and keeping the codebase modular and robust.

- The `API` layer contains only **interaction points** (Controllers), responsible for handling HTTP requests and responses.
- The `Services` layer holds the **business logic** and acts as a bridge between controllers and the data layer.
- The `Repository` layer abstracts direct database access, with:
  - A **Generic Repository** to handle common CRUD operations.
  - **Specific repositories** to manage domain-specific queries.
- All repositories are coordinated using the **Unit of Work pattern**, ensuring atomic commits and better transaction management.

This structure promotes:
- **Code reuse** through generic components.
- **Ease of testing** by isolating logic layers.
- **Flexibility** for future scaling or integration needs.


####  Solution Overview

```
PlanTopia Solution
│
├── Api                                # Entry point of the application
│   ├── Controllers                    # HTTP endpoints to handle requests
│   ├── Dtos                           # Data Transfer Objects for request/response shaping
│   ├── Migrations                     # EF Core migrations history
│   ├── Services                       # Business logic, separated by domain
│   │   ├── AdminServices
│   │   ├── AuthService
│   │   ├── AwsS3Service
│   │   ├── CartService
│   │   ├── CustomerServices
│   │   ├── FavoriteItemService
│   │   ├── InvoiceService
│   │   ├── MLModelService
│   │   ├── OrderService
│   │   ├── PaymentService
│   │   ├── PlantService
│   │   ├── TaskService
│   │   └── WorkerServices
│   ├── appsettings.json              # App configuration (connection strings, tokens, etc.)
│   ├── Plantopia.http                # HTTP request collections (for testing APIs)
│   └── Program.cs                    # Main startup class
│
├── DAL                                # Data Access Layer
│   ├── Configuration                  # Entity configurations (Fluent API)
│   ├── Data                           # DbContext and seed logic
│   └── Repository                     # Repository pattern implementation
│       ├── Interfaces                 # Repository contracts
│       └── Implementations           # Repository logic + UnitOfWork
│
├── Entities                           # Domain layer for models and shared types
│   ├── Enums                          # Enum types (e.g., Roles, Statuses)
│   ├── Interfaces                     # Shared logic or marker interfaces
│   └── Models                         # Core data models (User, Order, Plant, etc.)



```
####  Repository & Unit of Work Patterns

```
DAL
│
└── Repository
├── Interfaces
│ ├── IGenericRepository.cs
│ ├── IOrderRepository.cs
│ └── IUnitOfWork.cs
└── Implementations
├── GenericRepository.cs
├── OrderRepository.cs
└── UnitOfWork.cs

```

- **GenericRepository<T>** handles reusable CRUD and LINQ logic.
- **UnitOfWork** coordinates all repositories and commits changes transactionally.

---

##  Non-Technical Contributions

### 1. From Real-World Needs to a Viable Project Idea

The journey began with a simple but insightful conversation. One of our team members, Abdelrahman Amer, had real-world experience working in a nursery. We quickly realized that this space was full of inefficiencies and unaddressed needs — from poor task assignment to the absence of disease monitoring tools. After several discussions, we concluded that building a nursery management system with intelligent features would be not only practical but also meaningful. That was the seed of what became **PlanTopia**.

### 2. Leading the Backend Team

I took the lead role in the **backend development team**, ensuring our work was well-organized and aligned with the overall system architecture. My responsibility wasn’t limited to dividing tasks — it included:
- Assigning roles based on each team member’s strengths and interests.
- Monitoring progress to ensure deadlines and technical quality were met.
- Explaining tasks in detail so everyone understood *why* and *how* a component fits into the larger system.
  
My goal was to foster clarity, ownership, and collaboration — not just delegate.

### 3. Supporting Team Members When It Mattered

No project goes perfectly — and PlanTopia was no exception. There were times when some teammates couldn’t complete their tasks due to confusion or lack of time. When this happened, I stepped in to either:
- Help them debug the issue and guide them step by step.
- Or take ownership of the task myself, ensuring we didn’t fall behind.

This approach ensured continuity, avoided delays, and allowed everyone to learn and grow.

### 4. Maintaining Team Energy & Momentum

I often played the role of the reminding the team of our goals and progress whenever things slowed down. Whether through checking in on our WhatsApp Group or speak to each one privately 

### 5. Coordinating Across Technical Tracks

As we had multiple technical areas — backend, machine learning, and frontend — I often found myself bridging the gap. I worked closely with both the ML team and the BackEnd team to:
- Align backend endpoints with what frontend and ML expected.
- Provide the needed data formats and payloads.
- Quickly respond to integration bugs or edge cases.

This ensured smoother handoffs and faster development cycles.

### 6. Learning & Growing Together

I encouraged **knowledge sharing** within the team by sharing tutorials,snippest codes examples or even search for info together whenever someone needed help. I also made sure to:
- Learn from teammates in areas where they were stronger.
- Stay supportive and collaborative throughout the journey.
- Focus on learning by doing — even when mistakes happened.


This non-technical side of the journey was just as important as the code. It taught me leadership, communication, and problem-solving skills that I’ll carry long after graduation.
