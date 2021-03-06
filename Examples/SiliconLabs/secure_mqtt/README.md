# Secure MQTT Example

The purpose of this application is to provide an example of secure MQTT client implementation using a WFx chip and the FMAC driver.
The lwIP stack and Mbed TLS libraries being already used by WFx projects, the natural choice has been to use the MQTT client implementation
provided by the lwIP stack and the TLS layer implementation provided by Mbed TLS. A comforting reason is that Mbed TLS is intended to be easily 
integrated to the lwIP stack. Additionally the Non-Volatile Memory driver, NVM3, provided in the SDK, is used in order to keep the configuration
between reboots and thus simplify the example use.

## Requirements

### Hardware Prerequisites

One of the supported platforms listed below is required to run the example:

* [**EFM32 Giant Gecko GG11 Starter Kit (SLSTK3701A)**](https://www.silabs.com/products/development-tools/mcu/32-bit/efm32-giant-gecko-gg11-starter-kit) with
  [**WF200 Wi-Fi® Expansion Kit (SLEXP8022A)**](https://www.silabs.com/products/development-tools/wireless/wi-fi/wf200-expansion-kit) or
  [**WFM200S Wi-Fi® Expansion Kit (SLEXP8023A)**](https://www.silabs.com/products/development-tools/wireless/wi-fi/wfm200-expansion-kit)
* [**WGM160P Wi-Fi® Module Starter Kit**](https://www.silabs.com/products/development-tools/wireless/wi-fi/wgm160p-wifi-module-starter-kit)

Additionally, a PC is required to configure the board and it can also be used to load a binary file on the board, to compile the Simplicity Studio project or to run a local MQTT broker.

### Software Prerequisites

* The required software includes Simplicity Studio and the Gecko SDK Suite (32-bit MCU, Micrium OS Kernel, and lwIP)
* The example projects available in this repository
* A Serial terminal to communicate with the board. For example, [**Tera Term**](https://osdn.net/projects/ttssh2/releases/) or [**Putty**](https://www.putty.org/)
* A MQTT broker either integrated to a cloud platform like [**AWS IoT Core**](https://aws.amazon.com/) or [**Azure IoT Hub**](https://azure.microsoft.com/), or local for the tests like a [**Mosquitto Broker**](https://mosquitto.org/) 

*The Micrium OS Kernel is designed to run on Silicon Labs devices only and it is free of charge. Lightweight IP (lwIP) is an open-source TCP/IP stack licensed under the BSD license. Mbed TLS is a cryptographic library licensed under Apache-2.0 license.*

## Set Up your Kit

Please follow the instructions related to the platform suiting your case:

* [**EFM32 Giant Gecko GG11 Starter Kit setup**](../shared/doc/slstk3701a/slstk3701a-setup.md)
* [**WGM160P Wi-Fi® Module Starter Kit setup**](../shared/doc/wgm160p/wgm160p-setup.md)

## Start the Example

1. Once the binary file transferred, you should be prompted on the serial terminal to provide information.
2. Enter the SSID and Passkey of the Wi-Fi Access Point you want your product to connect.
3. Wait for the Wi-Fi connection establishment.
4. Enter the MQTT broker address, both IP and Domain address are supported.
5. Enter the port used by the MQTT broker.
6. Enter the MQTT client Id of your device.

	> The choice of a client Id is especially essential when communicating with a cloud service because this element is often used to identify the device.
	Therefore enter the name of the device you gave when registering it into your cloud service. Some cloud services may also require that this client Id
	is present in the *Common Name* field of the device certificate to pass the authentication.

7. Enter the name of the topic you want to publish (It may be imposed by the service).
8. Enter the name of the topic you want to subscribe (It may be imposed by the service).
9. **[Optional]** Set a username and password if required by the MQTT broker.
10. **[Optional]** Set or update the TLS certificates and device private key (Please refer to **TLS Security** section for more information).

	* Copy/Paste each certificate/key (x509 PEM format) in the terminal.
	* Validate each item by pressing Enter.

11. Once the example correctly started, a message containing the LED states is sent every second on the publish topic selected previously.
12. Play with the pushbuttons on the board, on each pressure a message, containing the name of the button pushed, is sent on the publish topic.
And the associated LED is toggled allowing to control the application state.
13. Send a message on the subscribe topic, either from the cloud interface or from another MQTT client, to remotely change the LED state.
The message must follow this format `{"name":"LED0","state":"On"}` to by accepted by the application, where the field _name_ contains the name of LED to change the state (i.e. `LED0` or `LED1`)
and the field _state_ contains the requested state (i.e. `On` or `Off`).

	> Keep in mind to escape the quotes in the message to send if you use a Shell, making the real message to send: `{\"name\":\"LED0\",\"state\":\"On\"}`.
	
14. Execute a reboot, either by pressing the reset button of the board or by unplugging then plugging it again.
The result of this action let you see that all the information regarding the Wi-Fi and MQTT connections have been kept in memory.
The MQTT message sending can restart as before, i.e. with the same configuration, or a new configuration can be applied.
Notice that the Wi-Fi information are independent from the MQTT information, meaning that a new Wi-Fi configuration or a new MQTT configuration can be applied without needing to reconfigure the other one.

	> If the **Page Erage** option is selected in the **advanced settings...** menu of the Simplicity Flash Commander tool, the information stored in the NVM will stay intact during a firmware download,
    as long as the new firmware doesn't overlap.	

## TLS Security

The application is expecting a double authentication between the client (i.e. the device) and the server (i.e. the MQTT broker) which is the most secured and the most used by cloud services.
This is why the **certificate of a Certification Authority (CA)** signing the server certificate, a **device certificate** signed by your own CA and a **device private key** are requested during the start.
The expected format for these information is the **x509 PEM format**.

These information can either be:

* Retrieved from the cloud service used
* Generated using the [**OpenSSL Toolkit**](https://www.openssl.org/)
* Retrieved from this repository (**only for a local test and debug**)

<br>

## Azure IoT Hub

> To test this example with an Azure IoT Hub server, an Azure IoT Hub account is required. First create an account if you don't already have one.

### Configuration

#### CA Certificate

The first thing to do, if it is not already done, is to add your Certification Authority (CA) certificate to your Azure IoT Hub account.
This certificate will be used to authenticate the device certificate, itself signed by this CA certificate.

If you don't already have a CA certificate, you can either:

* Create one from a known certification authority (e.g. [**Let's Encrypt**](https://letsencrypt.org/)). This option is recommended
in the case where the device certificate will be sent to third parties. Indeed, having a CA certificate, itself signed by a well known
certification authority enforces the trust.
* Create one yourself using the [**OpenSSL Toolkit**](https://www.openssl.org/) for instance. This option is sufficient if the device
certificate is used by "internal" services. The Azure IoT Hub service can be considered as "internal" since you manage the CA certificate
used to authenticate the devices.

The instructions below describe how to add your CA certificate to a Azure IoT Hub account:

* Click on **[Add]** from the **[Settings -> Certificates]** page.
* Enter a certificate name.
* Retrieve the CA certificate from your filesystem.
* Click on **[Save]**.

Now that the CA certificate is added, it should appear in the **[Settings -> Certificates]** page with an **_Unverified_** status.
Azure IoT Hub service requires a *Proof a Possession* of the CA certificate to change the status to **_Verified_**. This consist
into challenging you to create a dummy certificate signed by this CA certificate to make sure you have total access to this
CA certificate and private key.

To realize the *Proof of Possession* follow the instructions below:

* Select your CA certificate in the **[Settings -> Certificates]** page.
* Click on **[Generate Verification Code]** in the Certificate Details window.
* Copy the Verification Code.
* Create a new key
* Create a signature request to sign this new key by the CA certificate and enter the Verification Code generated by Azure IoT Hub as Common Name of this request.
* Create the verification certificate with the signature request.
* Upload the verification certificate from the Certificate Details window.
* Finally click on **[Verify]**.

Your CA certificate status should change to a **_Verified_** status.

#### Device Creation

* Click on **[New]** from the **[Explorers -> IoT devices]** page.
* Enter a Device ID.

> Take care to enter a Device ID corresponding to the Common Name field of the device certificate.

* Select **[X.509 CA Signed]**
* Finalize the device creation by clicking on **[Save]**.

### Device Certificate/Key Creation

As for the CA certificate, the Azure IoT Hub let you manage the generation of device certificates and private keys,
the [**OpenSSL Toolkit**](https://www.openssl.org/) provides the means to generate them.

### MQTT communication

| Name             | Value                                      | Comment                                                |
|------------------|--------------------------------------------|--------------------------------------------------------|
| Broker address   | xxxxxxxxxxxx.azure-devices.net             | Imposed                                                |
| Port             | 8883                                       | Imposed                                                |
| Username         | xxxxxxxxxxxx.azure-devices.net/_DeviceId_  | Imposed, e.g. xxxxxxxxxxxx.azure-devices.net/efm32gg11 |
| Password         |                                            | Empty                                                  |
| Publish Topic    | devices/_DeviceId_/messages/events/        | Imposed, e.g. devices/efm32gg11/messages/events/       |
| Subscribe Topic  | devices/_DeviceId_/messages/devicebound/#  | Imposed, e.g. devices/efm32gg11/messages/devicebound/# |

<br>

## AWS IoT Core Service

> To test this example with an AWS server, an AWS account is required. First create an account if you don't already have one.

### Configuration

#### Policy Creation

The policies define the actions allowed or denied to a device (aka Thing in AWS) or a group of devices.

Follow the instructions below to create a policy:

* Click on **[Create]** in **[Secure -> Policies]** page of the IoT Core Service.
* Enter a name describing the policy.
* Add the operation(s) you want to allow or deny. AWS autofills the ARN field with a template, adjust it to fit your needs.
* Finalize the policy creation by clicking on **[Create]**.

Here is an example of what a policy looks like:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:eu-west-3:xxxxxxxxxxxx:client/${iot:Connection.Thing.ThingName}"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:eu-west-3:xxxxxxxxxxxx:topic/${iot:Connection.Thing.ThingName}/data"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": "arn:aws:iot:eu-west-3:xxxxxxxxxxxx:topicfilter/${iot:Connection.Thing.ThingName}/rx"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Receive",
      "Resource": "arn:aws:iot:eu-west-3:xxxxxxxxxxxx:topic/${iot:Connection.Thing.ThingName}/rx"
    }
  ]
}
```

This policy allows only MQTT clients, with an Id corresponding to an existing thing name, to:

* connect to the MQTT broker.
* publish data **only** on the topic **_ThingName_/data**.
* subscribe and allow to receive messages **only** from the topic **_ThingName_/rx**.

> Notice that this Policy is quite restrictive especially concerning the allowed publish and subscribe topics.
Feel free to write your own suiting best your use case.

#### Thing Creation

* Click on **[Create]** in **[Manage -> Things]** page and **[Create a single thing]** in the next page. Only a name is required in this menu.
* Generate the device certificate and keys by selecting the option suiting your use case.

> You can either let AWS generate them, this option is the easiest and quickest especially if you don't have the knowledges about certificate and key generation.
Other options are useful if you already have a device private key, or even your own Certification Authority (CA) allow you to manage all the sensitive information.
These options will require the use of the [**OpenSSL Toolkit**](https://www.openssl.org/).

* **[Download]** the generate certificate and private key from the resulting page.
* **[Activate]** the device from the same page.
* **[Attach a policy]** (without it the thing is basically useless).
* Finalize the thing creation by clicking on **[Register Thing]**.

### AWS certificate

The Amazon server certificate (**Amazon Root CA 1**) is necessary to the device the authenticate the server.
This certificate can be retrieve at [**https://www.amazontrust.com/repository/**](https://www.amazontrust.com/repository/).

### MQTT communication

| Name             | Value                                                                       | Comment                                                |
|------------------|-----------------------------------------------------------------------------|--------------------------------------------------------|
| Broker address   | Available in the **[Settings -> Custom endpoint]** section of Azure IoT Hub | Imposed                                                |
| Port             | 8883                                                                        | Imposed                                                |
| Username         | None                                                                        |                                                        |
| Password         | None                                                                        |                                                        |
| Publish Topic    | Depends on the policies attached to the thing                               | e.g. efm32gg11/data                                    |
| Subscribe Topic  | Depends on the policies attached to the thing                               | e.g. efm32gg11/rx                                      |

<br>

## Local Mosquitto Broker

As mentioned in the **TLS Security** section, certificates and keys examples are provided in this repository to ease the first steps of starting this example in a local environment,
meaning running a MQTT broker on your PC. This section describes how to start a local MQTT broker.

> **The certificates and keys provided by this repository are only to use during tests and debug sessions on a local environment, and should not be used in production or outside of this example.**

### Configuration adaptation

A configuration file example (*.\security_files\mosquitto_tls_examples.conf.sample*) is provided in the repository, the following operations are still required to adapt the configuration to your installation:

1. Copy/paste the file.
2. Rename it, for instance *.\security_files\mosquitto_tls_examples.conf*.
3. Open it and set the according paths to files contained in the **security_files** project folder.

Now the configuration file is set, the MQTT broker can be launched.

### MQTT Broker Launch

1. Open a new shell at the **security_files** folder location.
2. Run the command:

**Windows:** `& 'C:\Program Files\mosquitto\mosquitto.exe' -c .\mosquitto_tls_examples.conf`

**Linux:** `mosquitto -c .\mosquitto_tls_examples.conf`

### Traffic Monitoring

A MQTT client, subscribed to all topics, can be launched for a monitoring purpose.

> This not recommended in a production environment but fits a test environment with a small traffic.

1. Open a new shell at the **security_files** folder location.
2. Run the following command suiting your use case to launch a MQTT client monitoring the traffic:

**Windows:** `& 'C:\Program Files\mosquitto\mosquitto_sub.exe' -h localhost -t "#" -v --cafile .\ca.crt --cert .\mosquitto_client.crt --key .\mosquitto_client.key`

**Linux:** `mosquitto_sub -h localhost -t "#" -v --cafile .\ca.crt --cert .\mosquitto_client.crt --key .\mosquitto_client.key`

### Test the MQTT Broker

Complementarily to the **Traffic Monitoring**, a new MQTT client can be executed to send a message on a topic and ensure that the MQTT broker dispatches the topic the other clients subscribed
to this topic, in this case the MQTT client monitoring the traffic.

1. Open a new shell at the **security_files** folder location.
2. Run the following command suiting your use case:

**Windows:** `& 'C:\Program Files\mosquitto\mosquitto_pub.exe' -h localhost -t "test/broker" -m "Hello World!" --cafile .\ca.crt --cert .\mosquitto_client.crt --key .\mosquitto_client.key`

**Linux:** `mosquitto_pub -h localhost -t "test/broker" -m "Hello World!" --cafile .\ca.crt --cert .\mosquitto_client.crt --key .\mosquitto_client.key`


## NVM3

NVM3 is the name of the driver provided in the SDK and used in this example to keep the configurations between reboots, for more information about NVM3
please refer to [AN1135 Using Third Generation NonVolatile Memory (NVM3) Data Storage](https://www.silabs.com/documents/public/application-notes/an1135-using-third-generation-nonvolatile-memory.pdf).
 