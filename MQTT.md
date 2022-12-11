# MQTT Topic Definition

  

This document compiles the complete list of MQTT topics that are used by any component. The following is described about each topic:

- MQTT topic in wildcard notation e.g. `request/create-booking/#`
- The component that interacts with this topic e.g. `Booking Manager`
- The type of action they take on this topic e.g. `PUB`
- General purpose e.g. `Request to create a booking`
- Interface e.g. `{ email: string; name: string; ... }`

## Structure

### Request/Response prefix

The first part of the MQTT topic is always either `request` or `response`. Any component that wishes to request data from another component first subscribes to `response/{service}/{id}`, then publishes to `request/{service}/{id}`. The component that responds (which is already subscribed to `request/{service}/#`) first extracts `id` from the topic, then it processes the incoming payload (either internally or by communicating with other components) and publishes the response to `response/{service}/{id}`, where the consuming component is already subscribed to and receives the data.

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

The third and final part of the MQTT topic is the ID, which is generated using `Math.random().toString(36).substring(2, 7)` and returns a string of 5 random characters in base 36 (approx. 60 million unique strings). This ensures, with a sufficient level of confidence, that two components trying to concurrently consume the same service are each publishing and subscribing to a unique topic. For example, Consumer #1 subscribes to `response/login/21a7f` and Consumer #2 subscribes to `response/login/a93d6`. Now, instead of the Authentication Component publishing to `response/login` and potentially mixing up data between consumers, it will instead publish to both topics, meaning each component will get the data they expect.

  

## List of Topics

**Notes:**
- `ObjectId` refers to `mongoose.SchemaTypes.ObjectId`, which is [mongoose's](https://mongoosejs.com/) internal ID type
- `Booking` refers to our System's Booking type  `{ email: string; name: string; clinicId: string; issuance: string; date: string; start: string; end: string; details: string; }`
- `Dentist` refers to our System's Dentist type `{ name: string; username: string; password: string; email: string; clinicId: ObjectId; token: string; }`
- `Clinic` refers to our System's Clinic type `{ name: string; dentists: number; owner: string; address: string; city: string; coordinate: { longitude: number; latitude: number; }; openinghours: { monday: { start: number; end: number; }; ... friday: { start: number; end: number; }; }; `

  

| MQTT Topic | Interface | Component | Action Type | General Purpose |  
|--|--|--|--|--|
|`request/login/{id}`| `{ email: string; password: string; }` | Client/Server| `PUB`| Login with provided credentials 
|||Authentication|`SUB`| Listen for Dentist login requests |
|`response/login/{id}`| `{ token: string; id: ObjectId; clinicId: ObjectId; }` | `{ message: string; } `| Client/Server | `SUB` | Receive Auth details or error message |
|||Authentication| `PUB` | Return login status |
|`request/availability/{id}`|`Booking` | Client/Server | `PUB` | Check if a specified booking is available |
| | | Availability Checker | `SUB` | Receive booking and check availability |
|`response/availability/{id}` | `{ accepted: boolean; }` | Client/Server | `SUB` | Receive the availability of the booking |
| | | Booking Manager | `PUB` | Notify consumer about the availability |
| `request/create-booking/{id}` | `Booking` | Availability Checker | `PUB` | Ask Booking Manager to persist the booking |
|||Booking Manager | `SUB` | Receive booking to persist |
| `response/create-booking/{id}` **NOT USED! AFTER PUB CREATE-BOOKING, AVAILABILITY CLOCKS OUT AND BM PUBS TO AVAILABILITY TOPIC**| `Booking` | Availability Checker | `SUB` | Receive the saved booking |
||| Booking Manager | `PUB` | Send the saved the booking |
|`request/{approved \| denied}/{id}` | `{ _id: ObjectId; }` | Client/Server | `PUB` | Delegate a Booking (deny or approve) |
|||Booking Manager | `SUB` | Receive ID and persist the Booking's status |
|`response/{approved \| denied}/{id}` | `{ message: string; }` | Client/Server | `SUB` | Notify User whether delegation was successful |
|||Booking Manger| `PUB` | Sends delegation result message |
|`request/booking-requests/{id}`| `{ clinicID: ObjectId; }` | Client/Server | `PUB` | Request specified Clinic's Bookings |
|||Booking Manager| `SUB` | Query clinic `clinicID` for Bookings |
|`response/booking-requests/{id}` | `Booking[]` | Client/Server | `SUB` | Get a list of Bookings |
||| Booking Manager | `PUB` | Send a list of Bookings |
| `request/dentist/{id}` | `ObjectId` | Client/Server | `PUB` | Request a specific dentist
||| Clinic Portal | `SUB` | Query the specified dentist |
|`response/dentist/{id}` | `Dentist \| { message: string; }` | Client/Server | `SUB` | Display the returned dentist's information or error message |
|||Clinic Portal| `PUB` | Send error message or queried dentist  |
| `request/clinic/{id}` | `ObjectId` | Client/Server | `PUB` | Request a specific clinic
||| Clinic Portal | `SUB` | Query the specified clinic |
|`response/clinic/{id}` | `Clinic \| { message: string; }` | Client/Server | `SUB` | Display the returned clinic's information or error message |
|||Clinic Portal| `PUB` | Send error message or queried clinic |
| `request/clinics/{id}` | `<empty>` | Client/Server | `PUB` | Request all clinics
||| Clinic Portal | `SUB` | Query all clinics |
|`response/clinics/{id}` | `Clinic[] \| { message: string; }` | Client/Server | `SUB` | Display all clinics information or error message |
|||Clinic Portal| `PUB` | Send error message or queried clinics |



## Use Cases

### Case: Dentist login

| Step | Component | Type | Topic name | Payload | Purpose |
|-|-|-|-|-|-|
| 1 | Client component | Pub | request/login/{id} | { email: "[example123@gmail.com](mailto:example123@gmail.com)", password: "24hjwb134bwnbj13b4jdwb234"} | When the user enters login credentials, the client components publish a message which contains email and password. |
| 2 | Authentication component | Sub | request/login/# | { email: "[example123@gmail.com](mailto:example123@gmail.com)", password: "24hjwb134bwnbj13b4jdwb234"} | When the authentication component receives the topic with the message, the authentication function is initiated and checks the database if the credentials in the payload are valid. |
| 3.a | Authentication component | Pub | response/login/{id} | {message: "Incorrect Credentials"} | If the credentials are invalid the authentication components send back a message to the client which informs the user of the invalid credentials. |
| 4.a | Client component | Sub | response/login/{id} | {message: "Incorrect Credentials"} | The client displays the message coming from the authentication component. |
| 3 | Authentication component | Pub | response/login/{id} | {id:"q3423hgw39234rfn23", token:"24df7s89df78sd7f9ssd0f7ds", clinicId:"1dsfsdg889sdf8f9dfsf"} | If the credentials are valid, the authentication component sends back the email and password together with a token. |
| 4 | Client component | Sub | response/login/{id} | {id:"q3423hgw39234rfn23", token:"24df7s89df78sd7f9ssd0f7ds", clinicId:"1dsfsdg889sdf8f9dfsf"} | The client continues to login the user and stores the token in local storage. |

### Case: Create booking request

| Step | Component | Type | Topic name | Payload | Purpose |
|-|-|-|-|-|-|
| 1 | Client component | Pub | /request/availability/{id} | {"email":"Björn@gmail.com","name": "Björn Svensson ","clinicId": 1, "issuance": "1602406766314", "date": "2020-12-14", "start": "0900", "end": "1000", "details": "My tooth hurts} | When the user selects a time slot for booking they fill out a form. The form is then converted into a payload message which is sent to the availability component. |
| 2 | Availability component | Sub | request/availability/{id} | {Booking Request Object} | When the availability component intercepts the message, it first checks if the payload is valid format and then checks the database if the selected time slot is reserved. |
| 3.a | Availability component | Pub | response/availability/{id} | { accepted: false } | If the time slot is already reserved the availability component sends back a message informing the user. |
| 4.a | Client component | Sub | response/availability/{id} | { accepted: false } | The message is displayed for the user that the selected time slot already is reserved by someone else. |
| 3 | Availability component | Pub | request/create-booking/{id} | {Booking Request Object} | If the time slot is available then the booking message is sent to the booking component. |
| 4 | Booking component | Sub | request/create-booking/{id} | {Booking Request Object} | The booking component takes the incoming message from the availability component and creates and stores the booking in the database. |
| 5 | Booking component | Pub | /response/availability/{id} | {Booking Request Object} | The booking component sends back the created booking request to the client. |
| 6 | Client component | Sub | /response/availability/{id) | {Booking Request Object} | The client displays a popup box with information about the booking request. |

### Case: Approve pending booking request

| Step | Component | Type | Topic name | Payload | Purpose |
| ---- | ----------------- | ---- | --------------------- | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Clinic Component | Pub | /request/approve/{id} | { “_id”: “6388849b7a73466d753304ad”} | When the employee wants to approve the booking request we send a message with the booking requests object\_id to the booking component. |
| 2 | Booking component | Sub | /request/approve/{id} | { “_id”: “6388849b7a73466d753304ad”} | When the Booking component receives the message it finds the booking request with the matching object\_id in our database and updates the “state” field from pending to approved. |
| 3 | Booking component | Pub | response/approve/{id} | { message: "Booking request has been approved" } | Sends back a message informing the client that the booking has been approved. |
| 3.a | Booking component | Pub | response/approve/{id} | { Booking request was not found } | Send back a error message informing the client of the failed request |


### Case: Deny pending booking request

| Step | Component | Type | Topic name | Payload | Purpose |
| ---- | ----------------- | ---- | -------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Clinic Component | Pub | /request/denied/{id} | { “\_id”: “6388849b7a73466d753304ad”} | When the employee wants to deny the booking request we send a message with the booking requests object\_id to the booking component. |
| 2 | Booking component | Sub | /request/denied/{id} | { “\_id”: “6388849b7a73466d753304ad”} | When the Booking component receives the message it finds the booking request with the matching object\_id in our database and updates the “state” field from pending to denied. |
| 3 | Booking component | Pub | response/denied/{id} | { message: "Booking request has been denied" } | Sends back a message informing the client that the booking has been denied. |
| 3.a | Booking component | Pub | response/denied/{id} | { Booking request was not found } | Send back an error message informing the client of the failed request. |

### Case: Retrieving all booking request

| Step | Component | Type | Topic name | Payload | Purpose |
| ---- | ----------------- | ---- | ------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| 1 | Client component | Pub | /request/booking-requests/{id} | {“clinicID”: “6388849b7a73466d753304ad” } | The client sends a request to retrieve all booking requests. |
| 2 | Booking component | Sub | /request/booking-requests/{id} | {“clinicID”: “6388849b7a73466d753304ad” } | When the incoming message is intercepted by the booking component, the booking component queries for the requested clinic\_id. |
| 3 | Booking component | Pub | /response/booking-requests/{id} | {\[list of bookings\]} | The booking component sends back a list of booking requests from that clinic\_id. |
| 4 | Client component | Sub | /request/booking-requests/{id} | {\[list of bookings\]} | The client intercepts the response message and displays the booking. |
| 3.a | Booking component | Pub | /response/booking-requests/{id} | { message: "Bookings could not be found" } | The booking component sends a error message |
| 4.a | Client component | Sub | /request/booking-requests/{id} | { message: "Bookings could not be found" } | The client intercepts the response message and informs the user. |
