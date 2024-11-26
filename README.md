A quick n' dirty Go port of [py-webrtcvad](https://github.com/wiseman/py-webrtcvad) Voice Activity Detector (VAD).

A VAD classifies a piece of audio data as being voiced or unvoiced. It can be useful for telephony and speech recognition.

The VAD that Google developed for the WebRTC project is reportedly one of the best available, being fast, modern and free.

Overview
-----
A VAD classifies a piece of audio data as being voiced or unvoiced. It is highly beneficial for telephony systems (like FreeSWITCH) and speech recognition pipelines, particularly when dealing with real-time audio streams.

The VAD originally developed for the WebRTC project by Google is one of the best available, being:

Fast
Modern
Free
Installation
Go-get the package. You don’t need to have WebRTC installed.

```bash

go get github.com/aflyingHusky/go-webrtcvad
```
Usage
Example: Processing Audio in FreeSWITCH Environments
This example demonstrates how to use the VAD with raw audio data, tailored for PBX systems:

``` go
package main

import (
	"fmt"
	"io"
	"log"

	"github.com/aflyingHusky/go-webrtcvad"
	"github.com/youpy/go-wav" // For reading WAV files
)

func main() {
	// Load audio (PCM, single-channel, 16-bit, 8kHz for FreeSWITCH)
	reader, err := wav.NewReader("test.wav")
	if err != nil {
		log.Fatal(err)
	}

	// Initialize VAD
	vad, err := webrtcvad.New()
	if err != nil {
		log.Fatal(err)
	}

	// Set VAD mode for PBX (mode 3 = very aggressive)
	if err := vad.SetMode(3); err != nil {
		log.Fatal(err)
	}

	rate := 8000 // FreeSWITCH-compatible sample rate (Hz)
	frame := make([]byte, 320) // 20ms frames for 8kHz audio

	// Validate rate and frame length
	if ok := vad.ValidRateAndFrameLength(rate, len(frame)); !ok {
		log.Fatal("invalid rate or frame length for FreeSWITCH")
	}

	for {
		_, err := io.ReadFull(reader, frame)
		if err == io.EOF || err == io.ErrUnexpectedEOF {
			break
		}
		if err != nil {
			log.Fatal(err)
		}

		// Process the audio frame
		active, err := vad.Process(rate, frame)
		if err != nil {
			log.Fatal(err)
		}

		fmt.Printf("Speech detected: %v\n", active)
	}
}
```
Adjustments for FreeSWITCH
Sampling Rate: FreeSWITCH typically processes audio at 8000 Hz (8kHz), so the example uses this rate.
Frame Size: The frame size is set to 320 bytes (20ms of audio for 8kHz) to match PBX real-time requirements.
Aggressiveness Mode: Set to 3 (very aggressive) for environments with significant background noise, ensuring minimal false positives.
Notes
Ensure that your input audio conforms to PCM, mono, 16-bit, and matches the specified sample rate (e.g., 8kHz).
For real-time use cases, this VAD can be integrated with FreeSWITCH modules for speech detection, such as mod_audio_fork or WebSocket audio streams.
Future Enhancements
Add FreeSWITCH-specific examples and integrations.
Extend support for dynamic mode adjustments based on live call conditions.
Optimize performance for concurrent call processing in PBX environments.
Credits
Original library by [maxhawkins](https://github.com/maxhawkins/go-webrtcvad).
WAV reader by youpy.
License
MIT License

