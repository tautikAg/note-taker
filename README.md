Here's a Low-Level Design (LLD) and boilerplate structure for your offline voice-to-notes system using Vosk:

---

### **System Architecture**
```
Audio Input → Preprocessing → Vosk Transcription → Postprocessing → Note Generation → Storage
```

### **File Structure**
```
voice_notes/
├── audio_files/              # Raw recordings
├── models/
│   └── vosk-model-en-us/     # Vosk language model
├── notes/                    # Generated notes
├── src/
│   ├── audio_handler.py
│   ├── transcription.py
│   ├── note_generator.py
│   └── utils/
│       ├── vad.py           # Voice Activity Detection
│       └── file_io.py
└── requirements.txt
```

---

### **Component Breakdown**

**1. Audio Handler (`audio_handler.py`)**
- **Dependencies**: `pyaudio`, `webrtcvad`
- **Functions**:
  ```python
  class AudioRecorder:
      def __init__(self, sample_rate=16000, chunk_size=1024):
          self.sample_rate = sample_rate
          self.chunk_size = chunk_size
          self.format = pyaudio.paInt16
          self.channels = 1

      def record_stream(self, duration=None):
          """Continuous recording with optional time limit"""
          
      def save_wav(self, frames, filename):
          """Save raw audio frames to file"""
  ```

**2. Voice Activity Detection (`utils/vad.py`)**
```python
from webrtcvad import Vad

class VoiceActivityDetector:
    def __init__(self, aggressiveness=3):
        self.vad = Vad(aggressiveness)
    
    def is_speech(self, audio_frame):
        """Detect voice activity in 30ms chunks"""
        return self.vad.is_speech(audio_frame, sample_rate=16000)
```

**3. Transcription Core (`transcription.py`)**
```python
from vosk import Model, KaldiRecognizer

class Transcriber:
    def __init__(self, model_path="models/vosk-model-en-us"):
        self.model = Model(model_path)
        self.recognizer = KaldiRecognizer(self.model, 16000)
    
    def transcribe_stream(self, audio_stream):
        """Real-time streaming transcription"""
        results = []
        for frame in audio_stream:
            if self.recognizer.AcceptWaveform(frame):
                results.append(json.loads(self.recognizer.Result()))
        return " ".join([res['text'] for res in results])
```

**4. Note Generator (`note_generator.py`)**
```python
import ollama

class NoteEngine:
    def __init__(self):
        self.prompt_template = """Convert this transcript into structured notes:
        {transcript}
        
        Format as:
        - Key Concepts
        - Action Items
        - Questions Raised
        - Summary (<100 words)"""
    
    def generate_notes(self, text):
        response = ollama.chat(
            model='deepseek-llm:1.8b',
            messages=[{'role': 'user', 'content': self.prompt_template.format(transcript=text)}]
        )
        return response['message']['content']
```

---

### **Core Dependencies (requirements.txt)**
```
vosk==0.3.45
pyaudio==0.2.13
webrtcvad==2.0.10
ollama==0.1.11
numpy==1.26.0
python-dotenv==1.0.0
```

---

### **Workflow Implementation**
```python
# src/main.py
from audio_handler import AudioRecorder
from utils.vad import VoiceActivityDetector
from transcription import Transcriber
from note_generator import NoteEngine

def main():
    # 1. Initialize components
    recorder = AudioRecorder()
    vad = VoiceActivityDetector()
    transcriber = Transcriber()
    note_engine = NoteEngine()
    
    # 2. Record audio with VAD
    print("Recording... (Press Ctrl+C to stop)")
    audio_stream = recorder.record_stream()
    
    # 3. Process audio in chunks
    speech_frames = []
    for frame in audio_stream:
        if vad.is_speech(frame):
            speech_frames.append(frame)
    
    # 4. Transcribe
    transcript = transcriber.transcribe_stream(speech_frames)
    
    # 5. Generate notes
    notes = note_engine.generate_notes(transcript)
    
    # 6. Save output
    with open(f"notes/{datetime.now().isoformat()}.md", "w") as f:
        f.write(notes)

if __name__ == "__main__":
    main()
```

---

### **Setup Instructions**
1. Download Vosk model:
   ```bash
   wget https://alphacephei.com/vosk/models/vosk-model-en-us-0.22.zip
   unzip vosk-model-en-us-0.22.zip -d models/
   ```
   
2. Install requirements:
   ```bash
   pip install -r requirements.txt
   ```

3. Run Ollama in background:
   ```bash
   ollama serve
   ```

---

### **Key Optimizations**
1. **Audio Format**: Use 16kHz mono PCM for best Vosk compatibility
2. **VAD Settings**: Aggressiveness level 3 filters non-speech effectively
3. **Memory Management**: Process audio in 300ms chunks for real-time performance
4. **Model Choice**: `vosk-model-en-us-0.22` balances accuracy/speed

---

### **Error Handling Additions**
- Add timeout for long pauses
- Implement retry logic for Ollama calls
- Add audio normalization for consistent input levels
- Include fallback for offline model loading

Would you like me to elaborate on any specific component or share the complete boilerplate code?