# Sparkplug B support for Siemens S7
This is a limited PoC of a Sparkplug B implementation for Siemens S7 1500 PLCs.

## Source project
For MQTT functionalities, the following MQTT implementation was used:
https://github.com/ChristofGroschke/MQTT-Siemens-S7-1500

## Why MQTT/Sparkplug?
Sparkplug is an increasingly popular standard for IIoT use cases which is based on MQTT. It adds certain features to MQTT which are useful in an IIoT context and it standardizes the topic structure and data format for the MQTT messages. The standard can be found on the [Eclipse Sparkplug Homepage](https://sparkplug.eclipse.org/). 

Unfortunately only a few PLCs support Sparkplug natively so far (e.g. from [Opto22](https://www.opto22.com/products)). For Siemens PLCs there is no Sparkplug library available yet. This project is a PoC that implements a limited amount of the Sparkplug specification and of Google Protocol Buffers which is used by Sparkplug to encode messages.

Here are some links where you can find further informations:

[Google Protcol Buffers Specification](https://developers.google.com/protocol-buffers/docs/encoding)

[Sparkplug Specification](https://www.eclipse.org/tahu/spec/Sparkplug%20Topic%20Namespace%20and%20State%20ManagementV2.2-with%20appendix%20B%20format%20-%20Eclipse.pdf)

## What's working?
The following messages can be published from the PLC:
1. CONNECT message + NDEATH: Connect Command message via MQTT 3.1.1 and NDEATH message (Last will message)
2. Publish (NBIRTH)
3. Publish (NDATA)

Fixed MQTT Sparkplug message schema:
- Payload:

  -> UUID: unique ID of the message (32 bit random number as UTF-8 String)
  
  -> timestamp: 64 bit Integer (UNIX time)
  
  -> seq: sequence number 0 - 255 (NBIRTH message has always seq 0)
  
- Metric:

  -> name: metric name (UTF-8 String)
  
  -> value: metric value
  
  -> type: datatype (encoded as 32 bit Integer)
  
  -> timestamp: 64 bit Integer (UNIX time)

## Testing the project
Open TIA Portal and "Add new device", then you have two options.

First option: import source files
1. Download all source files
2. TIA Portal Folder "External source files" -> "Add new external file"
3. Select all downloaded files and import
4. Select all imported files -> Right mouse click: "Generate blocks from source"
5. Open OB1 and call "mqttSparkplugCall" FB, by adding this lines:
    ```
    //Call mqttSparkplugCall FB
    "mqttSparkplugCallDB"();
    ```
Second option: import library
1. Download compressed library file "MqttSparkplug.zal16"
2. TIA Portal Tab "Libraries" -> "Global libraries" -> "Open global library"
3. Drag and drop the files in your project structure
4. Open OB1 and call "mqttSparkplugCall" FB, by adding this lines:
    ```
    //Call mqttSparkplugCall FB
    "mqttSparkplugCallDB"();
    ```
To test this project you can use a HiveMQ broker in a docker container, just chance the IP address in the "ControllingMqttSparkplug" data block. 
All the relevant parameters for connecting to the MQTT broker and for publishing messages are found in the "ControllingMqttSparkplug" data block.

### Operational sequence
When you test the project, the blocks will be called in this order:
1. Cyclic call of the "mqttSparkplug" FB.
2. Make "ControllingMqttSparkplug.con" high, to call "mqttConnect" once. A connection to the MQTT Broker will be established, make sure to enter the right parameters in the "ControllingMqttSparkplug" data block. For the NDEATH message, the "ControllingMqttSparkplug.mqttWillTopic" will be used. For the NBIRHT message, it is necessary that the "ControllingMqttSparkplug.MqttMetrik" is filled with values, this is automatically done by the "mqttSparkplugPrepMetrik" Function.
3. The instance of the "mqttSparplugPublish" FB writes the data for the NBIRTH message in the TCP send buffers and triggers the "mqttSparplug" FB to send the message.
4. Make "ControllingMqttSparkplug.pub" high, to call "mqttPublish" once. For the NDATA message, it is necessary that the "ControllingMqttSparkplug.MqttMetrik" is filled with values, this is automatically done by the "mqttSparkplugPrepMetrik" Function.
5. The instance of the "mqttSparplugPublish" FB writes the data for the NDATA message in the TCP send buffers and triggers the "mqttSparplug" FB to send the message.
6. Make "ControllingMqttSparkplug.discon" high, to disconnect from the MQTT broker.
