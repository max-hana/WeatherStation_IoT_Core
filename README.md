# Weather Station for IoT Core

This project is made from the AndroidThings sample project "Weather Station" hosted [here](https://github.com/AndroidThings/).

These are the only differences:
- Sends data to IoT Core instead of PubSub.
- Works even without a monitor on HDMI
- Tested only with Raspberry Pi 3B with RainbowHat.


## Screenshots

![Weather Station sample demo][demo-gif]

[(Watch the demo on YouTube)][demo-yt]

## Pre-requisites

- Android Things compatible board
- Android Studio 2.2+
- [Rainbow Hat for Android Things](https://shop.pimoroni.com/products/rainbow-hat-for-android-things) or the following individual components:
    - 1 [bmp280 temperature sensor](https://www.adafruit.com/product/2651)
    - 1 [segment display with I2C backpack](https://www.adafruit.com/product/879)
    - 1 push button
    - 1 resistor
    - jumper wires
    - 1 breadboard
    - (optional) 1 [APA102 compatible RGB Led strip](https://www.adafruit.com/product/2241)
    - (optional) 1 [Piezo Buzzer](https://www.adafruit.com/products/160)
    - (optional) [Google Cloud Platform](https://cloud.google.com/) project

## Schematics

If you have the Raspberry Pi [Rainbow Hat for Android Things](https://shop.pimoroni.com/products/rainbow-hat-for-android-things), just plug it onto your Raspberry Pi 3.

![Schematics for Raspberry Pi 3](rpi3_schematics.png)

## Build and install

On Android Studio, click on the "Run" button.
If you prefer to run on the command line, type
```bash
./gradlew installDebug
adb shell am start com.example.androidthings.weatherstation/.WeatherStationActivity
```

If you have everything set up correctly:
- The segment display will show the current temperature.
- If the button is pressed, the display will show the current pressure.
- If a Piezo Buzzer is connected, it will plays a funny sound on startup.
- If a APA102 RGB Led strip is connected, it will display a rainbow of 7 pixels indicating the current pressure.
- If a Google Cloud Platform project is configured (see instruction below), it will publish the sensor data to Google Cloud PubSub.

## Google Cloud Platform Pub/Sub configuration (optional)

1. Go to your project in the [Google Cloud Platform console](https://console.cloud.google.com/)
1. Under *API Manager*, enable the following APIs: Cloud Pub/Sub
1. Under *IAM & Admin*, create a new Service Account, provision a new private key and save the generated json credentials.
1. Under *Pub/Sub*: create a new topic and in the *Permissions* add the service account created in the previous step with the role *Pub/Sub Publisher*.
1. Under *Pub/Sub*: create a new *Pull subscription* on your new topic.
1. Import the project into Android Studio. Add a file named `credentials.json` inside `app/src/main/res/raw/` with the contents of the credentials you downloaded in the previous steps.
1. In `app/build.gradle`, replace the `buildConfigField` values with values from your project setup.

After running the sample, you can check that your data is ingested in Google Cloud Pub/Sub by running the following command:
```
gcloud --project <CLOUD_PROJECT_ID> beta pubsub subscriptions pull <PULL_SUBSCRIBTION_NAME>
```

Note: If there is no `credentials.json` file in `app/src/main/res/raw`, the app will run offline and will not send sensor data to the [Google Cloud Pub/Sub](https://cloud.google.com/pubsub/).


## Create a public key for Iot Core (optional)

When the app is started without any intent information of IoT Core, it just creates a pair of keys and stores the public key to "/sdcard/rpi3_pub.pem" for the later use of device registration in IoT Core.

```
adb shell am start com.example.androidthings.weatherstation/.WeatherStationActivity
```

Note: If there is no `rpi3_pub.pem` file in `/sdcard/`, the app will regenerate its private key and stores the corresponding public key in the same filename.

You can easily copy the public key to local by the following command:

```
adb shell pull /sdcard/rpi3_pub.pem
```


## Register the device to Iot Core (optional)

Now with `rpi3_pub.pem` file, you can register your device to Cloud IoT Core:

```
gcloud iot devices create <DEVICE_ID> --project=<PROJECT_ID> --region=<CLOUD_REGION> --registry=<REGISTRY_ID> --public-key path=rpi3_pub.pem ,type=<CERTIFICATE_TYPE>
```

Where:
- `DEVICE_ID`: your device ID (it can be anything that identifies the device for you)
- `PROJECT_ID`: your Cloud IoT Core project id
- `CLOUD_REGION`: the cloud region for project registry
- `REGISTRY_ID`: the registry name where this device should be registered
- `CERTIFICATE_TYPE`: at this moment choose "rsa-x509-pem" instead of "es256-x509-pem", since your device key algorithm is fixed to "RSA" not "EC".

## Configure the device for IoT Core (optional)

Now that your device's public key is registered, you can start the device app so that it can securely connect to Cloud IoT Core:

```
adb shell am start -e project_id <PROJECT_ID> -e cloud_region <CLOUD_REGION> -e registry_id <REGISTRY_ID> -e device_id <DEVICE_ID>  com.example.androidthings.sensorhub/.SensorHubActivity
```
Where PROJECT_ID, CLOUD_REGION, REGISTRY_ID and DEVICE_ID must be the
corresponding values used to register the device on Cloud IoT Core


## Next steps

Now your weather sensor data is continuously being published to [Google Cloud Pub/Sub](https://cloud.google.com/pubsub/):
- process weather data with [Google Cloud Dataflow](https://cloud.google.com/dataflow/) or [Google Cloud Functions](https://cloud.google.com/functions/)
- persist weather data in [Google Cloud Bigtable](https://cloud.google.com/bigtable/) or [BigQuery](https://cloud.google.com/bigquery/)
- create some weather visualization with [Google Cloud Datalab](https://cloud.google.com/datalab/)
- build weather prediction model with [Google Cloud Machine Learning](https://cloud.google.com/ml/)

## Enable auto-launch behavior

This sample app is currently configured to launch only when deployed from your
development machine. To enable the main activity to launch automatically on boot,
add the following `intent-filter` to the app's manifest file:

```xml
<activity ...>

    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.HOME"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>

</activity>
```

## License

Copyright 2016 The Android Open Source Project, Inc.

Licensed to the Apache Software Foundation (ASF) under one or more contributor
license agreements.  See the NOTICE file distributed with this work for
additional information regarding copyright ownership.  The ASF licenses this
file to you under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License.  You may obtain a copy of
the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the
License for the specific language governing permissions and limitations under
the License.

[demo-yt]: https://www.youtube.com/watch?v=FcdwfKehX_0&list=PLWz5rJ2EKKc-GjpNkFe9q3DhE2voJscDT&index=14
[demo-gif]: demo1.gif
