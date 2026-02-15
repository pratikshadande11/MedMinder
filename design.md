# Design Document: MedMinder

## Overview

MedMinder is an AI-powered smart medication adherence system that combines hardware, mobile app, and cloud backend to ensure elderly patients take the right medicine at the right time. The system uses weight sensors for intake detection, LED indicators for guidance, intelligent alarms for reminders, and AI for behavior analysis and refill prediction.

**Core Innovation**: Sensor-driven verification (not user-reported), guardian-controlled AI insights (human-in-the-loop), and predictive refill alerts to prevent stock-outs.

## System Architecture

### Three-Component Design

**1. Smart Pill Box (Hardware)**
- Compartment-based design where each compartment holds one specific medicine
- Each compartment has an LED indicator and weight sensor
- Shared alarm system (buzzer) for audio alerts
- ESP32 microcontroller with WiFi connectivity
- Battery-powered with offline capability

**2. Mobile Application (Android)**
- Separate interfaces for patients and guardians
- Real-time notifications and adherence tracking
- Schedule management and behavior insights
- Offline support with local caching

**3. Backend System (Firebase)**
- Firestore database for all data storage
- Cloud Functions for scheduled tasks and AI processing
- Firebase Authentication for user management
- Cloud Messaging for push notifications

### Data Flow

```
Smart Pill Box ←→ Firebase Backend ←→ Mobile App
    (WiFi)              (HTTPS)
```

All components sync through Firebase as the central hub. The smart pill box sends intake events to Firebase, which then pushes updates to the mobile app in real-time.

## Smart Pill Box Design

### Hardware Components

**Compartments**: Each compartment is independent with its own LED and weight sensor. Compartments are labeled (1, 2, 3...) and mapped to specific medicines in the backend.

**Weight Sensors**: Load cells with 0.5g accuracy detect when medicine is removed. Weight data is used for both intake detection and consumption tracking.

**LED Indicators**: Individual LEDs per compartment glow to show which medicine to take. LEDs remain on during the entire alarm cycle until medicine is taken.

**Alarm System**: Shared piezo buzzer provides audio alerts. Alarm can be snoozed but not dismissed by the patient.

**Microcontroller**: ESP32 handles sensor reading, LED control, alarm triggering, and WiFi communication with backend.

### Medication Intake Flow

1. **Scheduled Time Arrives**: Backend triggers alarm command to smart pill box
2. **Alarm Activates**: Buzzer sounds and correct compartment LED glows
3. **Patient Response**: Patient has 2 minutes to take medicine (intake window)
4. **Weight Detection**: Sensor detects medicine removal
5. **Alarm Stops**: Both alarm and LED turn off immediately
6. **Backend Update**: Intake event recorded with timestamp

### Alarm Repetition Logic

- If medicine not taken within 2 minutes, alarm repeats after a 2-minute gap
- Maximum 3 alarm attempts per scheduled dose
- LED stays on continuously during all attempts (up to 6 minutes total)
- After 3 failed attempts, dose marked as missed and guardian notified

### Snooze Behavior

- Patient can snooze alarm up to 3 times per dose
- Snooze pauses alarm for 2 minutes then resumes
- LED remains continuously on during snooze (independent of alarm sound)
- Alarm stops only when weight sensor detects medicine removal

### Offline Operation

- Smart pill box stores schedule locally
- Continues alarm operations without internet
- Stores intake events in local memory
- Uploads all events when connectivity restored

## Mobile Application Design

### Patient Interface

**Today's Schedule View**: Shows all medications for the day with times and status (taken/missed/upcoming).

**Adherence Dashboard**: Visual summary of medication adherence with percentage and recent history.

**Snooze Control**: Button to snooze active alarms (maximum 3 times per dose).

**Notifications**: Real-time alerts for medication times and reminders.

### Guardian Interface

**Multi-Patient Dashboard**: View all supervised patients and their adherence status at a glance.

**Medication Management**: Create, edit, and assign medications to compartments. Set schedules with multiple daily times.

**Behavior Insights**: View AI-analyzed patterns showing when patient struggles with adherence (time-of-day difficulties, frequent delays, missed dose patterns).

**Schedule Adjustment**: Review AI recommendations and approve timing changes based on behavior patterns.

**Refill Alerts**: Receive early warnings when medicines are running low with estimated days remaining.

**Missed Dose Notifications**: Immediate alerts when patient misses medication after all alarm attempts.

### Role-Based Access

- Patients see only their own data and cannot modify schedules
- Guardians can manage multiple patients and configure all settings
- Guardian assignment requires patient consent
- Patients can revoke guardian access anytime

## AI Integration

### AI Feature #1: Behavior Pattern Analysis

**Purpose**: Identify when and why patients struggle with medication adherence.

**How It Works**:
- Analyzes historical adherence data (intake times, delays, missed doses)
- Identifies patterns: time-of-day difficulties, day-of-week trends, specific medication issues
- Converts raw data into clear, explainable insights

**Human-in-the-Loop**:
- AI only identifies patterns, never takes action automatically
- Guardian reviews insights in the mobile app
- Guardian decides whether to adjust schedules or take other action
- All schedule changes require explicit guardian approval

**Example Patterns**:
- "Patient frequently misses morning 8 AM dose but takes 10 AM doses on time"
- "Weekend adherence is 40% lower than weekdays"
- "Medicine A has 90% adherence, Medicine B has 60% adherence"

**Ethical AI**: System respects patient autonomy by keeping humans in control of all decisions.

### AI Feature #2: Intelligent Refill Prediction

**Purpose**: Predict when medicines will run low and alert guardian before stock-out.

**How It Works**:
- Weight sensors continuously monitor medicine quantity in each compartment
- AI tracks weight reduction rate over time (consumption rate)
- Predicts when medicine will run low based on current weight and consumption pattern
- Sends refill alert to guardian with estimated days remaining

**Benefits**:
- Prevents missed doses due to empty compartments
- Reduces guardian stress by providing advance notice
- Accounts for variable consumption patterns (some medicines taken more consistently than others)

**Example Alert**: "Medicine A in Compartment 2 will run out in 5 days. Current weight: 15g. Consumption rate: 3g/day."

## Backend System Design

### Data Storage (Firestore)

**Collections**:
- `users`: User accounts with roles (patient/guardian)
- `patients`: Patient profiles with guardian links and pill box assignments
- `medications`: Medicine details, compartment assignments, and schedules
- `intakeEvents`: Historical record of all medication intake (taken/missed/snoozed)
- `behaviorPatterns`: AI-analyzed patterns for each patient
- `refillAlerts`: Predicted low inventory alerts

### Cloud Functions

**Scheduled Alarm Trigger**: Runs every minute to check for medications due and sends alarm commands to smart pill boxes.

**AI Behavior Analysis**: Runs daily to analyze adherence data and identify patterns for each patient.

**Refill Prediction**: Runs every 6 hours to calculate consumption rates and predict low inventory.

**Guardian Notifications**: Triggered when missed doses or refill alerts occur, sends push notifications via Firebase Cloud Messaging.

### Real-Time Synchronization

- Firestore real-time listeners keep mobile app in sync with backend
- Changes propagate to app within 10 seconds
- Smart pill box syncs schedule updates within 10 seconds
- Intake events uploaded within 5 seconds of detection

### Security

- Firebase Authentication for user login
- Role-based access control (patient vs guardian permissions)
- Data encryption at rest and in transit
- Audit logs for all data access and modifications
- Patient consent required for guardian assignment

## Key Design Decisions

**Why compartment-based (not date/time-based)?**
- Simpler for elderly patients to understand
- LED clearly shows which medicine to take
- Reduces confusion and medication errors

**Why sensor-driven (not user-reported)?**
- Eliminates false reporting (intentional or accidental)
- Provides objective adherence data
- Enables accurate refill prediction

**Why guardian-controlled AI?**
- Respects patient autonomy and dignity
- Prevents inappropriate automatic interventions
- Builds trust through transparency
- Aligns with ethical AI principles

**Why snooze but no dismiss?**
- Ensures patient eventually takes medicine or guardian is notified
- Prevents accidental dismissal leading to missed doses
- Balances patient flexibility with adherence goals

**Why Firebase?**
- Real-time synchronization out of the box
- Scalable cloud infrastructure
- Built-in authentication and security
- Serverless architecture reduces maintenance

## Impact and Benefits

**For Patients**:
- Clear visual guidance (LED) reduces confusion
- Maintains independence while ensuring safety
- Flexible snooze option respects their schedule

**For Guardians**:
- Peace of mind through real-time monitoring
- Reduced stress with early refill alerts
- Data-driven insights for better care decisions
- Immediate notification of missed doses

**For Healthcare**:
- Improved medication adherence rates
- Objective adherence data for healthcare providers
- Reduced hospitalizations due to missed medications
- Scalable solution for aging population

## Technology Stack Summary

- **Hardware**: ESP32 microcontroller, load cell weight sensors, LEDs, piezo buzzer
- **Mobile**: Android (Kotlin), MVVM architecture, Room database for offline
- **Backend**: Firebase (Firestore, Cloud Functions, Authentication, Cloud Messaging)
- **AI**: Time-series analysis for behavior patterns, linear regression for refill prediction
- **Communication**: WiFi (hardware to backend), HTTPS (app to backend), FCM (push notifications)
