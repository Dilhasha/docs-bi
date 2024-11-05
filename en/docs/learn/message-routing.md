# Message Routing

In this tutorial, you'll create a service that allows users to reserve appointments at various hospitals. Requests will be directed to the appropriate hospital based on the request payload's content.
To accomplish this, you’ll build a REST service with a single resource in Visual Studio Code using the Ballerina Swan Lake extension. The resource will handle user requests, identify the hospital endpoint based on the hospital ID, forward the request to the specified hospital service to make the reservation, and return the reservation details.

Here’s an overview of the process flow.

<a href="{{base_path}}/assets/img/message-routing/introduction.png"><img src="{{base_path}}/assets/img/message-routing/introduction.png" alt="Message Routing" width="70%"></a>

1. Receive a request with a JSON payload similar to the following.
    ```json title="ReservationRequest.json"
    {
        "patient": {
            "name": "John Doe",
            "dob": "1940-03-19",
            "ssn": "234-23-525",
            "address": "California",
            "phone": "8770586755",
            "email": "johndoe@gmail.com"
        },
        "doctor": "thomas collins",
        "hospital": "grand oak community hospital",
        "hospital_id": "grandoak",
        "appointment_date": "2023-10-02"
    }
    ```
2. Extract the `hospital_id` field and select the corresponding hospital service endpoint.

    * grandoak -> http://localhost:9090/grandoak/categories
    * clemency -> http://localhost:9090/clemency/categories
    * pinevalley -> http://localhost:9090/pinevalley/categories 
   
3. Forward the request to the selected hospital service and retrieve the response which will be similar to the following.
    ```json title="ReservationResponse.json"
    {
       "appointmentNumber": 8,
       "doctor": {
           "name": "thomas collins",
           "hospital": "grand oak community hospital",
           "category": "surgery",
           "availability": "9.00 a.m - 11.00 a.m",
           "fee": 7000.0
       },
       "patient": {
           "name": "John Doe",
           "dob": "1940-03-19",
           "ssn": "234-23-525",
           "address": "California",
           "phone": "8770586755",
           "email": "johndoe@gmail.com"
       },
       "hospital": "grand oak community hospital",
       "confirmed": false,
       "appointmentDate": "2023-10-02"
    }
    ```   
 
## Prerequisites
Download the JAR file for the [backend service](https://github.com/ballerina-guides/integration-tutorials/blob/main/backends/hospital-service/hospitalservice.jar) and execute the following command to start the service.

```bash
$ bal run hospitalservice.jar
```

## Implementation
Follow the steps below to implement the message routing service.

### Step 1: Set up the project structure

### Step 2: Create an Integration Service.
1. In the design view, click on the **`Add Construct`** button.
2. Select **`Service`** from the menu.
3. Select **`HTTP Service`** from the Service Type dropdown.
4. Select **`Design From Scratch`** option.
5. Enter the service name as `HealthCare`, path as `/healthcare`, and `8290` as the port.
6. Click on the **`Create Service`** button to create the new service with the specified configurations.

### Step 3: Add connectors
1. Navigate to design view and click on the **`Add Connector`** button.
2. Search and select the **`HTTP`** connector.
3. Enter the connector name as `grandOakEP`, URL as `http://localhost:9090/grandoak/categories`, and config as `{}`.
4. Click on the **`Create Connector`** button to create the new connector with the specified configurations.
<a href="{{base_path}}/assets/img/message-routing/add-connector.gif"><img src="{{base_path}}/assets/img/message-routing/add-connector.gif" alt="Add Connector" width="70%"></a>
5. Repeat the above steps to add connectors for the `clemency` and `pinevalley` hospitals with the following configurations.

    |Connector Name|URL|Config|
    |---|---|---|
    |grandOakEP|http://localhost:9090/grandoak/categories|`{}`|
    |clemencyEP|http://localhost:9090/clemency/categories|`{}`|
    |pineValleyEP|http://localhost:9090/pinevalley/categories|`{}`|

6. The final connectors will look like below.     
<a href="{{base_path}}/assets/img/message-routing/connectors.png"><img src="{{base_path}}/assets/img/message-routing/connectors.png" alt="Connectors" width="70%"></a>
   
??? info "HTTP Connector"
    To learn more about HTTP client, see [Ballerina HTTP Client](https://ballerina.io/learn/by-example/http-client-send-request-receive-response/).
    See supported client configurations in the [HTTP Client Configurations](https://central.ballerina.io/ballerina/http/2.12.2#ClientConfiguration).

### Step 3: Add a resource method
1. The service will be generated with a default resource named `greeting`. Click on the service to view and update the resource details.
2. Click on three dots appear in front of the `greeting` resource and select edit to view and update the resource details.
3. Change the resource HTTP method to **`POST`**.
4. Change the resource name as `categories/[string category]/reserve`
5. Add a payload parameter named `reservation` to the resource of type `ReservationRequest`.
6. Change the 201 response return type to `ReservationResponse`.
7. Add a new response of type **`HttpNotFound`** under the responses.
8. Click on the **`Save`** button to update the resource with the specified configurations.
   <a href="{{base_path}}/assets/img/message-routing/update-resource.png"><img src="{{base_path}}/assets/img/message-routing/update-resource.png" alt="Update Resource" width="70%"></a>

### Step 4: Add the routing logic
1. Click on the `categories/[string category]/reserve` resource to navigate to the resource implementation designer view.
2. Hover to the arrow after start and click the ➕ button to add a new action to the resource.
3. Select **`Variable`** from the node panel on the left. This variable will be used to store the request payload for the hospital service.
4. Change the variable name to `hospitalRequset`, type as `json` and expression:
    ```ballerina
         {
         patient: reservation.patient.toJson(),
         doctor: reservation.doctor,
         hospital: reservation.hospital,
         appointment_date: reservation.appointment_date
        }
    ```
5. Select **`If`** from the node panel on the left.
6. Enter the conditions as **`If`** **`Else If`** blocks as below for each hospital id and press **`Save`**.
 conditions:
    * grandOak -> `reservation.hospital_id == "grandoak"`
    * clemency -> `reservation.hospital_id == "clemency"`
    * pineValley -> `reservation.hospital_id == "pinevalley"`
   <a href="{{base_path}}/assets/img/message-routing/add-if.png"><img src="{{base_path}}/assets/img/message-routing/add-if.png"" alt="Add If" width="70%"></a>
7. Select the `grandOakEP` condition true path ➕ sign and select **`grandOakEP`** connector from the node panel.
<a href="{{base_path}}/assets/img/message-routing/add-connector-action.png"><img src="{{base_path}}/assets/img/message-routing/add-connector-action.png" alt="Add Connector Action" width="70%"></a>
8. Select **`Post`** from the dropdown. Then fill the required fields with values given below and click **`Save`**.

    |Field|Value|
    |---|---|
    |Variable Name|`oakEPResponse`|
    |Variable Type|`ReserveResponse`|
    |Resource Path|`/[category]/reserve`|
    |message|`hospitalRequset`|

9. Click on the ➕ sign again and select **`Return`** from the node panel.
10. Select the `oakEPResponse` variable from the dropdown and click **`Save`**.
 Repeat the above steps for the `clemency` and `pinevalley` hospitals with the following configurations.
    
    **clemency:**

    |Field| Value                 |
     |---|-----------------------|
    |Variable Name| `clemncyEPResponse`   |
    |Variable Type| `ReserveResponse`     |
    |Resource Path| `/[category]/reserve` |
    |message| `hospitalRequset`     |

    **pinevalley:**

    |Field| Value                  |
    |---|------------------------|
    |Variable Name| `pineValleyEPResponse` |
    |Variable Type| `ReserveResponse`      |
    |Resource Path| `/[category]/reserve`  |
    |message| `hospitalRequset`      |

11. For the else condition, click on the `If` condition `Else` path ➕ sign and add a **`Return`** from the node panel. Enter `http:NOT_FOUND` as the value and click **`Save`**.             
12. The final design will look like below.             
    <a href="{{base_path}}/assets/img/message-routing/final-design.png"><img src="{{base_path}}/assets/img/message-routing/final-design.png"" alt="Final Design" width="70%"></a>

### Step 5: Run the service
1. Start the backend service by executing the following command.
    ```bash
    $ bal run hospitalservice.jar
    ```
2. Click on the **`Run`** on the run button in the top right corner to run the service.
3. The service will start and the service will be available at `http://localhost:8290/healthcare/categories/[category]/reserve`.
4. Use a tool like [Postman](https://www.postman.com/) to send a POST request to the service with the following payload.
   ```bash
    curl -X POST "http://localhost:8290/healthcare/categories/surgery/reserve" \
    -H "Content-Type: application/json" \
    -d '{
    "patient": {
    "name": "John Doe",
    "dob": "1940-03-19",
    "ssn": "234-23-525",
    "address": "California",
    "phone": "8770586755",
    "email": "johndoe@gmail.com"
    },
    "doctor": "thomas collins",
    "hospital_id": "grandoak",
    "hospital": "grand oak community hospital",
    "appointment_date": "2023-10-02"
    }'
   ```
 5. The response will be similar to the following.
    ```json title="ReservationResponse.json"
    {
    "appointmentNumber": 1,
    "doctor": {
    "name": "thomas collins",
    "hospital": "grand oak community hospital",
    "category": "surgery",
    "availability": "9.00 a.m - 11.00 a.m",
    "fee": 7000.0
    },
    "patient": {
    "name": "John Doe",
    "dob": "1940-03-19",
    "ssn": "234-23-525",
    "address": "California",
    "phone": "8770586755",
    "email": "johndoe@gmail.com"
    },
    "hospital": "grand oak community hospital",
    "confirmed": false,
    "appointmentDate": "2023-10-02"
    }
    ```  