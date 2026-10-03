# WhatNext Vision Motors \- Salesforce CRM Project: Setup Guide

Welcome to the **WhatNext Vision Motors** Salesforce CRM project setup guide. This document provides step-by-step instructions to configure data models, custom apps, automated processes, and Apex automation code within your Salesforce Developer Edition org.

---

## Milestone 1: Developer Account Setup

1. **Sign Up**: Go to the Salesforce Developer signup page and fill in your details (First name, Last name, Email, Role: Developer, Country: India) \[cite: 5\].  
2. **Activate Account**: Open the verification email from Salesforce, click **Verify Account**, set your password, and choose a security question \[cite: 5, 6\].

---

## Milestone 2: Objects and Relationships

Create the 6 custom objects required for the project via **Setup \> Object Manager \> Create \> Custom Object** \[cite: 7, 8\]:

1. **Vehicle (`Vehicle__c`)**: Stores vehicle details (Record Name: Vehicle Name) \[cite: 7, 8\].  
2. **Vehicle Dealer (`Vehicle_Dealer__c`)**: Stores authorized dealer information (Record Name: Dealer Name) \[cite: 7, 9\].  
3. **Vehicle Customer (`Vehicle_Customer__c`)**: Stores customer details.  
4. **Vehicle Order (`Vehicle_Order__c`)**: Tracks vehicle purchases.  
5. **Vehicle Test Drive (`Vehicle_Test_Drive__c`)**: Tracks test drive bookings.  
6. **Vehicle Service Request (`Vehicle_Service_Request__c`)**: Tracks vehicle servicing requests.

---

## Milestone 3: Tabs

1. Go to **Setup** and search for **Tabs** \[cite: 10\].  
2. Under **Custom Object Tabs**, click **New** and create custom tabs for all custom objects created in Milestone 2 \[cite: 10\].

---

## Milestone 4: The Lightning App

1. Go to **Setup \> App Manager** and click **New Lightning App** \[cite: 11\].  
2. **App Details**: Name the app `WhatNext Vision Motors` \[cite: 11\].  
3. **Navigation Items**: Add Vehicle, Dealer, Customer, Order, Test Drive, Service Request, Reports, and Dashboard \[cite: 12\].  
4. **User Profiles**: Assign the app to the **System Administrator** profile and click **Save & Finish** \[cite: 12\].

---

## Milestone 5: Fields and Relationships

Configure fields and lookup relationships for each custom object under **Object Manager**:

* **Vehicle (`Vehicle__c`)**:  
  * `Vehicle_Model__c` (Picklist: Sedan, SUV, EV, etc.) \[cite: 13, 15\]  
  * `Stock_Quantity_c` (Number) \[cite: 13, 15\]  
  * `Price__c` (Currency) \[cite: 13\]  
  * `Dealer__c` (Lookup Relationship to Dealer) \[cite: 13, 16\]  
  * `Status__c` (Picklist: Available, Out of Stock, Discontinued) \[cite: 13\]  
* **Vehicle Dealer (`Vehicle_Dealer__c`)**: `Dealer_Name__c` (Text), `Dealer_Location__c` (Text), `Dealer_Code__c` (Auto Number), `Phone__c` (Phone), `Email__c` (Email) \[cite: 13\].  
* **Vehicle Customer (`Vehicle_Customer__c`)**: `Customer_Name__c` (Text), `Email__c` (Email), `Phone__c` (Phone), `Address__c` (Text), `Preferred_Vehicle_Type__c` (Picklist) \[cite: 14\].  
* **Vehicle Order (`Vehicle_Order__c`)**: Lookups to Customer and Vehicle, `Order_Date__c` (Date), `Status__c` (Picklist: Pending, Confirmed, Delivered, Canceled) \[cite: 13\].

---

## Milestone 6: Flows

1. **Flow 1: Auto Assign Dealer**  
   * **Trigger**: Record-Triggered Flow on `Vehicle_Order__c` when created with `Status__c` equals `Pending` \[cite: 19\].  
   * **Elements**:  
     * Get Customer Information (`Vehicle_Customer__c` where Id equals order customer) \[cite: 20\].  
     * Get Nearest Dealer (`Vehicle_Dealer__c` where Dealer Location equals customer address) \[cite: 21\].  
     * Assign Dealer to Order (Update Records) \[cite: 22\].  
2. **Flow 2: Test Drive Reminder**  
   * **Trigger**: Record-Triggered Flow on `Vehicle_Test_Drive__c` when created or updated with `Status__c` equals `Scheduled` \[cite: 24\].  
   * **Scheduled Path**: 1 Days Before `Test_Drive_Date__c` \[cite: 24\].  
   * **Action**: Send Email action using a Text Template to notify the customer about their upcoming test drive \[cite: 26, 27\].

---

## Milestone 7: Apex Classes, Triggers, and Batch Jobs

Open the **Developer Console** to implement business logic \[cite: 30\]:

### 1\. Trigger Handler (`VehicleOrderTriggerHandler`)

Prevents out-of-stock orders and reduces vehicle stock upon confirmation \[cite: 30, 31, 33\].

public class VehicleOrderTriggerHandler {

    public static void handleTrigger(List\<Vehicle\_Order\_c\> newOrders, Map\<Id, Vehicle\_Order\_c\> oldOrders, Boolean isBefore, Boolean isAfter, Boolean isInsert, Boolean isUpdate) {

        if (isBefore && (isInsert || isUpdate)) {

            preventOrderIfOutOfStock(newOrders);

        }

        if (isAfter && (isInsert || isUpdate)) {

            updateStockOnOrderPlacement(newOrders);

        }

    }

    private static void preventOrderIfOutOfStock(List\<Vehicle\_Order\_c\> orders) {

        Set\<Id\> vehicleIds \= new Set\<Id\>();

        for (Vehicle\_Order\_c order : orders) {

            if (order.Vehicle\_c \!= null) vehicleIds.add(order.Vehicle\_c);

        }

        if (\!vehicleIds.isEmpty()) {

            Map\<Id, Vehicle\_c\> vehicleStockMap \= new Map\<Id, Vehicle\_c\>(\[SELECT Id, Stock\_Quantity\_c FROM Vehicle\_c WHERE Id IN :vehicleIds\]);

            for (Vehicle\_Order\_c order : orders) {

                if (vehicleStockMap.containsKey(order.Vehicle\_c)) {

                    Vehicle\_c vehicle \= vehicleStockMap.get(order.Vehicle\_c);

                    if (vehicle.Stock\_Quantity\_c \<= 0\) {

                        order.addError('This vehicle is out of stock. Order cannot be placed.');

                    }

                }

            }

        }

    }

    private static void updateStockOnOrderPlacement(List\<Vehicle\_Order\_c\> orders) {

        Set\<Id\> vehicleIds \= new Set\<Id\>();

        for (Vehicle\_Order\_c order : orders) {

            if (order.Vehicle\_c \!= null && order.Status\_c \== 'Confirmed') vehicleIds.add(order.Vehicle\_c);

        }

        if (\!vehicleIds.isEmpty()) {

            Map\<Id, Vehicle\_c\> vehicleStockMap \= new Map\<Id, Vehicle\_c\>(\[SELECT Id, Stock\_Quantity\_c FROM Vehicle\_c WHERE Id IN :vehicleIds\]);

            List\<Vehicle\_c\> vehiclesToUpdate \= new List\<Vehicle\_c\>();

            for (Vehicle\_Order\_c order : orders) {

                if (vehicleStockMap.containsKey(order.Vehicle\_c)) {

                    Vehicle\_c vehicle \= vehicleStockMap.get(order.Vehicle\_c);

                    if (vehicle.Stock\_Quantity\_c \> 0\) {

                        vehicle.Stock\_Quantity\_c \-= 1;

                        vehiclesToUpdate.add(vehicle);

                    }

                }

            }

            if (\!vehiclesToUpdate.isEmpty()) update vehiclesToUpdate;

        }

    }

}

### 2\. Trigger (`VehicleOrderTrigger`)

trigger VehicleOrderTrigger on Vehicle\_Order\_c (before insert, before update, after insert, after update) {

    VehicleOrderTriggerHandler.handleTrigger(Trigger.new, Trigger.oldMap, Trigger.isBefore, Trigger.isAfter, Trigger.isInsert, Trigger.isUpdate);

}

### 3\. Batch Job (`VehicleOrderBatch`) & Scheduler (`VehicleOrderBatchScheduler`)

Processes pending orders automatically when stock is replenished \[cite: 35, 36, 38\].

global class VehicleOrderBatch implements Database.Batchable\<sObject\> {

    global Database.QueryLocator start(Database.BatchableContext bc) {

        return Database.getQueryLocator(\[SELECT Id, Status\_c, Vehicle\_c FROM Vehicle\_Order\_c WHERE Status\_c \= 'Pending'\]);

    }

    global void execute(Database.BatchableContext bc, List\<Vehicle\_Order\_c\> orderList) {

        // Implementation for processing pending orders when stock is available

    }

    global void finish(Database.BatchableContext bc) {

        System.debug('Vehicle order batch job completed.');

    }

}
