# Documentation

## Purpose:

### Team members Roles, and Tasks:
- Georg Zsolnai (guszsoge@student.gu.se): Product Owner
- Ansis Plepis (gusplean@student.gu.se): Frontend Scrum Master
- Bardia Forooraghi (gusforoba@student.gu.se): Backend Scrum Master
- Ivan Vidackovic (gusvidiv@student.gu.se): Frontend Scrum Member
- John Webb (guswebbjo@student.gu.se): Backend Scrum Member
- Carl Dahlqvist Thuresson (gusdahcan@student.gu.se): Backend Scrum Member
- Daniel Dovhun (gusdovda@student.gu.se): Backend Scrum Member

### Tasks:

Overall Task of the Team
- Dentismo aims to solve all of these problems by providing residents of Gothenburg (users) seeking dentist services, and dentists and clinics providing these services with a web application that brings them together. This means users will be able to pick out the clinic most suitable for them in terms of time and location, and request an appointment with that clinic during any free time slot, and dentists will be able to respond to their appointments by either denying or approving them. 

Individual Contributions:
- See section 6 in [Retrospective 4](https://docs.google.com/document/d/11K9uM2yjYUJ6nMYQHH5Kn9ruBBBFW9tWnzwZryI9NcA/edit#)

### Links to all relevant related team resources (Trello board, source-code repositories etc.) 
- Trello Board: https://trello.com/b/t51qFSzb/dentist-appointment-booking-system
- Project Repository T3: https://git.chalmers.se/courses/dit355/dit356-2022/t-3 (includes all components)
- Google Drive for Documentation: https://drive.google.com/drive/folders/1t9P5LLRflxRhtpsNokmKQcDp6hB2fBKB?usp=share_link


## Software Requirement Specification (SRS):
Written by:

- Ansis Plepis - <gusplean@student.gu.se>
- Carl Dahlqvist Thuresson - <gusdahcan@student.gu.se>
- John Christopher Webb - <guswebbjo@student.gu.se> 

DIT-356 2022 Team 03



## **Introduction**

### **Purpose**

At the time of writing this document (2022.16.12.), there doesn’t exist a unified system where Users can:

- See available clinics providing dentist services in Gothenburg;
- See each clinic’s available time slots;
- Book an appointment with that clinic.

Dentismo aims to solve all of these problems by providing residents of Gothenburg (**users**) seeking dentist services, and dentists and clinics providing these services with a web application that brings them together. This means users will be able to pick out the clinic most suitable for them in terms of time and location, and request an appointment with that clinic during any free time slot, and dentists will be able to respond to their appointments by either denying or approving them.

### **Intended Audience**

This document is intended for

- *All* Scrum members, including developers, the Product Owner and the Scrum Master;
- Stakeholders who are directly impacted by the system requirements, including the Teaching Assistant (TA), Course Teacher (Sam), as well as Clinics and Dentists of Gothenburg providing dentistry services, who are interested in Dentismo representing their services and schedules.

### **Intended Use**

This document is used by:

- Scrum Members to refer to the complete list of the System’s requirements and assumptions as a source of truth during development;
- Stakeholders (TA and Course Teacher) to verify that requirements follow SMART, and that they are fulfilled in Gitlab issues and the Trello board;
- Stakeholders (Clinics and Dentists of Gothenburg) to verify that the system’s requirements and scope are inline with their business needs, as well as use the risks identified for the product to decide whether they want to invest in the product.
### **Scope**

As was discussed in the purpose of the product, since there exist no systems satisfying the mentioned criteria, the product has several benefits, such as:

- Creating a centralized system for booking dentist appointments in Gothenburg, meaning:
  - Dentists and clinics can rely on their services being reliably discoverable;
  - Users can rely on the system to provide them with everything they need to decide which clinic best fits them and get in contact with the clinic;
- The only software required to access the system’s services is a web browser, which is very highly accessible nowadays on virtually every platform - phones, computers, consoles, watches etc.
- Potential to inspire products focusing on providing the same services for other branches of medicine - dermatology, psychiatry, surgery etc.

The objectives of the product are to:

- Deliver a software solution that meets the benefits described above;
- Delivering the solution in a fault tolerant manner by utilizing a Circuit Breaker to prevent the System from crashing during times of very high traffic;
- Delivering the solution in a transparent manner by hiding everything related to the distributed nature of the system from the user.

### **Risks**


|Risk|Technique of mitigating the risk|
| :- | :- |
|The more users Dentismo attracts, the more traffic it will have to handle, therefore if there is an unexpected surge in user activity, the system might crash.|Either switching to a more expensive plan on the cloud, which would enable the server to handle more traffic, or using a CDN to route requests to servers that are geographically closer to the user, ensuring multiple servers are handling traffic concurrently.|
|The more users Dentismo attracts, the more standards it will have to uphold, such as increasing UX, SEO, performance/latency, implementing better accessibility, upholding data privacy regulations, handling legal issues etc.|Investing more resources into frontend development, backend development, as well as management i.e. hiring lawyers, [medicine] area experts, data scientists and engineers etc.|
|If one of the components crashes, the entire system ceases to function.|Either investing more money for a more expensive plan on the cloud, thus making the components less prone to crashes, or investing more resources into backend development for making changes to or even completely refactoring the architecture of the system.|

<br>

## **Overall Description**

### **User Needs**

We have identified two primary users of the system:

- Employees of dentistry clinics, specifically dentists and clerks, will be important users of our system; they can use our services to reach more patients by making their clinic available on our website. The website will also provide them with a simple user interface to organize and handle incoming appointments, which will appear immediately when a patient books an appointment with their clinic, letting the employee know *who* requested the appointment, *when* the appointment is requested, and *why* the appointment is requested;
- Patients seeking dentist services; We see a need to facilitate the process of finding a dentistry clinic and scheduling an appointment. Our application provides this solution by having multiple dentists listed on our website and providing customers with interfaces to easily find a time slot that suits them and allowing them to get in contact with that dentist immediately.

### **Assumptions and Dependencies**

- Frontend clients (i.e. User browsers) can execute JavaScript;
- The app functions on every browser;
- The app requires a stable internet connection to function properly
- The frontend team develops the frontend of the system using [React](https://reactjs.org/) and [MaterialUI](https://mui.com/);
- The backend team develops the backend of the system using [Express](https://expressjs.com/) and the MQTT protocol;
- A monolithic MongoDB database is used for persisting data;
- The frontend software code is written from scratch, with the exception of scaffolding the frontend application using [Create React App](https://create-react-app.dev/);
- The software is finished as per all deliverables in no later than 10 weeks.

<br>

## **System Features and Requirements**

### **Functional Requirements**
---
<br>

1. ### **REST API:**
   1. The system shall use a RESTful API for performing CRUD operations on Dentist entities;
   1. The system shall use a RESTful API for performing CRUD operations on Booking entities;
   1. The system shall use a RESTful API for performing CRUD operations on Clinic entities.

1. ### **Home Interface:**
   1. When the user navigates to the home page, the system shall display the home interface;
   1. The system shall display an “about us” section on the home interface, describing the systems provided services;
   1. The system shall display a “clinics” section on the home interface;
   1. The system shall display all available dentistry clinics in the “clinics” section.

1. ### **Navigation bar:**
   1. The system shall display a navigation bar on every page;
   1. The system shall allow users to navigate to the home page using the navigation bar;
   1. The system shall allow users to navigate to the login page using the navigation bar.

1. ### **Map Interface:**
   1. When the user navigates to a clinic page, the system shall display a Google Maps instance for the map view;
   1. The system shall allow users to interact and navigate with the map view;
   1. The system shall allow users to go to Google Maps via the map instance;
   1. The system shall display the exact location of the dentist clinic on the map view.

1. ### **Calendar Interface:**
   1. When the user navigates to a clinic page, the system shall display a calendar view of the week broken down into time slots;
   1. The system shall populate the calendar view with the navigated clinic’s appointments;
   1. The system shall clearly distinguish which time slots are available and which time slots are unavailable in the calendar view;
   1. The system shall display dates and time slots in the current timezone in the calendar view;
   1. The system shall allow the user to navigate through future and past weeks in the calendar view to book slots;

1. ### **Appointment Form Interface:**
   1. When the user selects a time slot in the calendar view, the system shall display an appointment form prompting all values for creating an appointment;
   1. The system shall alert the users if a mandatory value is missing;
   1. The system shall notify the user when their request has been sent;

1. ### **Appointment Interface:**
   1. When the user navigates to the dashboard page, the appointment interface is displayed;
   1. The system shall display all relevant requests grouped by their date and sorted by time in the appointment interface;
   1. The system shall provide all request information for each request in the appointment interface;
   1. The system shall alert the dentist that their response has sent a message to the relevant user upon request delegation;
   1. The system shall provide an error message if the message could not be sent to the relevant user upon request delegation;
   1. The system shall display a sideview for filtering appointments based on their state in the appointment interface;

1. ### **Login interface:**
   1. When the user navigates to the login page, the system shall display the login interface;
   1. The system shall display a form prompting login credentials in the login interface;
   1. The system shall verify incoming credentials on the login process;
   1. The system shall display the response based on the verification process;
   1. The system shall send users to their relevant clinic page.

1. ### **Documentation:**
   1. The system’s database schema shall be documented with an ER diagram;
   1. The system’s frontend shall be prototyped with Figma designs;
   1. The system's MQTT topics shall be documented;
   1. The system's code shall be documented.

1. ### **Availability Checker:**
   1. The availability checker shall check the incoming booking against other bookings to avoid conflicting times;
   1. The availability checker shall check if the incoming bookings fall within the opening times of the clinic;
   1. The availability checker shall return a response based on whether a conflict occurred;
   1. The availability checker shall subscribe and publish to its corresponding topic on the MQTT broker.

1. ### **Booking Manager:**
   1. The booking manager shall allow for fetching bookings;
   1. The booking manager shall allow for delegating bookings;
   1. The booking manager shall subscribe and publish to its corresponding topic on the MQTT broker.

1. ### **Authentication:**
   1. The authentication component shall verify incoming credentials against existing users;
   1. The authentication component shall respond based on whether or not the credentials are valid;
   1. The authentication component shall subscribe and publish to its corresponding topic on the MQTT broker.

1. ### **Clinic Portal:**
   1. The clinic portal shall allow for fetching clinics and dentists;
   1. The clinic portal shall allow for creating clinics and dentists;
   1. The clinic portal shall subscribe and publish to its corresponding topic on the MQTT broker.

1. ### **MQTT Broker:**
   1. The system shall have a running MQTT broker;
   1. The system components shall connect to the MQTT broker.

### **Nonfunctional Requirements**
---

1. ### **System Security:**
   1. Clients communicate with the system server over HTTPS;
   1. Client’s localStorage is used to store authorization tokens;
   1. Booking requests go through validation before being persisted to the database.

1. ### **System Reliability:**
   1. The server can successfully handle 1000 concurrent requests;
   1. The broker can successfully 
   1. The server handles all thrown exceptions without crashing.

1. ### **System Performance:**
   1. API calls are handled[^1] within 1000 ms.

1. ### **System Usability:**
   1. The frontend pages adhere to material design;
   1. The user receives informative feedback when sending requests.

1. ### **System Scalability:**
   1. The system’s architecture allows for further components to be added to the system;
   1. The system can provide services to 10,000 clinics.

<br>

---
[1]: The requests successfully make the full [Client-Server-Broker-Server-Client](https://git.chalmers.se/courses/dit355/dit356-2022/t-3/documentation/-/blob/main/MQTT.md#structure) path

<br>

## Software Architecture Document (SAD):


Written by: John Webb [guswebbjo@student.gu.se](mailto:guswebbjo@student.gu.se) -- DIT-356 Team 03

### **Description of the conceptual design of the architecture** 

The software architecture of our distributed systems project is composed of several architectural styles and design patterns derived from the Project Specification. The system architecture consists of a client for running the web app, an MQTT Broker for communication between the four different nodes of the system and the server, as well as a monolithic database where components can fetch and save data via mongoose. All communication between components is done via MQTT protocol. As the client is the presentation layer, data is merely fetched by the client via MQTT and displayed on the webpage. In this way, all information and business operations are performed outside of the client. As a distributed system, our system can offer increased performance in parallel execution of tasks with efficient use of resources. 

### **Mapping the Conceptual design onto implementation/technologies.** 

As can be seen from our component diagram, we have built a layered system in which the client forms the presentation later, the MQTT Broker and components comprise the business logic layer, and the monolithic database is alone on the database layer. 



![alt_text](images/componentDiagram.png "image_tooltip")



The components within our distributed system act as filters, and the flow of information between two components carried over MQTT pub-sub acts as the pipe. The individual components receive information via designated MQTT topics. It is then processed and mutated so that it may then be published to the next component which may further augment and filter the data so that it may perform its designated function. An instance of pipe and filter within our system can be seen when new booking requests are sent through our Availability Checker. The component checks if the requested time slot is available, and either returning the data to the client in case of a failure or mutating the request to include the status that the time slot is available for writing it to the database.

Upon starting the clinic portal server, the mqtt client will be connected and the component will be subscribed to topics regarding the flow of information on clinics and dentists, as can be seen below in


<table>
  <tr>
   <td>Clinic Portal subscribing to topics on server start
   </td>
  </tr>
  <tr>
   <td>
        <img src="images/image.png" width="" alt="alt_text" title="image_tooltip">
   </td>
  </tr>
</table>


In a similar fashion, the booking manager will subscribe to the relevant mqtt topics upon starting the component.


<table>
  <tr>
   <td>Booking manager subscribing to topics on server start
   </td>
  </tr>
  <tr>
   <td>
        <img src="images/image2.png" width="" alt="alt_text" title="image_tooltip">
   </td>
  </tr>
</table>



<table>
</tr>
  <tr>
   <td>Figure 1
   </td>
  </tr>
  <tr>
   <td>
        <img src="images/image3.png" width="" alt="alt_text" title="image_tooltip">
   </td>
</table>


For example, a user in the web application will attempt to book an appointment. At this time a message including the booking request JSON Object is sent to the Availability Checker via the ‘/request/create-booking’ topic (Figure 1). Here the booking request is compared against other bookings for the particular clinic from the database. If the booking request has any time conflicts with booked appointments, the availability checker will inform the user on by publishing a message with {“accepted”: false} via ‘/request/create-booking’. The client will inform the user via a snackbar that the appointment was not created. In addition, should the booking request provided by the client fail to be validated by the Availability Checker, a list of the errors will be included in the message. Figure 2 shows an example response in which the booking request failed to be validated and was missing the clinic id.


<table>
<tr>
   <td>Figure 2
   </td>
  </tr>
  <tr>
   <td>
        <img src="images/image4.png" width="" alt="alt_text" title="image_tooltip">
   </td>
  </tr>
</table>

<br>

<table>
<tr>
   <td>Code showing validation and subsequent checking of availabilty for incoming booking
   </td>
  </tr>
  <tr>
   <td>
        <img src="images/image5.png" width="" alt="alt_text" title="image_tooltip">
   </td>
  </tr>
</table>


In the event there are no booking conflicts and the requested time falls within the opening hours within the clinic and {“accepted”: true} is added to the message body along with the booking request JSON object and is published to the booking manager so that the booking may be created and saved to the database. The user will then be informed that the booking has been created via publishing a response message to the client.  This will trigger a snackbar notification that will alert the user that the appointment was created successfully.


####  Identify, state and justify any architecture design decisions or tactics used 

Our design decisions were primarily influenced by the Project Specification which required the inclusion of the three architectural styles: Layered, Pipe-Filter and Publish-Subscribe. 


1. MQTT protocol is used for communication between the components as well as the client.

MQTT uses a lightweight messaging protocol which allows for low-cost and high-efficiency communication between the nodes in our system, allowing for high performance and fast response times across the system.. Additionally the MQTT pub-sub model is extremely reliable due to its lightweight nature, giving it high performance. Additionally it has very useful applications for our system and its distributed nature due to MQTTs strong scalability and flexibility. The broker allows for the easy addition and removal of components from the system without compromising other communication throughout the system. It is as easy as unsubscribing from a topic.

The broker can handle a large amount of requests simultaneously with strong performance.


2. Pipe and filter is used between components to ensure each component possesses a unique purpose

	As each component fulfills a specific purpose, behavior between components is decoupled, making it easier to both understand and debug. Issues become more easily to isolate when each component performs a specific functionality. The pipe and filter style offers a number of benefits, including a high degree of modularity. As a result, the individual components may be more easily replaced or modified without jeopardizing or affecting the overall stability and functions of the greater system. In addition, modular components promote scalability. Adding new components or increasing the capacity of already existing components is easier. Furthermore pipe-filter style is also known for strong fault tolerance performance. If one component should crash or fail, the rest of the system may continue to operate without disruption. 


3. Layered architecture separates parts of our system into presentation, business and database layers.

Using a layered architecture is great for creating separation of concern between components which increases modularity and decoupling throughout the system. Additionally maintainability and clarity are also improved as responsibilities are divided between the various components on each level of the system. This allows for easy identification of errors within the system by isolating issues by layer and component. 


##### Road Ahead:

According to what was discussed above, the creation of Deployment diagrams was a necessary addition to inform stakeholders how the architecture of the system will look like on deployment and how we predict the deployment diagram will look in the future as development continues after launch. Therefore, two deployment diagrams were produced to also demonstrate to stakeholders that the architecture and structure of our project will change and be optimized well after its launch to the public.

<br>

Current Deployment Diagram:

<img src="https://i.imgur.com/LN7yhqw.png" width="" alt="alt_text" title="image_tooltip">

<br>

Future Deployment Diagram:
<img src="https://i.imgur.com/LdIXD7a.png" width="" alt="alt_text" title="image_tooltip">


## Project Management Report (PMR): 
○ describe the project management practices used 
○ report on important project management decisions regarding schedule and 
scope. (weekly updates) 
