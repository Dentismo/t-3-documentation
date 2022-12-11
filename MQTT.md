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
### Case: Dentist login




| Step | Component                 | Type | Topic name          | Purpose                                                                                                                                                                              |
| ---- | ------------------------- | ---- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Client component          | Pub  | request/login/{id}  | When the user enters login credentials, the client components publish a message which contains email and password.                                                                   |
| 2    | Authentication  component | Sub  | request/login/#     | When the authentication component receives the topic with the message, the authentication function is initiated and checks the database if the credentials in the payload are valid. |
| 3.a  | Authentication component  | Pub  | response/login/{id} | If the credentials are invalid the authentication components send back a message to the client which informs the user of the invalid credentials.                                    |
| 4.a  | Client component          | Sub  | response/login/{id} | The client displays the message coming from the authentication component.                                                                                                            |
| 3    | Authentication component  | Pub  | response/login/{id} | If the credentials are valid, the authentication component sends back the email and password together with a token.                                                                  |
| 4    | Client component          | Sub  | response/login/{id} | The client continues to login the user and stores the token in local storage.                                                                                                        |

### Case: Create booking request


| Step | Component              | Type | Topic name                  | Purpose                                                                                                                                                                    |
| ---- | ---------------------- | ---- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Client component       | Pub  | /request/availability/{id}  | When the user selects a time slot for booking they fill out a form. The form is then converted into a payload message which is sent to the availability component.         |
| 2    | Availability component | Sub  | request/availability/#      | When the availability component intercepts the message, it first checks if the payload is valid format and then checks the database if the selected time slot is reserved. |
| 3.a  | Availability component | Pub  | response/availability/{id}  | If the time slot is already reserved the availability component sends back a message informing the user.                                                                   |
| 4.a  | Client component       | Sub  | response/availability/{id}  | The message is displayed for the user that the selected time slot already is reserved by someone else.                                                                     |
| 3    | Availability component | Pub  | request/create-booking/{id} | If the time slot is available then the booking message is sent to the booking component.                                                                                   |
| 4    | Booking component      | Sub  | request/create-booking/#    | The booking component takes the incoming message from the availability component and creates and stores the booking in the database.                                       |
| 5    | Booking component      | Pub  | /response/availability/{id} | The booking component sends back the created booking request to the client.                                                                                                |
| 6    | Client component       | Sub  | /response/availability/{id) | The client displays a popup box with information about the booking request.                                                                                                |


### Case: Approve pending booking request

| Step | Component         | Type | Topic name            | Purpose                                                                                                                                                                           |
| ---- | ----------------- | ---- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Clinic Component  | Pub  | /request/approve/{id} | When the employee wants to approve the booking request we send a message with the booking requests object\_id to the booking component.                                           |
| 2    | Booking component | Sub  | /request/approve/{id} | When the Booking component receives the message it finds the booking request with the matching object\_id in our database and updates the “state” field from pending to approved. |
| 3    | Booking component | Pub  | response/approve/{id} | Sends back a message informing the client that the booking has been approved.                                                                                                     |
| 3.a  | Booking component | Pub  | response/approve/{id} | Send back a error message informing the client of the failed request                                                                                                              |

### Case: Deny pending booking request

| Step | Component         | Type | Topic name           | Purpose                                                                                                                                                                         |
| ---- | ----------------- | ---- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Clinic Component  | Pub  | /request/denied/{id} | When the employee wants to deny the booking request we send a message with the booking requests object\_id to the booking component.                                            |
| 2    | Booking component | Sub  | /request/denied/{id} | When the Booking component receives the message it finds the booking request with the matching object\_id in our database and updates the “state” field from pending to denied. |
| 3    | Booking component | Pub  | response/denied/{id} | Sends back a message informing the client that the booking has been denied.                                                                                                     |
| 3.a  | Booking component | Pub  | response/denied/{id} | Send back an error message informing the client of the failed request.                                                                                                          |

### Case: Retrieving all booking request

| Step | Component         | Type | Topic name                      | Purpose                                                                                                                        |
| ---- | ----------------- | ---- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Client component  | Pub  | /request/booking-requests/{id}  | The client sends a request to retrieve all booking requests.                                                                   |
| 2    | Booking component | Sub  | /request/booking-requests/{id}  | When the incoming message is intercepted by the booking component, the booking component queries for the requested clinic\_id. |
| 3    | Booking component | Pub  | /response/booking-requests/{id} | The booking component sends back a list of booking requests from that clinic\_id.                                              |
| 4    | Client component  | Sub  | /request/booking-requests/{id}  | The client intercepts the response message and displays the booking.                                                           |
| 3.a  | Booking component | Pub  | /response/booking-requests/{id} | The booking component sends a error message                                                                                    |
| 4.a  | Client component  | Sub  | /request/booking-requests/{id}  | The client intercepts the response message and informs the user.                                                               |



