# MQTT Topic Definition

This document compiles the complete list of used MQTT topics by all components. The following is described about each topic:

- MQTT topic in wildcard notation e.g. `request/create-booking/#`
- The components who interacts with this topic e.g. `Booking Manager`
- The type of action they take on this topic e.g. `PUB`
- General purpose e.g. `Request to create a booking`
- Interface e.g. `{ email: string; name: string; ... }`

## Structure

### Request/Response prefix

Whoever requests data publishes to `request/*/{id}` and subscribes to `response/*/{id}`. The receiver subscribes to `request/*/#` and publishes to `response/*/{id}` (the topic where the client is subscribed to)

### Service name

Available services:

- create booking
- check availability etc

### ID suffix

Why?
How?

## List of Topics

| MQTT Topic           | Interface                              | Component      | Action Type | General Purpose                   |
| -------------------- | -------------------------------------- | -------------- | ----------- | --------------------------------- |
| `request/login/{id}` | `{ email: string; password: string; }` | Server/Client  | PUB         | Login with provided credentials   |
|                      |                                        | Authentication | SUB         | Listen for Dentist login requests |

## Use Cases

### Dentist logs in to the Dentist Page (Dashboard)

### User sends a Booking request

### Dentist approves pending Booking request

### Dentist denies pending Booking request

### Dashboard retrieves list of all Booking requests
