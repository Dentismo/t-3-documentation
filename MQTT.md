# MQTT Topic Definition  

This document compiles the complete list of used MQTT topics by all components. The following is described about each topic:
- MQTT topic in wildcard notation e.g. `request/create-booking/#`
- The components who interacts with this topic e.g. `Booking Manager` 
- The type of action they take on this topic e.g. `PUB`
- General purpose e.g. `Request to create a booking`
- Interface e.g. `{ email: string; name: string; ... }`

## Structure

### Request/Response prefix

The first part of the MQTT topic is always either `request` or `response`. Any component that wishes to request data from another component first subscribes to `response/{service}/{id}`, then publishes to `request/{service}/{id}`. The component that responds (which is already subscribed to `request/{service}/#`)  first extracts `id` from the topic, then it processes the incoming payload (either internally or by communicating with other components) and publishes the response to `response/{service}/{id}`, where the consuming component is already subscribed to and receives the data.

### Service name

The second part of the MQTT topic is the service name, which is a short name for the service that the component is trying to consume or supply.
- If a component **Requires** a service, that component subscribes to the `response` and publishes to the `request` topic.
- If a component **Provides** a service, that component subscribes to the `request` and publishes to the `response` topic. 

| Component | Required Services | Provided Services |
|--|--|--|
|Client/Server| `login`, `availability`, `approve`, `denied`, `booking-requests`, `dentist`, `clinic`,`clinics`| - |
|Authentication| - | `login` |
|Availability Checker|`create-booking`|`availability`|
| Booking Manager |-|`create-booking`, `approve`, `denied`, `booking-requests`|
| Clinic Portal |-|`dentist`, `clinic`, `clinics`|

### ID suffix

The third and final part of the MQTT topic is the ID, which is generated using `Math.random().toString(36).substring(2, 7)` and returns a string of 5 random characters in base 36. This ensures that two components trying to concurrently consume the same service are each publishing and subscribing to a unique topic. For example, Consumer #1 subscribes to `response/login/21a7f` and Consumer #2 subscribes to `response/login/a93d6`. Now, instead of the Authentication Component publishing to `response/login` and potentially mixing up data between consumers, it will instead publish to both topics, meaning each component will get the data they expect.

## List of Topics

| MQTT Topic | Interface | Component | Action Type | General Purpose |  
|--|--|--|--|--|
|`request/login/{id}`| `{ email: string; password: string; }` | Client/Server| `PUB`| Login with provided credentials 
|||Authentication|`SUB`| Listen for Dentist login requests |
|`response/login/{id}`| `{ token: string; id: string; clinicId: string; } \| { message: string; } `| Client/Server | `SUB` | Receive Auth details or error message |
|||Authentication| `PUB` | Return login status |
## Use Cases

### Case: Dentist login

| Payload step 1 example     | { email: "example123@gmail.com", password: "24hjwb134bwnbj13bb234"}                             |
| -------------------------- | ----------------------------------------------------------------------------------------------- |
| Payload step 3 example     | {id: "q3423hgw39234rfn23", token:"24df7s89df78sd7f9ssd0f7ds", clinicId: "1dsfsdg889sdf8f9dfsf"} |
| Payload step 3 bad example | {message: "Incorrect Credentials"}                                                              |

| Step | Component                | Type | Topic name          | Purpose                                                                                                                                                                              |
| ---- | ------------------------ | ---- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Client component         | Pub  | request/login/{id}  | When the user enters login credentials, the client components publish a message which contains email and password.                                                                   |
| 2    | Authentication component | Sub  | request/login/#     | When the authentication component receives the topic with the message, the authentication function is initiated and checks the database if the credentials in the payload are valid. |
| 3.a  | Authentication component | Pub  | response/login/{id} | If the credentials are invalid the authentication components send back a message to the client which informs the user of the invalid credentials.                                    |
| 4.a  | Client component         | Sub  | response/login/{id} | The client displays the message coming from the authentication component.                                                                                                            |
| 3    | Authentication component | Pub  | response/login/{id} | If the credentials are valid, the authentication component sends back the email and password together with a token.                                                                  |
| 4    | Client component         | Sub  | response/login/{id} | The client continues to login the user and stores the token in local storage.                                                                                                        |

### Case: Create booking request

| Payload step 1, 3 and 5 | { "email": "Emailg@gmail.com", "name": "Björn","clinicId": 1, "issuance": "1602406766314", "date": "2020-12-14", "start": "0900", "end": "1000", "details": "My tooth hurts} |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Payload step 3.a        | {accepted: false}                                                                                                                                                            |

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

| Payload step 1   | { “\_id”: “6388849b7a73466d753304ad”}            |
| ---------------- | ------------------------------------------------ |
| Payload step 3   | { message: "Booking request has been approved" } |
| Payload step 3.a | { Booking request was not found }                |

| Step | Component         | Type | Topic name            | Purpose                                                                                                                                                                          |
| ---- | ----------------- | ---- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Clinic Component  | Pub  | /request/approve/{id} | When the employee wants to approve the booking request we send a message with the booking requests object_id to the booking component.                                           |
| 2    | Booking component | Sub  | /request/approve/{id} | When the Booking component receives the message it finds the booking request with the matching object_id in our database and updates the “state” field from pending to approved. |
| 3    | Booking component | Pub  | response/approve/{id} | Sends back a message informing the client that the booking has been approved.                                                                                                    |
| 3.a  | Booking component | Pub  | response/approve/{id} | Send back a error message informing the client of the failed request                                                                                                             |

### Case: Deny pending booking request

| Payload step 1   | { “\_id”: “6388849b7a73466d753304ad”}          |
| ---------------- | ---------------------------------------------- |
| Payload step 3   | { message: "Booking request has been denied" } |
| Payload step 3.a | { Booking request was not found }              |

| Step | Component         | Type | Topic name           | Purpose                                                                                                                                                                        |
| ---- | ----------------- | ---- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | Clinic Component  | Pub  | /request/denied/{id} | When the employee wants to deny the booking request we send a message with the booking requests object_id to the booking component.                                            |
| 2    | Booking component | Sub  | /request/denied/{id} | When the Booking component receives the message it finds the booking request with the matching object_id in our database and updates the “state” field from pending to denied. |
| 3    | Booking component | Pub  | response/denied/{id} | Sends back a message informing the client that the booking has been denied.                                                                                                    |
| 3.a  | Booking component | Pub  | response/denied/{id} | Send back an error message informing the client of the failed request.                                                                                                         |

### Case: Retrieving all booking request

| Payload 1        | {“clinicID”: “6388849b7a73466d753304ad” }  |
| ---------------- | ------------------------------------------ |
| Payload step 4   | {\[list of bookings\}                      |
| Payload step 3.a | { message: "Bookings could not be found" } |

| Step | Component         | Type | Topic name                      | Purpose                                                                                                                       |
| ---- | ----------------- | ---- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1    | Client component  | Pub  | /request/booking-requests/{id}  | The client sends a request to retrieve all booking requests.                                                                  |
| 2    | Booking component | Sub  | /request/booking-requests/{id}  | When the incoming message is intercepted by the booking component, the booking component queries for the requested clinic_id. |
| 3    | Booking component | Pub  | /response/booking-requests/{id} | The booking component sends back a list of booking requests from that clinic_id.                                              |
| 4    | Client component  | Sub  | /request/booking-requests/{id}  | The client intercepts the response message and displays the booking.                                                          |
| 3.a  | Booking component | Pub  | /response/booking-requests/{id} | The booking component sends a error message                                                                                   |
| 4.a  | Client component  | Sub  | /request/booking-requests/{id}  | The client intercepts the response message and informs the user.                                                              |
