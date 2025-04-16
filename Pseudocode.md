# Pseudocode 

BEGIN APPLICATION

# CONFIGURATION & INITIALIZATION 
FUNCTION initialize_app():
    # Load environment variables (API keys, endpoints)
    config = load_config(".env")  # e.g., OPENAI_API_KEY, CLOUD_SYNC_URL
    
    # Initialize databases
    initialize_database()
    
    # Set up UI components
    setup_ui(
        lesson_selection_screen = True,
        conversation_screen = True,
        input_controls = ["text", "microphone"]
    )
    
    # Initialize audio services
    tts_engine = init_tts_engine(config.TTS_PROVIDER)
    stt_engine = init_stt_engine(config.STT_PROVIDER)
END FUNCTION


# DATABASE SETUP  
FUNCTION initialize_database():
    # Local SQLite for conversations
    EXECUTE SQL:
        CREATE TABLE IF NOT EXISTS conversations (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            session_id TEXT NOT NULL,
            timestamp TEXT DEFAULT CURRENT_TIMESTAMP,
            speaker TEXT CHECK(speaker IN ('User', 'Tutor')),
            message TEXT,
            audio_path TEXT
        )
        
    # Table for tracking user errors
    EXECUTE SQL:
        CREATE TABLE IF NOT EXISTS user_errors (
            user_id TEXT,
            error_type TEXT,
            count INTEGER DEFAULT 0,
            PRIMARY KEY (user_id, error_type)
        )
END FUNCTION


# SESSION MANAGEMENT  
FUNCTION on_lesson_selection(selected_lesson):
    session_id = generate_uuid()  # e.g., "sess_1234abcd"
    
    # Retrieve and play lesson-specific greeting
    greetings = get_greetings(lesson=selected_lesson)
    greeting = select_random(greetings)
    greeting_audio = TTS(greeting)
    
    # Log greeting to database
    add_message(
        session_id = session_id,
        speaker = "Tutor",
        message = greeting,
        audio_path = greeting_audio
    )
    
    render_conversation()
END FUNCTION


# CORE AI & AUDIO FUNCTIONS  
FUNCTION generate_ai_response(user_message, session_id):
    # Retrieve full conversation history
    history = query_db(
        "SELECT speaker, message FROM conversations WHERE session_id = ? 
         ORDER BY timestamp",
        params = (session_id,)
    )
    
    # Build context-aware prompt
    prompt = f"You are a {current_lesson} tutor. Focus on these errors: {get_user_errors()}\n"
    FOR entry IN history:
        prompt += f"{entry['speaker']}: {entry['message']}\n"
    prompt += f"User: {user_message}\nTutor:"
    
    # Generate response
    response = call_ai_api(prompt, model="gpt-4-turbo")
    response_audio = TTS(response)
    
    # Log response
    add_message(session_id, "Tutor", response, response_audio)
    RETURN response
END FUNCTION

FUNCTION TTS(text):
    audio_file = f"tts/{generate_uuid()}.mp3"
    tts_engine.convert(text, output=audio_file)
    RETURN audio_file
END FUNCTION

FUNCTION STT(audio_file):
    RETURN stt_engine.transcribe(audio_file).text
END FUNCTION


# FEEDBACK LOOP & ADAPTATION  
FUNCTION update_feedback_loop(session_id, user_id):
    # Analyze conversation for errors
    history = query_db("SELECT message FROM conversations 
                       WHERE session_id = ? AND speaker = 'User'", 
                       params=(session_id,))
    
    common_errors = []
    FOR message IN history:
        errors = grammar_check(message) + pronunciation_errors(message)
        common_errors.extend(errors)
    
    # Update error database
    FOR error IN common_errors:
        EXECUTE SQL:
            INSERT INTO user_errors (user_id, error_type, count)
            VALUES (?, ?, 1)
            ON CONFLICT(user_id, error_type) DO UPDATE
            SET count = count + 1
        PARAMS (user_id, error)
END FUNCTION


# PERSISTENCE & SYNC  
FUNCTION add_message(session_id, speaker, message, audio_path=None):
    # In-memory log
    conversation_log.append({
        "session_id": session_id,
        "timestamp": now(),
        "speaker": speaker,
        "message": message,
        "audio_path": audio_path
    })
    
    # Database write with retries
    attempts = 0
    WHILE attempts < 3:
        TRY:
            EXECUTE SQL:
                INSERT INTO conversations 
                (session_id, speaker, message, audio_path)
                VALUES (?, ?, ?, ?)
            PARAMS (session_id, speaker, message, audio_path)
            BREAK
        EXCEPT SQLiteError as e:
            log_error(f"DB Write Failed: {e}")
            attempts += 1
            sleep(0.5)
END FUNCTION

FUNCTION sync_to_cloud(session_id):
    local_data = query_db("SELECT * FROM conversations WHERE session_id = ?",
                         params=(session_id,))
    response = http_post(
        url = config.CLOUD_SYNC_URL,
        body = {
            "session_id": session_id,
            "data": local_data
        }
    )
    IF response.status != 200:
        queue_for_retry(local_data)
END FUNCTION


# DATA SIMULATION (TESTING)   
FUNCTION simulate_session(user_id="test_user"):
    session_id = generate_uuid()
    
    # Mock lesson setup
    on_lesson_selection("Business English")
    
    # Simulate user interactions
    test_inputs = [
        ("audio", "audio/welcome.mp3"),
        ("text", "I want to discuss contracts"),
    ]
    
    FOR mode, content IN test_inputs:
        IF mode == "audio":
            user_message = STT(content)
        ELSE:
            user_message = content
        
        process_user_input(
            session_id = session_id,
            user_id = user_id,
            message = user_message
        )
END FUNCTION


# MAIN APPLICATION FLOW   
initialize_app()

IF development_mode:
    simulate_session()
ELSE:
    session_id = initialize_session()
    selected_lesson = wait_for_lesson_selection()
    on_lesson_selection(selected_lesson)

    WHILE True:
        input_mode = get_input_mode()
        user_message = process_user_input(input_mode)
        
        update_feedback_loop(
            session_id = session_id,
            user_id = current_user.id
        )
        
        IF config.CLOUD_SYNC_ENABLED:
            sync_to_cloud(session_id)
        
        render_conversation()
    END WHILE

END APPLICATION