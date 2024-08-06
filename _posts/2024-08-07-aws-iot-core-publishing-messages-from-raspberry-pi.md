---
title: AWS IoT Core - Publishing Sensor data to AWS IoT from Raspberry PI with Balena OS
date: 2024-08-06 06:00:00 +1000
categories: [AWS, IoT, Docker]
tags: [AWS, IoT, DevOps, Container-Orchestration]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2024-08-07/cover.png
---

The Internet of Things (IoT) is revolutionizing how we interact with the world around us, enabling devices to collect and exchange data seamlessly. In this blog post, we'll delve into a practical IoT project where we'll publish sensor data from a Raspberry Pi Zero to AWS IoT Core using Balena OS. Our goal is to create a reliable and efficient system for monitoring environmental conditions with a DHT22 sensor and leveraging the power of cloud services for data storage and analysis.

We'll start by setting up our Raspberry Pi Zero with Balena OS, a powerful and easy-to-use platform for managing IoT devices. Using Golang, we'll write a program to read temperature and humidity data from the DHT22 sensor. This data will then be sent to AWS IoT Core, where it's securely transmitted and stored in a DynamoDB table. Notably, this entire project can be accomplished using free-tier resources from both Balena and AWS, making it an accessible and cost-effective solution for IoT enthusiasts.

By the end of this tutorial, you'll have a fully functional IoT setup capable of collecting, transmitting, and storing sensor data. Moreover, we'll lay the groundwork for the next step: creating a Lambda function with API Gateway to access this data through a REST API endpoint, enabling real-time data retrieval and integration with other applications.

Join us on this exciting journey into the world of IoT, where we'll harness the power of Raspberry Pi, Balena OS, and AWS IoT Core to build a scalable and efficient sensor data collection system.

### Creating a Thing in AWS IoT Core

o set up your Raspberry Pi Zero to communicate with AWS IoT Core, you'll need to create a "thing" in the AWS IoT Core console. Follow the steps below to get started on create a thing on AWS for your device.

#### Step 1: Access AWS IoT Core Console

In the AWS Management Console, type **"IoT Core"** in the search bar and select **"AWS IoT Core"** from the list of services.

#### Step 2: Create a Thing

1. In the AWS IoT Core console, click on **"Connect one device"**.
2. Click **"Next"** and then select **"Create a new thing"**.
3. Name your thing (e.g., `RPI_DHT22`).
4. Skip the optional Platform and SDK selection, you can keep the default selections.
5. On the next page, click **"Download connection kit"** to get the certificates bundle.

#### Step 3: Finalize Thing Creation

1. Finish the wizard to finalize the creation of your IoT thing.
2. Unzip the downloaded connection kit. The kit contains a shell script (`start.sh`) to download the root certificate.

#### Step 4: Obtain the Root Certificate

1. Open the unzipped connection kit folder.
2. Open the `start.sh` file find the command for downloading the root certificate.
3. Execute that command in your terminal to download the root certificate. For example:

```bash
    curl https://www.amazontrust.com/repository/AmazonRootCA1.pem > root-CA.crt
```

#### Step 5: Prepare Certificates and Keys

1. Copy the following files to a secure location:
   - `THING_ID.cert.pem` (certificate for the device)
   - `THING_ID.private` (private key for the device)
   - `root-CA.crt` (root certificate)

These items will be needed when connecting your Raspberry Pi Zero to AWS IoT Core to publish sensor data.

By following these steps, you've successfully created a thing in AWS IoT Core and prepared the necessary certificates and keys for secure communication. Lets configure Thing's policy to add our configurations.

---

### Updating the Policy for Your IoT Thing

Now that you have configured your thing in AWS IoT Core, the next step is to update the associated policy. This involves adding the MQTT topic you plan to use from your Raspberry Pi device and whitelisting the Client ID of the MQTT session. Follow these steps to update the policy:

#### Step 1: Access the Policy for Your Thing

In the AWS IoT Core console, navigate to **Security** -> **Policies** from the left-hand navigation panel.

Find and click on the policy associated with your thing.

#### Step 2: Edit the Policy

Click **"Edit active version"** to modify the active policy.

Switch to the **JSON** view to edit the policy inline for convenience.

#### Step 3: Update the Policy Actions

1. **Update `iot:Publish` and `iot:Subscribe` Actions**:

    Add your desired MQTT topic to use.

    For example:

    ```json
        "arn:aws:iot:YOUR_REGION:YOUR_ACCOUNT_ID:topicfilter/rpi/dht22"
    ```

    Replace `YOUR_REGION` and `YOUR_ACCOUNT_ID` with your AWS region and account ID. You can refer to existing records in the policy to determine the correct format.

2. **Verify and Update `iot:Connect` Action**:
    Ensure the `iot:Connect` action includes the client ID you plan to use in your code. Add the following line to use the thing name:

    ```json
    "arn:aws:iot:YOUR_REGION:YOUR_ACCOUNT_ID:client/${iot:Connection.Thing.ThingName}"
    ```

    Replace `YOUR_REGION` and `YOUR_ACCOUNT_ID` with the correct values.

#### Step 4: Save the Updated Policy

1. Tick the box to set the edited version as active.
2. Save the policy to create the new active version.

By following these steps, you have successfully updated the policy for your IoT thing. This ensures that your Raspberry Pi device can communicate with AWS IoT Core using the specified MQTT topic and client ID. Now, you are ready to proceed with configuring your device to publish data to AWS IoT Core.

### Configuring AWS IoT Core to Send Data to DynamoDB

To configure AWS IoT Core to send data to a DynamoDB table, we need to create a DynamoDB table, create an IAM role for AWS IoT to access the DynamoDB table, and create a routing rule to forward messages to the DynamoDB table.

Lets start with creating the DynamoDB table.

#### Step 1: Create a DynamoDB Table

##### Create a New Table

- In the AWS Management Console, type **"DynamoDB"** in the search bar and select **"DynamoDB"** from the list of services.
- In the DynamoDB console, click **"Create table"**.
- **Table name**: Enter `DHT22Data`.
- **Partition key**: Enter `device_id` and set the data type to `String`.
- **Add sort key**: Click to add a sort key, enter `timestamp`, and set the data type to `String`.
- Keep the default settings for the table (e.g., read/write capacity modes).
- Click **"Create"** to create the table.

By following these steps, you have successfully created a DynamoDB table named `DHT22Data` with `device_id` as the partition key and `timestamp` as the sort key. This table will store the sensor data sent from your Raspberry Pi device. Next, we'll create an IAM role for AWS IoT to access the DynamoDB table and set up a routing rule to forward messages to the table.

---

To allow AWS IoT Core to access the DynamoDB table, we need to create an IAM role with the necessary permissions. Follow these steps to create the IAM role:

#### Step 2: Create a New Role

1. In the IAM console, click on **"Roles"** in the left-hand navigation panel.
2. Click **"Create role"**.
3. Select **"AWS account"** as the type of trusted entity.
4. Ensure **"This account"** is selected.
5. Click **"Next: Permissions"**.

#### Step 3: Attach Policies

1. In the **"Attach permissions policies"** section, search for and select the following policies:
   - **"AWSIoTFullAccess"**
   - **"AmazonDynamoDBFullAccess"**
2. Click **"Next: Tags"** (you can optionally add tags, but it's not required).
3. Click **"Next: Review"**.

#### Step 4: Name and Create the Role

1. **Role name**: Enter `IoT_to_DynamoDB_Role`.
2. Review the role details and click **"Create role"**.

#### Step 5: Edit the Trust Policy

1. In the IAM console, find and select the role you just created (`IoT_to_DynamoDB_Role`).
2. Go to the **"Trust relationships"** tab and click **"Edit trust policy"**.
3. Change the **"Principal"** to allow the IoT service to assume this role by modifying the policy as follows:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "iot.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

4. Click **"Update Trust Policy"** to save your changes.

By following these steps, you have successfully created an IAM role named `IoT_to_DynamoDB_Role` with the necessary permissions for AWS IoT Core to access your DynamoDB table. This role will enable your IoT devices to interact with the DynamoDB table to store and retrieve sensor data. Next, we will create a routing rule to forward messages from AWS IoT Core to the DynamoDB table.

---

Now that we have a DynamoDB table and an IAM role with the necessary permissions, the next step is to create a message routing rule in AWS IoT Core to send data to the DynamoDB table. Follow these steps to create the IoT rule:

#### Step 6: Create a New Rule

1. In the AWS IoT Core console, click on **"Act"** in the left-hand navigation panel.
2. Click **"Rules"** and then **"Create"**.
3. **Name your rule**: Enter `RPI_DHT22_to_DynamoDB`.

##### Define the Rule Query Statement

1. In the **SQL statement** section, enter the following SQL query:

   ```sql
   SELECT sensor_id as device_id, timestamp, humidity, temperature, pressure FROM 'rpi/dht22'
   ```

   This query selects the data fields from the MQTT topic `rpi/dht22` and maps them to the corresponding DynamoDB table columns.

##### Add an Action

1. Click **"Add action"**.
2. Choose **"Split message into multiple columns of a DynamoDB table (DynamoDB)"**.
3. Click **"Configure action"**.

##### Configure the DynamoDB Action

1. **Select the DynamoDB table**: Choose the `DHT22Data` table you created earlier.
2. **Hash key value**: Enter `${device_id}` (this value will be provided by your RPI device code).
3. **Range key value**: Enter `${timestamp}`.
4. **Select IAM role**: Choose the IAM role you created (`IoT_to_DynamoDB_Role`).

##### Create the Rule

1. Click **"Add action"** to finalize the configuration.
2. Click **"Create rule"** to complete the rule creation process.

By following these steps, you have successfully created an IoT rule that routes messages from AWS IoT Core to your DynamoDB table. This rule will ensure that sensor data from your Raspberry PI device is stored in the DynamoDB table, allowing you to access and analyze the data efficiently.

---

## Reading DHT22 Sensor with Raspberry Pi on BalenaOS and Sending Data to AWS IoT Core via MQTT

In this section, we'll guide you through configuring your Raspberry Pi device to read sensor data from a DHT22 module and send it to AWS IoT Core using MQTT. We'll use BalenaOS to manage the Raspberry Pi and a Go application to read and publish the sensor data. Below are the steps to achieve this.

### Requirements

We need Raspberry Pi with BalenaOS flashed. Refer their documentation to see how this is can be done using `Balena etcher`

 We are using DHT22 Module connected to Raspberry PI device. I am using GPIO2 of Raspberry PI to connect data pin from DHT22. Following are the pin mapping.

![Pin layout](</assets/img/posts/2024-08-07/RPIZero-DHT22.svg>)

1. Connect the VCC pin of the DHT22 to the 3.3V power pin on the Raspberry Pi.
2. Connect the GND pin of the DHT22 to a ground pin on the Raspberry Pi.
3. Connect the data pin of the DHT22 to GPIO2 (Pin 3) on the Raspberry Pi.

Configured AWS IoT Core broker and the downloaded corticates are needed to connect to AWS IoT Core.

### Setting Up BalenaOS on Raspberry Pi

1. **Install Balena CLI:**
    - Follow the instructions on the [official Balena CLI documentation](https://www.balena.io/docs/reference/cli/#installation) to install the Balena CLI.

2. **Log in to Balena:**

    ```sh
    balena login
    ```

Then you can create a new fleet for this device. Use `Balena Etcher` to flash micro-sd of your Raspberry PI device.

### Environment Variables

We are using the following environment variables. You need to configure your Balena fleet and add these environment variables there.

- `AWS_BROKER`: The URL of your AWS IoT Core broker.
- `AWS_TOPIC`: The MQTT topic to which the sensor data will be published.
- `AWS_CLIENT_ID`: The Client ID for the MQTT client.
- `REFRESH_INTERVAL`: The interval (in seconds) at which the sensor data is read and published.
- `ID`: A unique ID for your sensor.

### Creating the Go Application

Here is the Go code for reading data from the DHT22 sensor and publishing it to AWS IoT Core via MQTT. You can refer [this Git Repo](https://github.com/Lilanga/aws-iot-dht22-sensor) for the completed code.

```go
package main

import (
    "context"
    "crypto/tls"
    "crypto/x509"
    "encoding/json"
    "fmt"
    "log"
    "os"
    "os/signal"
    "strconv"
    "syscall"
    "time"

    "github.com/MichaelS11/go-dht"
    mqtt "github.com/eclipse/paho.mqtt.golang"
    "github.com/joho/godotenv"
)

type SensorData struct {
    Humidity    string `json:"humidity"`
    Temperature string `json:"temperature"`
    Pressure    string `json:"pressure"`
    SensorID    string `json:"sensor_id"`
    Timestamp   string `json:"timestamp"`
}

type App struct {
    sensor       *dht.DHT
    sensorID     string
    currentData  SensorData
    interval     time.Duration
    awsIoTClient mqtt.Client
}

var awsBrokerURL = os.Getenv("AWS_BROKER")
var topic = os.Getenv("AWS_TOPIC")
var awsClientID = os.Getenv("AWS_CLIENT_ID")
var sensorID = os.Getenv("ID")

const (
    qos             = 0
    rootCAPath      = "/app/cert/root-CA.crt"
    certificatePath = "/app/cert/cert.pem"
    privateKeyPath  = "/app/cert/private.key"
)

func NewApp() (*App, error) {
    loadEnvVariables()

    if err := initializeHardware(); err != nil {
        return nil, fmt.Errorf("failed to initialize hardware: %w", err)
    }

    sensor, err := dht.NewDHT("GPIO2", dht.Celsius, "DHT22")
    if err != nil {
        return nil, fmt.Errorf("error creating DHT sensor: %w", err)
    }

    interval := getRefreshInterval()

    fmt.Println("Setting up MQTT client...")
    fmt.Println("Setting up AWS IoT client...")
    awsIoTClient, err := setupAWSIoT()
    if err != nil {
        return nil, fmt.Errorf("failed to setup AWS IoT: %w", err)
    }

    return &App{
        sensor:       sensor,
        sensorID:     sensorID,
        interval:     time.Duration(interval) * time.Second,
        awsIoTClient: awsIoTClient,
    }, nil
}

func (a *App) Run(ctx context.Context) error {
    fmt.Println("Starting sensor data collection...")
    ticker := time.NewTicker(a.interval)
    defer ticker.Stop()

    fmt.Println("Starting sensor data publishing...")
    go a.publishSensorData(ctx, ticker)

    <-ctx.Done()
    log.Println("Shutting down gracefully...")

    // Disconnect from AWS IoT Core
    a.awsIoTClient.Disconnect(250)

    return nil
}

func loadEnvVariables() {
    err := godotenv.Load(".env")
    if err != nil {
        log.Printf("No .env file found: %v", err)
    }
}

func getRefreshInterval() int {
    interval, err := strconv.Atoi(os.Getenv("REFRESH_INTERVAL"))
    if err != nil {
        return 30 // Default to 30 seconds if not set or invalid
    }
    return interval
}

func initializeHardware() error {
    if err := dht.HostInit(); err != nil {
        return fmt.Errorf("host initialization failed: %w", err)
    }
    return nil
}

func loadTLSConfig() (*tls.Config, error) {
    // Load root CA certificate
    rootCA, err := os.ReadFile(rootCAPath)
    if err != nil {
        return nil, fmt.Errorf("failed to load root CA certificate: %w", err)
    }

    // Load client certificate
    cert, err := tls.LoadX509KeyPair(certificatePath, privateKeyPath)
    if err != nil {
        return nil, fmt.Errorf("failed to load client certificate and key: %w", err)
    }

    // Create a certificate pool for the root CA
    rootCAs := x509.NewCertPool()
    if ok := rootCAs.AppendCertsFromPEM(rootCA); !ok {
        return nil, fmt.Errorf("failed to append root CA certificate")
    }

    // Create TLS configuration
    tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
        ClientAuth:   tls.NoClientCert,
        ClientCAs:    nil,
        RootCAs:      rootCAs,
    }

    return tlsConfig, nil
}

func setupAWSIoT() (mqtt.Client, error) {
    // Load TLS configuration
    tlsConfig, err := loadTLSConfig()
    if err != nil {
        return nil, fmt.Errorf("failed to load TLS config: %w", err)
    }

    // Create AWS MQTT client options
    opts := mqtt.NewClientOptions().
        AddBroker(awsBrokerURL).
        SetClientID(awsClientID).
        SetTLSConfig(tlsConfig).
        SetCleanSession(true).
        SetAutoReconnect(true).
        SetKeepAlive(30 * time.Second).
        SetDefaultPublishHandler(func(client mqtt.Client, msg mqtt.Message) {
            fmt.Printf("TOPIC: %s\n", msg.Topic())
            fmt.Printf("MSG: %s\n", msg.Payload())
        }).
        SetOnConnectHandler(func(client mqtt.Client) {
            fmt.Println("Connected to AWS IoT Core")
        }).
        SetConnectionLostHandler(func(client mqtt.Client, err error) {
            fmt.Printf("Connection lost: %v\n", err)
        })

    // Connect to the broker
    client := mqtt.NewClient(opts)
    if token := client.Connect(); token.Wait() && token.Error() != nil {
        return nil, fmt.Errorf("failed to connect to MQTT broker: %w", token.Error())
    }

    return client, nil
}

func (a *App) publishSensorData(ctx context.Context, ticker *time.Ticker) {
    for {
        select {
        case <-ticker.C:
            humidity, temperature, err := a.sensor.ReadRetry(11)
            if err != nil {
                log.Printf("Read error: %v", err)
                continue
            }

            a.currentData = SensorData{
                Humidity:    fmt.Sprintf("%v", humidity),
                Temperature: fmt.Sprintf("%v", temperature),
                Pressure:    "0",
                SensorID:    a.sensorID,
                Timestamp:   time.Now().Format(time.RFC3339),
            }

            jsonData, err := json.Marshal(a.currentData)
            if err != nil {
                log.Printf("Error marshalling JSON: %v", err)
                continue
            }

            // Publish to AWS IoT Core
            token := a.awsIoTClient.Publish(topic, qos, false, jsonData)
            token.Wait()
            if token.Error() != nil {
                log.Fatalf("failed to publish message: %v", token.Error())
            } else {
                log.Printf("Successfully published message to topic: %s", topic)
            }

        case <-ctx.Done():
            return
        }
    }
}

func main() {
    app, err := NewApp()
    if err != nil {
        log.Fatalf("Failed to initialize app: %v", err)
    }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Set up graceful shutdown
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)


 go func() {
        <-sigChan
        cancel()
    }()

    if err := app.Run(ctx); err != nil {
        log.Fatalf("Application error: %v", err)
    }
}
```

### Dockerfile for BalenaOS

Create a `Dockerfile.template` to build and run the Go application on BalenaOS:

```Dockerfile
FROM balenalib/%%BALENA_ARCH%%-alpine-golang:latest-build as builder

RUN apk add git
WORKDIR /build
COPY . .
RUN go mod download
RUN go build -o weather-service .

FROM balenalib/%%BALENA_ARCH%%-alpine-golang:latest-run
WORKDIR /app
COPY --from=builder /build/weather-service .
COPY ./cert /app/cert

# command to run on container start
CMD [ "./weather-service" ]
```

### Docker Compose File

Create a `docker-compose.yml` file in the root directory to define the service for Balena:

```yaml
version: '2'
services:
  sensors:
    build: sensors
    network_mode: host
    privileged: true
    expose:
      - "80"
```

### Deploying the Application

When code is compete, you can using following command to push your application to Balena cloud. This will create a release and deployed to your device over the air.

In the terminal, navigate to the root directory of your project and run:

```sh
balena push <your-app-name>
```

This will build the Docker image and deploy it to your Raspberry Pi device.

### Conclusion

You have successfully set up a Raspberry Pi with BalenaOS to read data from a DHT22 sensor and publish it to AWS IoT Core via MQTT. This setup allows you to monitor temperature, humidity, and pressure data remotely using AWS IoT Core.

You can monitory DynamoDB table as Records will start to appear.  In the next post, we'll create a Lambda function with API Gateway to create a REST API endpoint to read the latest sensor reading from the DynamoDB table. Stay tuned!
