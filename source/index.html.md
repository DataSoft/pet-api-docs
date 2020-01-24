---
title: Personal Emergency Transmitter API Reference

language_tabs:
  - java

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

This document describes the API for accessing the Personal Emergency Transmitter device from third party Android applications.  Requires the PET [application](https://play.google.com/store/apps/details?id=com.datasoft.pet).

# Intents

```java
Intent intent = new Intent();
intent.setClassName("com.datasoft.pet", "com.datasoft.pet.ConnectionService");
```

Communications to the PET ConnectionService is done by `startService` with an explicit intent.  Use of implicit intents with intent filters is no longer available in Android 5.0+.  Use `Intent.setClassName` with a package name of `com.datasoft.pet` and a class name of `com.datasoft.pet.ConnectionService` for all commands going to the PET ConnectionService.  The service returns all responses as Broadcast Intents.

### Commands
Commands supported so far by the service:

* com.datasoft.pet.action.GET_STATUS
* com.datasoft.pet.action.START_SCAN
* com.datasoft.pet.action.STOP_SCAN
* com.datasoft.pet.action.CLEAR_EMERGENCY

### Events
Events reported by the service (broadcast intents):

* com.datasoft.pet.event.UPDATE_STATUS
* com.datasoft.pet.event.DEVICE_FOUND
* com.datasoft.pet.event.DEVICE_CONNECTED
* com.datasoft.pet.event.DEVICE_DISCONNECTED
* com.datasoft.pet.event.EMERGENCY

# Scanning for Devices

## Start Scanning

Use the com.datasoft.pet.action.START_SCAN action to scan for devices.  This will disconnect any currently connected devices while scanning.  There is currently not terribly useful because there isn't a way to select the PET devices using the API.

```java
Intent intent = new Intent("com.datasoft.pet.action.START_SCAN");
intent.setClassName("com.datasoft.pet", "com.datasoft.pet.ConnectionService");
startService(intent);
```

## Device Found

While scanning, devices are returned with a BroadcastIntent with action com.datasoft.pet.event.DEVICE_FOUND.  The service reports every device advertisement seen, it's up to the client to keep track of duplicates.  Device name and address are included in the Intent's extras bundle.

### DEVICE_FOUND parameters

Parameter | Type | Description
--------- | ---- | -----------
device-address | String | BLE address of the PET device
device-name | String | Advertised name of the PET device

```java
BroadcastReceiver deviceFoundReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String address = intent.getStringExtra("device-address");
        String name = intent.getStringExtra("device-name");
        // Do stuff..
    }
};
IntentFilter deviceFilter = new IntentFilter("com.datasoft.pet.event.DEVICE_FOUND");
registerReceiver(deviceFoundReceiver, deviceFilter);
```

## Stop Scanning

To stop scanning for devices, use a com.datasoft.pet.action.STOP_SCAN action.

```java
Intent intent = new Intent("com.datasoft.pet.action.STOP_SCAN");
intent.setClassName("com.datasoft.pet", "com.datasoft.pet.ConnectionService");
startService(intent);
```

# Clear Emergency

To clear the emergency status on the device, use a com.datasoft.pet.action.CLEAR_EMERGENCY action.  This should generally be done as the result of the user purposefully pressing a "Clear Emergency" button on the application or something requiring user interaction.  Note that once an emergency is triggered on the PET device, it can not be cleared from the PET device for one minute.  This is to avoid accidentally clearing an actual emergency, or an attacker clearing the emergency status.  During this time, the emergency can be cleared from the PET application, or through this API call.

```java
Intent intent = new Intent("com.datasoft.pet.action.CLEAR_EMERGENCY");
intent.setClassName("com.datasoft.pet", "com.datasoft.pet.ConnectionService");
startService(intent);
```

# Device Status

To get the status of the PET device, send a com.datasoft.pet.action.GET_STATUS action.

```java
Intent intent = new Intent("com.datasoft.pet.action.GET_STATUS");
intent.setClassName("com.datasoft.pet", "com.datasoft.pet.ConnectionService");
startService(intent);
```

## Status results

The service sends a com.datasoft.pet.event.UPDATE_STATUS BroadcastIntent in response.  Device status fields are in the Intent's extras bundle.

### UPDATE_STATUS parameters

Parameter | Type | Description
--------- | ---- | -----------
device-name | String | Advertised name of PET device
device-address | String | BLE address of front device
device-connected | boolean | True if phone is currently connected to this device
device-battery | int | Battery level of device in percent
device-emergency | int | Emergency status: 0=OK, 1=Emergency

```java
BroadcastReceiver statusReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String name = intent.getStringExtra("device-name");
        String address = intent.getStringExtra("device-address");
        boolean connected = intent.getBooleanExtra("device-connected", false);
        int battery = intent.getIntExtra("device-battery", -1);
        int emergency = intent.getIntExtra("device-emergency", 0);

        // Do stuff..
    }
};
IntentFilter statusFilter = new IntentFilter("com.datasoft.pet.event.UPDATE_STATUS");
registerReceiver(statusReceiver, statusFilter);
```

# Other Device Events

These events are broadcast based on device events and not as a response to an action.

## Device Disconnected

The com.datasoft.pet.event.DEVICE_DISCONNECTED event notifies the client that the PET device is no longer connected to the service.  For example, when the phone goes out of range of the device.  No extra data is included in this intent.

```java
BroadcastReceiver disconnectedReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "PET Device has been disconnected");
    }
};
IntentFilter disconnectedFilter = new IntentFilter("com.datasoft.pet.event.DEVICE_DISCONNECTED");
registerReceiver(disconnectedReceiver, disconnectedFilter);
```

## Device Connected

The com.datasoft.pet.event.DEVICE_CONNECTED event notifies the client that the PET device is connected to the service.  No extra data is included in this intent.

```java
BroadcastReceiver connectedReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "PET Device has been connected");
    }
};
IntentFilter connectedFilter = new IntentFilter("com.datasoft.pet.event.DEVICE_CONNECTED");
registerReceiver(connectedReceiver, connectedFilter);
```

## Emergency Detected

The com.datasoft.pet.event.EMERGENCY event notifies the client that the user has triggered an emergency on the PET device.

### EMERGENCY parameters

Parameter | Type | Description
--------- | ---- | -----------
emergency | int | 0=OK, 1=Emergency

```java
BroadcastReceiver emergencyReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        int emergency = intent.getIntExtra("device-emergency", 0);

        // Do stuff
    }
};
IntentFilter emergencyFilter = new IntentFilter("com.datasoft.pet.event.EMERGENCY");
registerReceiver(emergencyReceiver, emergencyFilter);
```

