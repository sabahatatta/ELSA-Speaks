## High-Level Pseudocode Outline

 BEGIN APPLICATION
  -  Load API keys and configuration from .env
  -  Initialize persistent session storage (e.g., using Streamlit’s session_state or an equivalent mechanism)
  -  Display a lesson-specific welcome message (via text and TTS audio)
  -  Provide unified UI options for input:
      - Text input field
      - Audio input (record or upload, then use Whisper for transcription)
  -  For each user interaction:
      - Retrieve conversation history from the session manager
      - Append the new user message (either typed or transcribed from audio) to the history
      - Compile the full context (previous history plus current input) for the AI engine
      - Call the AI engine with the compiled context to generate a response
      - Append the AI response (including generated TTS audio) to the session’s conversation history
      - Update the UI display with the latest conversation snippet
  -  Loop continuously until the session ends or the user exits
  -  Persist the session state as necessary
 END APPLICATION

This outline captures the core flow:

 -  Initialization: Load configurations and prepare the persistent session state.

 -  User Interaction: Accept input via text and audio.

 -  Context Management: Maintain and compile complete conversation history.

 -   AI Response: Invoke the AI engine to generate context-aware responses.

 -  Feedback Loop: Update the session and present responses (both as text and via TTS)