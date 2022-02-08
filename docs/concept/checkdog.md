# Checkdog

The client shall implement a client-side checkdog logic which repeat every second to monitor signal and engine(webrtc or hls or others) status.

## Track

The client shall track locally for last minute signal activities, room state change, engine status.

## Possible scenarios for all rooms

### Case: Missing join ack

Pre-condition: `initializing` or `reinitializing` state

Scenario: no join ack after 10 seconds

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.

### Case: Critical failure since last checkpoint

Pre-condition: `joined` state

Scenario: There is a critical failure since last checkdog check.

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.

## Possible scenarios for WebRTC

### Case: Consecutive low sent frame warning

Pre-condition: `joined` state

Scenario: Collect 30 consecutive low sent frame warning within a minute.

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.

### Case: Missing answer signal

Pre-condition: `joined` state and room type is `m2m` or `a2m`

Scenario: No answer signal recieved after 10 seconds of offer signal sent.

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.

### Case: P2P peer not arrived

Pre-condition: `joined` state and room type is `p2p`

Scenario: After 30 seconds, the callee is not arrived.

Solution: go to `failed` state.

## Possible scenarios for HLS

### Case: Consecutive high latency warning

Pre-condition: `joined` state and room type is `p2m`

Scenario: The streamer detect consecutive high latency warning for 10 times. High latenct warning is generated when the upload latency is higher than 700 ms.

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.

### Case: Consecutive slow buffering warning

Pre-condition: `joined` state and room type is `p2m`

Scenario: The viewer detect the player stuck at buffering state for 10 seconds.

Solution: recover by resend join signal. if it reaches the maximum retry of 3 times, go to `failed` state.
