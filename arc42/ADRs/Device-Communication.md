# ADR: How do we perform Cloud <-(>) Device Communication?

## Status

Accepted

## Decision Drivers

- Scope Leads
- Developer Team

## Context

PowerMate devices need a way to communicate the measured gas consumption to the cloud.
It is not needed, for now to have a bidirectional communication, i.e. the cloud does not need to send commands to the device.
However, this could be the option for future extensions, i.e. to have the cloud send commands to the device, e.g. to update the firmware.
The communication must be secured, i.e. encrypted, and the device must be authenticated to the cloud.

## Options

### MQTT Protocol with TLS Encryption

Use the MQTT (Message Queuing Telemetry Transport) protocol for communication between the PowerMate devices and the cloud. Implement TLS (Transport Layer Security) encryption to ensure secure data transmission. This option supports future extensions for bidirectional communication if needed.

### HTTP(S) RESTful API

Develop a RESTful API using HTTP or HTTPS for communication between the PowerMate devices and the cloud. Implement HTTPS for encryption and ensure device authentication using API keys or other authentication mechanisms.

### WebSocket Communication with Secure WebSocket (WSS)

Implement WebSocket communication for real-time data transfer between the devices and the cloud. Utilize WSS (Secure WebSocket) to ensure encrypted communication and device authentication.

### AMQP Protocol with SSL/TLS Encryption

Use the AMQP (Advanced Message Queuing Protocol) for communication. Implement SSL/TLS encryption to secure data transmission and device authentication through SSL certificates.

### LoRaWAN Technology

Use the Low Power Wide Area Network (LoRaWAN) technology for long-range, low-power communication between the devices and the cloud. Implement encryption and device authentication mechanisms provided by the LoRaWAN protocol.

## Decision: MQTT Protocol with TLS Encryption

After careful consideration and evaluation of various communication options, the team has decided to adopt the MQTT (Message Queuing Telemetry Transport) protocol with TLS (Transport Layer Security) encryption for enabling Cloud <-> Device communication for the PowerMate devices. This decision is based on the following benefits:

- Efficiency and Low Overhead
- Real-time Communication
- Future Extensibility for bidirectional communication
- Security via TLS
- Authentication via device certificates
- Broker as a Service offering from AWS with IoT Core

## Consequences

Due to the benefits already being mentioned, this section will focus on neutral or negative consequences.

1. Initial learning curve for new protocol
2. Complex TLS setup and certificate management
3. Integration of MQTT over TLS is non trivial for phone devices using the React Native framework, due to bad support of the MQTT library for React Native
