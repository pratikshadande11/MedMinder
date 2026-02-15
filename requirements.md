# Requirements Document

## Introduction

MedMinder is an AI-powered smart medication adherence system that combines a hardware smart pill box with a mobile application to ensure patients take the correct medications at the right times. The system uses compartment-based design with LEDs and weight sensors, provides timely reminders, enables guardian supervision, and offers intelligent refill alerts through AI-driven behavior analysis and consumption prediction.

## Glossary

- **Smart_Pill_Box**: Physical hardware device with multiple compartments, each containing one specific medicine, equipped with LEDs and weight sensors
- **Mobile_App**: Android application used by patients and guardians to manage medications and receive notifications
- **Patient**: End user who takes medications using the Smart_Pill_Box and receives reminders
- **Guardian**: Supervisor who monitors patient adherence, receives alerts, and makes decisions on schedule adjustments
- **Compartment**: Individual storage unit within the Smart_Pill_Box that holds one specific medicine
- **Medication_Schedule**: Predefined times when specific medications should be taken
- **Adherence_Data**: Historical record of medication intake including timing, delays, and missed doses
- **Behavior_Pattern**: AI-analyzed insights derived from Adherence_Data showing patient habits and difficulties
- **Refill_Alert**: Notification sent when medicine quantity in a compartment is predicted to run low
- **Backend_System**: Firebase Firestore database managing all system data
- **Intake_Window**: 2-minute period after each alarm trigger during which patient should take medicine
- **Alarm_Cycle**: Complete sequence of up to 3 alarm attempts with 2-minute gaps between attempts

## Requirements

### Requirement 1: Smart Pill Box Compartment Management

**User Story:** As a patient, I want each compartment to hold one specific medicine with clear identification, so that I can easily take the correct medication without confusion.

#### Acceptance Criteria

1. THE Smart_Pill_Box SHALL contain multiple compartments where each compartment stores one specific type of medicine
2. WHEN a medication time arrives, THE Smart_Pill_Box SHALL illuminate the LED on the correct compartment
3. THE Smart_Pill_Box SHALL maintain a mapping between each compartment and its assigned medicine in the Backend_System
4. WHEN a compartment is assigned a medicine, THE Backend_System SHALL store the medicine name, dosage, and schedule information

### Requirement 2: Medication Reminder and Alarm System

**User Story:** As a patient, I want to receive clear alarms and visual indicators when it's time to take my medicine, so that I don't miss doses.

#### Acceptance Criteria

1. WHEN a scheduled medication time arrives, THE Smart_Pill_Box SHALL activate an audible alarm
2. WHEN a scheduled medication time arrives, THE Smart_Pill_Box SHALL illuminate the LED on the corresponding compartment
3. WHEN a scheduled medication time arrives, THE Mobile_App SHALL send a notification to the Patient
4. WHEN the Patient does not take medicine within the Intake_Window, THE Smart_Pill_Box SHALL wait 2 minutes and then repeat the alarm
5. THE Smart_Pill_Box SHALL trigger the alarm a maximum of 3 times per scheduled dose
6. WHEN medicine is taken during any alarm attempt, THE Smart_Pill_Box SHALL immediately stop all further alarms for that dose
7. WHEN all 3 alarm attempts complete without medicine intake, THE Smart_Pill_Box SHALL stop alarming and mark the dose as missed
8. THE Intake_Window SHALL be 2 minutes in duration for each alarm attempt

### Requirement 3: Medication Intake Detection

**User Story:** As a patient, I want the system to detect when I've taken my medicine, so that alarms stop and my adherence is tracked.

#### Acceptance Criteria

1. WHEN a Patient removes medicine from a compartment, THE Smart_Pill_Box SHALL detect the weight change using the weight sensor
2. WHEN weight reduction is detected during an active alarm, THE Smart_Pill_Box SHALL deactivate the alarm and LED
3. WHEN medicine intake is detected, THE Backend_System SHALL record the intake event with timestamp
4. WHEN medicine intake is detected, THE Mobile_App SHALL update the Patient's adherence status
5. WHEN all alarm attempts for a scheduled dose complete without detecting weight reduction, THE Backend_System SHALL record a missed dose event

### Requirement 4: Guardian Notification System

**User Story:** As a guardian, I want to be notified when my patient misses medications, so that I can intervene and provide support.

#### Acceptance Criteria

1. WHEN a Patient misses a dose after alarm repetitions, THE Mobile_App SHALL send a notification to the Guardian
2. WHEN a Guardian notification is sent, THE Backend_System SHALL include the medicine name, scheduled time, and current time
3. THE Mobile_App SHALL display the Patient's current adherence status to the Guardian
4. WHEN multiple doses are missed in a day, THE Mobile_App SHALL send a summary notification to the Guardian

### Requirement 5: AI Behavior Pattern Analysis

**User Story:** As a guardian, I want to see clear patterns in my patient's medication behavior, so that I can make informed decisions about schedule adjustments.

#### Acceptance Criteria

1. THE Backend_System SHALL analyze Adherence_Data to identify Behavior_Patterns including delays, missed doses, and time-of-day difficulties
2. WHEN Behavior_Patterns are identified, THE Mobile_App SHALL present them to the Guardian in clear, explainable format
3. THE Mobile_App SHALL display pattern insights including frequency of delays, common missed times, and adherence trends
4. WHEN viewing patterns, THE Guardian SHALL be able to decide whether to adjust medication timings
5. THE Backend_System SHALL NOT automatically adjust medication schedules without Guardian approval
6. WHEN a Guardian approves a schedule adjustment, THE Backend_System SHALL update the Medication_Schedule

### Requirement 6: Intelligent Refill Prediction

**User Story:** As a guardian, I want to receive early alerts when medicines are running low, so that I can refill them before they run out.

#### Acceptance Criteria

1. THE Smart_Pill_Box SHALL continuously monitor weight in each compartment using weight sensors
2. THE Backend_System SHALL track weight reduction rate for each medicine over time
3. THE Backend_System SHALL predict when a medicine will run low based on consumption rate and current weight
4. WHEN a medicine is predicted to run low, THE Mobile_App SHALL send a Refill_Alert to the Guardian
5. THE Refill_Alert SHALL include the medicine name, estimated days remaining, and compartment identifier
6. THE Backend_System SHALL send Refill_Alerts with sufficient advance notice to prevent stock-outs

### Requirement 7: Mobile Application User Interface

**User Story:** As a patient, I want an intuitive mobile app interface, so that I can easily view my medication schedule and adherence status.

#### Acceptance Criteria

1. THE Mobile_App SHALL display the current day's medication schedule with times and medicine names
2. THE Mobile_App SHALL show visual indicators for taken, missed, and upcoming medications
3. WHEN a Patient opens the Mobile_App, THE Mobile_App SHALL display the most recent adherence summary
4. THE Mobile_App SHALL provide separate interfaces for Patient and Guardian roles
5. WHEN a Guardian logs in, THE Mobile_App SHALL display all supervised patients and their adherence data

### Requirement 8: Data Synchronization and Storage

**User Story:** As a system administrator, I want reliable data synchronization between the smart pill box, mobile app, and backend, so that all components have consistent information.

#### Acceptance Criteria

1. WHEN the Smart_Pill_Box detects an intake event, THE Backend_System SHALL receive and store the data within 5 seconds
2. WHEN the Backend_System updates Adherence_Data, THE Mobile_App SHALL synchronize the changes within 10 seconds
3. THE Backend_System SHALL store all medication schedules, adherence records, and behavior patterns in Firebase Firestore
4. WHEN network connectivity is lost, THE Smart_Pill_Box SHALL store intake events locally
5. WHEN network connectivity is restored, THE Smart_Pill_Box SHALL upload all locally stored events to the Backend_System
6. THE Backend_System SHALL maintain data integrity during concurrent updates from multiple devices

### Requirement 9: Medication Schedule Configuration

**User Story:** As a guardian, I want to configure medication schedules for my patient, so that the system knows when to trigger alarms.

#### Acceptance Criteria

1. THE Mobile_App SHALL allow a Guardian to create a new medication entry with name, dosage, and timing
2. THE Mobile_App SHALL allow a Guardian to assign a medication to a specific compartment
3. WHEN a Guardian creates or modifies a schedule, THE Backend_System SHALL validate the schedule for conflicts
4. THE Mobile_App SHALL allow a Guardian to set multiple daily times for a single medication
5. WHEN a schedule is saved, THE Backend_System SHALL propagate the schedule to the Smart_Pill_Box within 10 seconds
6. THE Mobile_App SHALL allow a Guardian to temporarily pause or resume medication schedules

### Requirement 10: System Security and Privacy

**User Story:** As a patient, I want my health data to be secure and private, so that my medical information is protected.

#### Acceptance Criteria

1. THE Mobile_App SHALL require authentication before allowing access to patient data
2. THE Backend_System SHALL encrypt all health data at rest and in transit
3. THE Mobile_App SHALL implement role-based access control separating Patient and Guardian permissions
4. WHEN a Guardian is assigned to a Patient, THE Backend_System SHALL require Patient consent
5. THE Backend_System SHALL maintain audit logs of all data access and modifications
6. THE Mobile_App SHALL allow a Patient to revoke Guardian access at any time

### Requirement 11: Weight Sensor Calibration and Accuracy

**User Story:** As a system administrator, I want accurate weight measurements from compartment sensors, so that intake detection and refill predictions are reliable.

#### Acceptance Criteria

1. THE Smart_Pill_Box SHALL calibrate weight sensors during initial setup
2. WHEN a compartment is refilled, THE Mobile_App SHALL allow recording the new full weight
3. THE Smart_Pill_Box SHALL detect weight changes with accuracy of at least 0.5 grams
4. WHEN weight sensor readings are inconsistent, THE Smart_Pill_Box SHALL flag the compartment for maintenance
5. THE Backend_System SHALL use weight thresholds to distinguish between medicine removal and environmental factors

### Requirement 12: Alarm Repetition and Escalation

**User Story:** As a patient, I want alarms to repeat if I don't respond immediately, so that I don't forget to take my medicine even if I'm distracted.

#### Acceptance Criteria

1. WHEN medicine is not taken within the Intake_Window, THE Smart_Pill_Box SHALL wait 2 minutes and then trigger the next alarm attempt
2. THE Smart_Pill_Box SHALL trigger the alarm a maximum of 3 times per scheduled dose
3. THE Smart_Pill_Box SHALL maintain a 2-minute gap between consecutive alarm attempts
4. WHEN medicine intake is detected via weight sensor during any alarm attempt, THE Smart_Pill_Box SHALL immediately stop the alarm and cancel all remaining alarm attempts for that dose
5. THE Smart_Pill_Box SHALL stop the alarm ONLY when weight sensor detects medicine removal
6. THE Mobile_App SHALL allow a Patient to snooze an active alarm
7. THE Mobile_App SHALL allow a maximum of 3 snooze actions per scheduled dose
8. WHEN a Patient snoozes an alarm, THE Smart_Pill_Box SHALL pause the alarm for 2 minutes and then resume
9. WHEN a Patient snoozes an alarm, THE compartment LED SHALL remain continuously illuminated during the snooze period
10. THE compartment LED SHALL remain continuously ON during all snooze intervals for a total duration of up to 6 minutes
11. THE compartment LED SHALL turn OFF ONLY when the weight sensor detects medicine removal or when the system escalates to Guardian notification
12. THE Mobile_App SHALL NOT provide a dismiss or turn-off option for alarms
13. WHEN all 3 alarm attempts complete without weight sensor detecting intake, THE Smart_Pill_Box SHALL escalate to Guardian notification


