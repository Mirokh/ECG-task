# Generative AI System for Personalized ECG Analysis & Reports

## 1. Overview
This system processes **ECG reports in PDF/image format**, extracts relevant medical information using **OCR**, and generates **personalized AI-driven interpretations** via **NLP**. It follows a **microservices architecture**, enabling **scalability, flexibility, and real-time processing** with Kafka event streaming.

## 2. Technology Stack
- **Frontend**: Vue.js, Quasar, Pinia, Socket.io
- **Backend**: NestJS (Microservices Architecture)
- **Database**: MongoDB
- **Message Broker**: Apache Kafka
- **Storage**: Google Cloud Storage
- **Authentication**: JWT with **HTTP-only cookies**
- **AI Processing**: OpenAI GPT-4o for NLP, Google Vision API & pdf-parse for OCR

---

## 3. System Architecture
The system consists of multiple microservices communicating via **Kafka**:

### **Frontend**
- Built with **Vue.js & Quasar**, state-managed with **Pinia**.
- Uses **Socket.io** to receive live updates for processing events:
  - `ecg.updated` (for each pipline)


- **Implemented Features**:
  - ✅ User Authentication: **Login, Register, Refresh Token**
  - ✅ File Upload to Backend
  - ✅ Real-time Notifications

### **Backend Microservices (NestJS-based)**

#### **1. Orchestrator Microservice**
- Manages the **workflow** for ECG processing.
- **Publishes** `ecg.uploaded` event after storing the file in **Google Cloud Storage**.
- **Consumes** all Kafka messages (`ecg.uploaded`, `ecg.extracted`, `ecg.interpreted`).
- **Notifies frontend via WebSocket** once processing is completed.

#### **2. Authentication Microservice**
- **Centralized authentication** service.
- Handles **Login, Registration, Token Verification**.
- Uses **MongoDB** for user storage.
- Stores **refresh tokens in HTTP-only cookies** for security.
- Orchestrator **interacts via Auth Proxy** for authentication.

#### **3. OCR Microservice**
- **Consumes `ecg.uploaded`** event.
- Downloads the **ECG file (PDF/Image)** from Google Cloud Storage.
- Uses **pdf-parse** for **PDF text extraction** (fallback to **Google Vision API** for images).
- **Produces `ecg.extracted`** event with extracted text.

#### **4. NLP Microservice**
- **Consumes `ecg.extracted`** event.
- Uses **OpenAI GPT-4o** for ECG text interpretation and generates structured insights.
- Produces `ecg.interpreted` message, consumed by **Orchestrator**.
- Explores **alternative LLMs like Google Med-PaLM** (restricted access).

---

## 4. Event-Driven Architecture (Kafka Topics & WebSocket Events)

### **Kafka Topics**
```typescript
export const KAFKA_TOPIC_ECG_UPLOADED = 'ecg.uploaded';
export const KAFKA_TOPIC_ECG_EXTRACTED = 'ecg.extracted';
export const KAFKA_TOPIC_ECG_INTERPRETED = 'ecg.interpreted';
export const KAFKA_TOPIC_ECG_REPORTED = 'ecg.reported';
```

### **WebSocket Events**
```typescript
export const WEBSOCKET_EVENT_ECG_UPDATED = 'ecg.updated';
```

---

## 5. API Endpoints (For Frontend Integration)

### **Authentication APIs**
| Method | Endpoint | Request Body | Description |
|--------|----------------------|----------------------------------------|------------------|
| POST | `/v1/auth/login` | `{ email, password }` | Login user |
| POST | `/v1/auth/register` | `{ fullName, email, password }` | Register new user |
| POST | `/v1/auth/refresh` | `{}` | Refresh access token |
| POST | `/v1/auth/logout` | `{}` | Logout user |

### **ECG Processing API**
| Method | Endpoint | Request Body | Description |
|--------|----------------------|----------------------------------------|------------------|
| POST | `/v1/ecg/upload` | FormData (ECG File) | Upload ECG file |

---

## 6. Future Improvements (Pending Due to Deadline Constraints)
Although the system is functional, the following enhancements are planned:

- ✅ **Implement Unit Tests** to ensure reliability and correctness.
- ✅ **Improve Code Quality** (follow **TypeScript best practices & linting rules**).
- ✅ **Security Testing** for vulnerabilities in authentication & data handling.
- ✅ **Enhance AI Processing** (optimize **OCR & NLP models**).
- ✅ **Improve ECG Report Generation** (generate structured, user-friendly reports).

---

## 7. OCR & NLP Research Findings
During development, different approaches were explored:

### **OCR Options**
- 📌 **pdf-parse** → **Lightweight & free**, effective for PDFs.
- 📌 **Google Vision API** → More accurate for **image-based ECGs**.
- 📌 **Alternative**: OpenCV (free but complex).

### **NLP Model Choices**
- 📌 **OpenAI GPT-4o** → Best performance, but lacks **medical-specific fine-tuning**.
- 📌 **Google Med-PaLM** → More **medically accurate**, but restricted for public use.

🔹 **Optimal Approach**: **Hybrid OCR & Multiple LLMs**
- Combine **pdf-parse & Google Vision** for OCR.
- Use **GPT-4o for now**, but explore **fine-tuned medical models** in the future.

---

## 8. Deployment Plan on Google Cloud

### **Steps for Deployment**
1. **Frontend Deployment (Vue.js + Quasar)**
   - Host on **Google Cloud Run** or **Firebase Hosting**.
   - Configure **CORS & WebSockets** for real-time updates.

2. **Backend Deployment (NestJS Microservices)**
   - Deploy each microservice as **Google Cloud Run services**.
   - Set up **Google Cloud Pub/Sub or Kafka for event streaming**.
   - Configure **Google Cloud Storage for ECG files**.

3. **Database Deployment (MongoDB Atlas)**
   - Use **MongoDB Atlas** (Managed DB with backups & scaling).
   - Set up **authentication & access control**.

4. **Authentication & Security**
   - Use **Google IAM roles & policies**.
   - Secure **JWT tokens & API keys** in **Google Secret Manager**.

---

## 9. Conclusion
This project showcases **a scalable, event-driven AI system** for **automated ECG interpretation**. The microservices architecture ensures **modularity**, while Kafka and WebSockets enable **real-time processing**. Further improvements in **unit testing, security, and AI accuracy** will enhance its **robustness and usability**.
