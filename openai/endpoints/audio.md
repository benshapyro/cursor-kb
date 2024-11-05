# Audio

Learn how to turn audio into text or text into audio.

Related guide: Speech to text

## Create Speech

### POST https://api.openai.com/v1/audio/speech

Generates audio from the input text.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| model | string | Yes | One of the available TTS models: `tts-1` or `tts-1-hd` |
| input | string | Yes | The text to generate audio for. Maximum length is 4096 characters. |
| voice | string | Yes | The voice to use. Supported voices: `alloy`, `echo`, `fable`, `onyx`, `nova`, and `shimmer`. |
| response_format | string | No | The audio format. Supported formats: `mp3` (default), `opus`, `aac`, `flac`, `wav`, and `pcm`. |
| speed | number | No | The speed of generated audio. Range: 0.25 to 4.0. Default: 1.0 |

#### Example Request

```python
from pathlib import Path
import openai

speech_file_path = Path(__file__).parent / "speech.mp3"
response = openai.audio.speech.create(
  model="tts-1",
  voice="alloy",
  input="The quick brown fox jumped over the lazy dog."
)
response.stream_to_file(speech_file_path)
```

## Create Transcription

### POST https://api.openai.com/v1/audio/transcriptions

Transcribes audio into the input language.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file | file | Yes | Audio file in supported formats: flac, mp3, mp4, mpeg, mpga, m4a, ogg, wav, or webm |
| model | string | Yes | Model ID. Currently only `whisper-1` is available |
| language | string | No | Input audio language in ISO-639-1 format |
| prompt | string | No | Optional text to guide model's style or continue previous segment |
| response_format | string | No | Output format: `json` (default), `text`, `srt`, `verbose_json`, or `vtt` |
| temperature | number | No | Sampling temperature between 0 and 1. Default: 0 |
| timestamp_granularities[] | array | No | Timestamp detail level: `word` or `segment`. Requires `verbose_json` format |

#### Example Request

```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.transcriptions.create(
  model="whisper-1",
  file=audio_file
)
```

#### Response

```json
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger. This is a place where you can get to do that."
}
```

## Create Translation

### POST https://api.openai.com/v1/audio/translations

Translates audio into English.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file | file | Yes | Audio file in supported formats: flac, mp3, mp4, mpeg, mpga, m4a, ogg, wav, or webm |
| model | string | Yes | Model ID. Currently only `whisper-1` is available |
| prompt | string | No | Optional text to guide model's style (in English) |
| response_format | string | No | Output format: `json` (default), `text`, `srt`, `verbose_json`, or `vtt` |
| temperature | number | No | Sampling temperature between 0 and 1. Default: 0 |

#### Example Request

```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.translations.create(
  model="whisper-1",
  file=audio_file
)
```

#### Response

```json
{
  "text": "Hello, my name is Wolfgang and I come from Germany. Where are you heading today?"
}
```

## Response Objects

### The Transcription Object (JSON)

Basic JSON response format:

```json
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger. This is a place where you can get to do that."
}
```

### The Transcription Object (Verbose JSON)

Detailed response format with additional metadata:

```json
{
  "task": "transcribe",
  "language": "english",
  "duration": 8.470000267028809,
  "text": "The beach was a popular spot on a hot summer day. People were swimming in the ocean, building sandcastles, and playing beach volleyball.",
  "segments": [
    {
      "id": 0,
      "seek": 0,
      "start": 0.0,
      "end": 3.319999933242798,
      "text": " The beach was a popular spot on a hot summer day.",
      "tokens": [
        50364, 440, 7534, 390, 257, 3743, 4008, 322, 257, 2368, 4266, 786, 13, 50530
      ],
      "temperature": 0.0,
      "avg_logprob": -0.2860786020755768,
      "compression_ratio": 1.2363636493682861,
      "no_speech_prob": 0.00985979475080967
    }
  ]
}
```

The verbose JSON format includes:
- `language`: Input audio language
- `duration`: Audio duration in seconds
- `text`: Transcribed text
- `words`: (When requested) Word-level timestamps
- `segments`: Detailed segment information including timing and confidence metrics
