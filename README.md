# ELSA Speak GenAI App Improvement Challenge Submission

## Introduction

This repository contains my submission for the ELSA Speak GenAI App Improvement Challenge. The goal is to evaluate the current GenAI functionalities of the ELSA Speak app and propose targeted improvements to enhance user experience. This submission includes folowing key tasks:

1. **Explore the App (Dogfooding)** – A summary of my hands-on experience with the app.
2. **Identify an Area for Improvement** – An analysis highlighting a significant improvement opportunity, along with the rationale for focusing on it.
3. **Proposal & Implementation Outline** – A technical outline with clean pseudocode and an architecture diagram.
4. **Roadmap & Next Steps** – A phased plan for further development.

---

## Task 1: Explore the App (Dogfooding)

### Objective
Gain hands-on experience with the app’s GenAI features such as onboarding, content generation, learning paths, and feedback.

### Process
I spent approximately 30 minutes using the ELSA Speak app to explore its key functionalities. I examined the onboarding process, personalized learning path generation, feedback mechanisms (for pronunciation, vocabulary, grammar, and conversation), user interface design, and technical performance.

### Summary of App Interaction
During my exploration, I found that the onboarding process was highly personalized and smooth—it collected essential details such as my name, email, mother tongue, English proficiency, learning goals, and areas of interest. This resulted in a tailored experience that included a customized accent selection and a learning path composed of 14 units focused on Data Science and Analytics.

The app’s GenAI features are robust. In pronunciation exercises, feedback is provided as a clear percentage score with visual cues (red for incorrect and green for correct letters). Vocabulary exercises offer detailed critiques and improvement suggestions, though the final overall feedback sometimes feels slightly out-of-context. Conversational exercises maintain a natural, engaging flow; however, grammar exercises occasionally suffered from connectivity issues (e.g., "reconnecting" messages), and I experienced a crash after a speaking session. The app’s gamification elements—badges, levels, and leaderboards—are well-integrated, although a few minor usability issues (like scrolling challenges) were observed on some screens.

---

## Task 2: Identify an Area for Improvement

### Observations
While the app excels in personalized onboarding and interactive exercises, one significant pain point is the lack of **Persistent Session Context & State Retention**. Currently, if a lesson is interrupted, the session is discarded and the lesson must be restarted from the beginning. There is no mechanism to preserve the session state or conversation context, which limits the continuity of adaptive feedback and impairs the overall user experience.

### Proposed Improvement: Persistent Session Context & State Retention

#### What It Involves
- **Context Preservation:**  
  Continuously capture and store all relevant session data—such as user inputs, AI responses, progress indicators, and intermediate feedback—throughout a lesson.
- **Seamless Resumption:**  
  Enable the system to restore an interrupted session by retrieving the stored data, allowing users to resume lessons without losing their progress or context.
- **Storage Options:**  
  Implement a hybrid approach using:
  - **Client-Side Storage:** Leverage technologies such as SQLite for immediate, low-latency storage.
  - **Server-Side Storage:** Use a session database or caching solution to securely store session data, ensuring scalability and cross-device accessibility.
- **Enhanced Feedback:**  
  With a preserved session context, the AI can generate more accurate, coherent, and context-aware feedback by analyzing the complete session rather than isolated fragments.

#### Benefits
- **Improved User Experience:**  
  Users can seamlessly continue their lessons, reducing the frustration caused by having to restart sessions after interruptions.
- **Enhanced Adaptive Learning:**  
  A continuous context allows the AI to better assess performance trends over the entire lesson, leading to more meaningful and personalized feedback.
- **Resilience:**  
  The app becomes more robust against interruptions due to network issues or accidental crashes, preserving valuable learning data.

---

## Task 3: Proposal & Implementation Outline

### 1. Motivation & Rationale Overview

**Current Limitation:**  
The current implementation of the ELSA Speak app, while delivering robust GenAI features, does not preserve the session context throughout a lesson. When a lesson is interrupted—whether due to an app crash, network issues, or accidental termination—the entire session is discarded and the user must restart from the beginning. This limitation disrupts the adaptive feedback process and breaks the continuity of the learning experience. Furthermore, based on industry best practices, the current system appears to lean towards a hybrid of fine-tuned, rule-based evaluation and selective context retrieval rather than a fully dynamic Retrieval-Augmented Generation (RAG) approach. However, regardless of the underlying model, the absence of persistent session state hampers the potential of generating truly context-aware feedback.

**Proposed Improvement:**  
To address this shortcoming, I propose implementing **Persistent Session Context & State Retention** across lessons. This enhancement would continuously capture and store all user interactions—such as completed exercises, AI responses, and intermediate feedback—allowing the application to restore the session seamlessly upon re-entry. By preserving the entire interaction context, the system can provide more coherent, adaptive, and personalized feedback that reflects the user’s progress throughout the lesson. This improvement not only mitigates the frustration associated with lost progress but also significantly augments the overall GenAI functionality by enabling more contextual analyses and tailored recommendations. Ultimately, incorporating persistent session management will bridge the gap between static, rule-based feedback and a truly dynamic, context-aware learning experience.

### 2. Technical Outline

#### 2.1 Design

**Overall Approach:**  
The solution is built around a persistent session manager that continuously captures user interactions and feedback during a lesson. The session data is stored in a local database on the device (using native storage such as Core Data/SQLite for iOS and SQLite/Room for Android) and, optionally, synchronized with a server for cross-device continuity.

**Architecture Diagram:**

 ┌────────────────────┐
 │  User Interactions │
 └─────────┬──────────┘
           │
           ▼
 ┌────────────────────┐
 │  Event Logging &   │
 │   State Capture    │
 └─────────┬──────────┘
           │
           ▼
 ┌─────────────────────────────────┐
 │ Persistent Session Storage      │
 │ (Local: SQLite,                 │
 │  Optional Server Sync via APIs) │
 └─────────┬───────────────────────┘
           │
           ▼
 ┌──────────────────────┐
 │  Session Restoration │
 │ & Context Recovery   │
 └──────────────────────┘

 **Key Design Decisions:**
- **Event-Driven Updates:** Each critical user action (e.g., starting an exercise, submitting a voice input, receiving AI feedback) generates an event that is appended to the current session state.
- **Periodic Persistence:** The session state is saved periodically (or upon key events) to minimize data loss.
- **Cross-Platform Compatibility:** Use native storage mechanisms for iOS and Android to ensure fast access and reliability. Optionally integrate server-side storage if multi-device support is required.

---

### 2.2 Data & Simulation

**Data Collection:**  
- **Real-Time Interaction Data:**  
  Capture details such as:
  - `user_id` and `lesson_id`
  - Timestamps for each event
  - Interaction type (e.g., pronunciation exercise, vocabulary quiz)
  - Feedback data (scores, visual indicators, text feedback)
  - Partial lesson progress (e.g., current unit/step)

- **Simulation (When Live Data Is Unavailable):**  
  Create a mock dataset in JSON format. For example:

```json
{
  "user_id": 123,
  "lesson_id": "lesson_xyz",
  "timestamp": "2025-04-15T10:15:00Z",
  "session_steps": [
    { "step": "pronunciation_exercise", "input": "Hello, world", "score": 78 },
    { "step": "vocabulary_exercise", "input": "I love data science", "feedback": "Improve sentence structure" }
  ]
}
```
### 2.3 Key Components

#### A. Data Collection

- **Event Listeners:**  
  Integrate event listeners within the app’s workflow to capture each user interaction.  
  - **Example events:**  
    - Lesson start  
    - Exercise completion  
    - Voice input received  
    - AI feedback generated

- **Local Storage Integration:**  
  - Use native APIs such as Core Data (iOS) or SQLite/Room (Android) to immediately store captured events.

#### B. Data Processing & Feedback

- **Session State Update:**  
  Create functions that append new interaction events to the current session state and trigger a save operation. These functions ensure that every interaction is logged and the session state is continuously updated for later restoration.


---

## Task 4: Roadmap & Next Steps

### Roadmap

**Phase 1 (2 Weeks):**  
• Develop local SQLite-based session storage for conversation logging.  
• Integrate session history into the AI prompt for context-aware tutoring.  
• Implement basic text-to-speech (TTS using gTTS) and speech-to-text (STT using Whisper or similar) for both text and audio inputs in a unified mobile interface.

**Phase 2 (3 Weeks):**  
• Add optional cloud synchronization to support cross-device session continuity.  
• Enhance AI prompt engineering to utilize deeper context and incorporate user error tracking.  
• Integrate a feedback loop to analyze user errors and adapt the lesson content dynamically.

**Phase 3 (4 Weeks):**  
• Refine the mobile UI for improved usability and a more engaging conversation experience.  
• Optimize TTS/STT performance to reduce latency, potentially using edge computing or local processing.  
• Conduct A/B testing to validate improvements in tutoring effectiveness and user satisfaction.

---

### Conclusion

This approach proposes a straightforward yet powerful solution for enhancing the ELSA Speak GenAI experience through **Persistent Session Context & State Retention**. By continuously storing conversation history, the app can resume interrupted sessions seamlessly and provide context-aware, adaptive feedback. The design is modular, scalable, and optimized for a mobile platform, significantly improving the overall user experience.
