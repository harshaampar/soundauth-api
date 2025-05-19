# Soundauth API Documentation

## REST API Endpoints

### Generate WAV File
**Endpoint:** `/api/generate-wav`  
**Method:** POST  
**Description:** Generates a modulated WAV file from text data using the SoundAuth SDK.

**Request Body:**
```json
{
  "data": "Text to encode",
  "range_config": 1,
  "api_key": "your-api-key"
}
```

**Parameters:**
- `data` (string, required): The text data to encode into the WAV file
- `range_config` (integer, required): Transmission range configuration (1-4)
- `api_key` (string, required): Valid API key for authentication

**Response:**
- Success: WAV file download (Content-Type: audio/wav)
- Error: JSON response with error details
  ```json
  {
    "status": "error",
    "message": "Error message"
  }
  ```

**Error Codes:**
- 400: Missing required parameters
- 401: Invalid or expired API key
- 500: Server-side error during processing

**Example:**
```bash
curl -X POST -H "Content-Type: application/json" \
     -d '{"data": "Hello World", "range_config": 1, "api_key": "your-api-key"}' \
     -o output.wav \
     http://api.soundauth.com/api/generate-wav
```
## WebSocket Connection

### Connection Establishment

**URL:** `ws://server:port/`  
**Description:** Establishes a WebSocket connection to the SoundAuth server for real-time audio processing.

**Connection Parameters:**
- `api_key` (query parameter or auth object): Valid API key for authentication
- `X-Demo-Mode` (header, optional): Set to "true" to enable demo mode with temporary API key

**Authentication Methods:**
1. Query parameter: `ws://server:port/?api_key=your-api-key`
2. Auth object:
   ```javascript
   socket.connect({
     auth: {
       api_key: "your-api-key"
     }
   });
   ```
3. Session key reuse: Include `X-Session-Key` header with previously issued temporary key

**Connection Response:**
- Success: `status` event with server information
  ```json
  {
    "message": "Connected to SoundAuth Python Server",
    "sdk_initialized": true,
    "sdk_version": "1.0.0",
    "api_key": "your-api-key",
    "api_key_tier": "PRO"
  }
  ```
- Error: `error` event with error details
  ```json
  {
    "message": "Invalid or expired API key"
  }
  ```

### WebSocket Events

#### Client to Server

**`audio_data`**  
Sends audio data for processing.
```javascript
socket.emit('audio_data', {
  buffer: audioBuffer, // ArrayBuffer or base64 string
  sampleRate: 16000,   // Default: 16000
  channels: 1          // Default: 1
});
```

**`start_recording`**  
Starts recording and processing audio.
```javascript
socket.emit('start_recording');
```

**`stop_recording`**  
Stops recording and finalizes processing.
```javascript
socket.emit('stop_recording');
```

#### Server to Client

**`status`**  
Server status updates.
```json
{
  "message": "Connected to SoundAuth Python Server",
  "sdk_initialized": true,
  "sdk_version": "1.0.0"
}
```

**`recording_started`**  
Confirmation that recording has started.
```json
{
  "timestamp": "20230101_120000",
  "auto_started": true
}
```

**`data_received`**  
Notification that data has been received and decoded.
```json
{
  "payload": "Hello World",
  "payload_len": 11,
  "ssi": 75,
  "range_config": 1,
  "channel": 0
}
```

**`result`**  
Processing status updates.
```json
{
  "status": "processing",
  "bytes_received": 1024
}
```

**`error`**  
Error notifications.
```json
{
  "message": "Error processing audio data"
}
```

**`decode_failed`**  
Notification that decoding failed.
```json
{
  "message": "Decoding failed"
}
```

## Usage Notes

1. The API server automatically initializes the Trill SDK when a WebSocket connection is established
2. Recording starts automatically when a connection is established
3. API usage is tracked based on connection time and is limited according to the API key tier
4. Temporary API keys are available in demo mode with limited usage
5. WebSocket connections are cleaned up when disconnected or when usage limits are reached 