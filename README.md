# WhatNext Vision Motors \- Salesforce CRM Project: README

## Project Overview

WhatNext Vision Motors Salesforce CRM project focuses on shaping the future of mobility by streamlining vehicle orders, inventory stock validation, automated nearest-dealer assignments using Flows, and scheduled Apex batch processes.

## Features & Implementation

1. **Data Modeling**: Custom objects created for Vehicles, Dealers, Customers, Orders, Test Drives, and Service Requests with appropriate lookup relationships and fields \[cite: 7, 13\].  
2. **Lightning App**: Custom navigation layout configured under the "WhatNext Vision Motors" Lightning App \[cite: 11, 12\].  
3. **Process Automation**:  
   * Record-triggered flow to automatically assign the nearest dealer based on customer location \[cite: 18\].  
   * Record-triggered flow with scheduled path to send test drive reminders \[cite: 23, 24\].  
4. **Apex Triggers & Handlers**: Validates stock availability before order placement and manages inventory adjustments \[cite: 30, 31, 33\].  
5. **Batch & Scheduled Apex**: Batch job to process pending orders upon stock replenishment, scheduled to run daily \[cite: 35, 38, 39\].

## Setup Guide

Please refer to the [Setup Guide](http://setup.guide.md) for step-by-step implementation instructions.