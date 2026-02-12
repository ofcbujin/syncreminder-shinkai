# SyncMinder – Cross-Device Reminder Sync App

> A Kotlin Multiplatform (KMP) reminder application built with Compose Multiplatform that synchronizes tasks, alarms, and notifications across all your devices in real time.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Feature Modules](#feature-modules)
- [Data Flow & Synchronization](#data-flow--synchronization)
- [Authentication](#authentication)
- [AI Voice Commands](#ai-voice-commands)
- [Notification System](#notification-system)
- [Screens & Navigation](#screens--navigation)
- [Getting Started](#getting-started)

---

## Project Overview

SyncMinder solves the problem of fragmented task management across multiple devices. When a user creates, completes, or modifies a reminder on one device, the change is instantly reflected on all connected devices — including scheduled local alarms and notifications. The app supports both Android and iOS through Kotlin Multiplatform with a shared Compose UI layer.

### Problem Statement

Users who own multiple devices (phone, tablet, work phone) often miss reminders because notifications only fire on the device where the reminder was created. SyncMinder ensures that when a task is marked as done on any device, the alarm is cancelled on **all** devices, and new reminders trigger notifications everywhere.

### Target Users

- Professionals managing tasks across work and personal devices
- Users who want cross-platform (Android + iOS) reminder synchronization
- Anyone who benefits from AI-powered natural language task creation via voice

---

## Key Features

### 1. Cross-Device Synchronization
- Real-time reminder sync via **Firebase Firestore**
- Silent push notifications via **Firebase Cloud Messaging (FCM)** to schedule/cancel alarms on all devices
- Device registration and management (add, remove, track synced devices)

### 2. AI-Powered Voice Commands
- Natural language task creation: speak a reminder, and the AI parses title, date, time, priority, and recurrence
- Powered by **Google Gemmma** with structured JSON output
- Automatic category matching and suggestion for new categories
- Animated voice command overlay with real-time transcript display

### 3. Smart Reminder Management
- Create reminders with title, notes, due date/time, priority (Low/Medium/High), and recurrence rules (RRULE standard)
- Personal Lists (categories) to organize reminders
- Mark reminders as complete with a satisfaction animation
- Snooze functionality with configurable default duration (5–60 minutes)

### 4. Multi-View Planner
- **Day View**: Timeline-based daily agenda with drag-to-reschedule
- **Week View**: Weekly grid with drag-and-drop reminder rescheduling
- **Month View**: Calendar with date filtering and task grouping by day
- Filter between Incomplete and Completed tasks in each view

### 5. Statistics Dashboard
- Completion score with percentage indicator
- Activity bar chart (weekly/monthly toggle)
- Category distribution donut chart
- Recently completed tasks section

### 6. Onboarding Experience
- 4-step animated onboarding flow using `HorizontalPager`
- Interactive demonstrations of key features (sync animation, voice command preview)
- Integrated login/signup on the final step with social auth options

### 7. Subscription & Monetization
- Free plan: single-device usage
- Pro plan: unlimited device sync
- Paywall screen with subscription management

### 8. Theming & Accessibility
- Full Material 3 Expressive design system
- Light and Dark mode support with dynamic color adaptation
- High-contrast text colors for accessibility

---

## Architecture

SyncMinder follows **Clean Architecture** principles with a modular feature-based structure:

```
┌─────────────────────────────────────────────────┐
│                  Presentation                     │
│  (Composables, ViewModels, UI State)             │
├─────────────────────────────────────────────────┤
│                  Domain Layer                     │
│  (Use Cases, Repository Interfaces, Models)      │
├─────────────────────────────────────────────────┤
│                  Data Layer                       │
│  (Repository Impls, Firestore, Room DB, API)     │
├─────────────────────────────────────────────────┤
│              Platform-Specific                    │
│  (Android: BroadcastReceiver, AlarmManager)       │
│  (iOS: UNUserNotificationCenter)                 │
└─────────────────────────────────────────────────┘
```

### Design Patterns

| Pattern | Usage |
|---|---|
| **Repository** | Abstracts data sources (Firestore, Room, API) behind interfaces |
| **Use Cases** | Single-responsibility business logic (e.g., `ProcessVoiceCommandUseCase`) |
| **Dependency Injection** | Koin for multiplatform DI |
| **MVI-like Events** | UI events dispatched to ViewModels via sealed event classes |
| **Expect/Actual** | Platform-specific implementations for notifications, auth, and platform APIs |

---

## Tech Stack

| Category | Technology |
|---|---|
| **Language** | Kotlin 2.x (Multiplatform) |
| **UI Framework** | Compose Multiplatform (Material 3 Expressive) |
| **Navigation** | AndroidX Navigation 3 |
| **Local Database** | Room (KMP) |
| **Cloud Database** | Firebase Firestore |
| **Authentication** | Firebase Auth (Google Sign-In, Apple Sign-In) |
| **Push Notifications** | Firebase Cloud Messaging (FCM) |
| **Subscription & Billing** | RevenueCat (Purchases KMP) |
| **Cloud Functions** | Firebase Cloud Functions (Node.js) |
| **AI/ML** | Google Gemma 3n (voice command parsing) |
| **Dependency Injection** | Koin |
| **Serialization** | Kotlinx Serialization |
| **Async** | Kotlin Coroutines + Flow |
| **Build System** | Gradle (Kotlin DSL) |

---

## Project Structure

```
SyncMinder/
├── composeApp/
│   └── src/
│       ├── commonMain/          # Shared KMP code (95%+ of the app)
│       │   └── kotlin/com/shinkaiapps/syncminder/
│       │       ├── App.kt                  # Root composable & navigation graph
│       │       ├── Koin.kt                 # DI setup
│       │       ├── core/                   # Shared infrastructure
│       │       │   ├── data/               # Repository implementations
│       │       │   │   ├── repository/     # AuthRepositoryImpl, SettingsRepositoryImpl, etc.
│       │       │   │   └── local/          # Room DAOs and database
│       │       │   ├── domain/             # Interfaces, models, use cases
│       │       │   │   ├── model/          # User, Device, UserSettings
│       │       │   │   └── repository/     # AuthRepository, SettingsRepository, NotificationScheduler
│       │       │   ├── ui/                 # Shared UI (theme, components)
│       │       │   │   ├── theme/          # Color.kt, Theme.kt, Typography, Shapes, Dimens
│       │       │   │   └── components/     # Reusable composables
│       │       │   ├── platform/           # expect declarations
│       │       │   ├── lifecycle/          # AppLifecycleObserver
│       │       │   ├── analytics/          # Analytics tracking
│       │       ├── di/                     # Koin modules per feature
│       │       ├── features/
│       │       │   ├── auth/               # Authentication (Login/Signup)
│       │       │   ├── onboarding/         # 4-step onboarding flow
│       │       │   ├── reminders/          # Core reminder CRUD & UI
│       │       │   ├── planner/            # Day/Week/Month planner views
│       │       │   ├── stats/              # Statistics & analytics dashboard
│       │       │   ├── settings/           # App settings & device management
│       │       │   ├── navigation/         # AppNavigation composable
│       │       │   └── usage/              # Usage tracking & limits
│       │       └── presentation/           # AppViewModel (root state)
│       ├── androidMain/         # Android-specific implementations
│       │   └── kotlin/          # NotificationScheduler, SocialAuthButtons, etc.
│       └── iosMain/             # iOS-specific implementations
│           └── kotlin/          # NotificationScheduler, Platform, etc.
├── iosApp/                      # iOS app entry point (SwiftUI wrapper)
├── shinkaibilling/              # Internal billing library (RevenueCat)
├── functions/                   # Firebase Cloud Functions
└── build.gradle.kts
```

---

## Feature Modules

### `features/reminders/` — Core Reminder Management

The heart of the application. Handles CRUD operations for reminders with full offline support.

**Domain Model:**
```kotlin
data class Reminder(
    val id: String,
    val userId: String,
    val categoryId: String?,     // Link to Personal Lists
    val title: String,
    val notes: String?,
    val dueDate: String,         // Epoch seconds as String
    val isCompleted: Boolean,
    val priority: Int,           // 1: Low, 2: Medium, 3: High
    val recurrenceRule: String?, // RRULE standard (e.g., "FREQ=DAILY")
    val createdAt: String,
    val snoozeMinutes: Int
)
```

**Key Use Cases:**
- `AddReminderUseCase` — Creates reminder locally and syncs to Firestore
- `CompleteReminderUseCase` — Marks done, triggers silent push to cancel alarms on other devices
- `ProcessVoiceCommandUseCase` — Parses natural language via Gemma 3n into structured reminder data

**UI Components:**
- `HomeScreen` — Dashboard with today's tasks, scheduled reminders, personal lists
- `NewReminderScreen` — Full reminder editor with date/time pickers, priority, recurrence
- `AllRemindersScreen` — Filtered list view grouped by date
- `VoiceCommandOverlay` — Animated voice input with waveform visualization
- `UrgentReminderCard` — Time-sensitive task display with pager

### `features/planner/` — Multi-View Planner

Three different views for planning and managing tasks:

- **DailyTimeline** — Hour-by-hour agenda with time-slot tapping
- **WeeklyPlannerGrid** — 7-day grid with drag-and-drop rescheduling
- **MonthlyCalendar** — Calendar with task indicators and day filtering

Includes filter chips to toggle between Incomplete and Completed tasks.

### `features/stats/` — Statistics Dashboard

Visual analytics for task completion habits:

- **ScoreCircleCard** — Animated circular progress showing completion rate
- **ActivityBarChart** — Weekly/monthly task completion bar chart
- **CategoryDonutChart** — Distribution of tasks across personal lists
- **RecentlyCompletedSection** — Latest completed tasks

### `features/auth/` — Authentication

- Google Sign-In (Android + iOS)
- Apple Sign-In (iOS native, Android web-based)
- Guest mode with local-only storage
- Session monitoring with automatic token refresh
- Remote logout via silent push

### `features/settings/` — Settings & Device Management

- Subscription plan management (Free/Pro)
- Custom snooze default (5–60 minutes slider)
- Persistent alerts toggle
- Smart reminders toggle
- Device list with sync status and remote unsync capability

### `features/onboarding/` — Onboarding Flow

4-step animated introduction:

1. **Sync Demo** — Animated device-to-device sync visualization
2. **Feature Overview** — Key capabilities with scroll support
3. **Voice Command Preview** — Animated mic pulse with transcript demo
4. **Login/Signup** — Integrated social auth with skip option

### `shinkaibilling` — Subscription Architecture

The app uses a dedicated internal multiplatform library for subscription management, abstracting the **RevenueCat** integration.

- **Technology**: Built on [Purchases KMP](https://github.com/RevenueCat/purchases-kmp) (Version `2.2.15+17.25.0`).
- **Identity Sync**: User IDs are synchronized between Firebase Auth and RevenueCat via `BillingIdentityService.logIn(appUserId)`.
- **Entitlement Verification**: Access to "Pro" features (like multi-device sync) is controlled by the `SyncMinder Pro` entitlement ID.
- **Service Layer**: `BillingIdentityServiceImpl` provides a unified reactive flow (`isPro()`) to observe subscription status throughout the app.

---

## Data Flow & Synchronization

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   Device A   │────▶│   Firestore  │────▶│ Cloud Functions  │
│  (Android)   │◀────│   (Source    │     │  (Trigger on     │
│              │     │   of Truth)  │     │   write/update)  │
└──────────────┘     └──────────────┘     └────────┬─────────┘
                           ▲                       │
                           │                       ▼
┌──────────────┐           │              ┌──────────────────┐
│   Device B   │───────────┘              │    FCM Silent    │
│    (iOS)     │◀─────────────────────────│   Push to all    │
│              │                          │   other devices  │
└──────────────┘                          └──────────────────┘
```

1. **Reminder Created** → Saved to Room DB (local) + Firestore (cloud)
2. **Firestore Trigger** → Cloud Function detects new document
3. **FCM Data Message** → Sent to all other registered devices
4. **Silent Push Received** → Local alarm scheduled on each device
5. **Reminder Completed** → Status updated in Firestore → Cloud Function sends cancellation push → Alarms cancelled on all devices

### Offline Support

- Room database serves as local cache
- Changes queued when offline, synced when connectivity returns
- Firestore's built-in offline persistence ensures data consistency

---

## Authentication

### Auth Flow

```
Guest Mode ──▶ Google/Apple Sign-In ──▶ Authenticated User
     │                                        │
     │  (local-only storage)                  │  (Firestore sync enabled)
     │                                        │  (FCM token registered)
     │                                        │  (Device added to user profile)
     ▼                                        ▼
  Single Device                         Multi-Device Sync
```

### Session Management

- `startSessionMonitoring()` watches the Firestore sessions collection
- If a session is invalidated remotely (e.g., device removed from settings), a high-priority silent push forces local logout
- `refreshSession()` handles token renewal

---

## AI Voice Commands

The `ProcessVoiceCommandUseCase` sends voice transcripts to the **Google Gemma 3n** with a structured prompt that extracts:

| Field | Description |
|---|---|
| `title` | Reminder title parsed from natural speech |
| `date` | Due date in ISO format |
| `time` | Due time in HH:mm format |
| `priority` | 1 (Low), 2 (Medium), 3 (High) |
| `recurrenceRule` | RRULE string for recurring tasks |
| `category` | Matched or suggested category name |

**Example:**  
*"Remind me to call the dentist tomorrow at 3pm, high priority"*  
→ Creates a reminder with title "Call the dentist", tomorrow's date, 15:00, priority 3.

---

## Notification System

### Android
- `AlarmManager` for exact alarm scheduling
- `BroadcastReceiver` for alarm triggers and silent push handling
- Foreground notification channel for persistent reminders
- Snooze action in notification

### iOS
- `UNUserNotificationCenter` for local notification scheduling
- Background push handling for silent sync messages
- Custom notification categories with snooze/complete actions

---

## Screens & Navigation

| Route | Screen | Bottom Bar | Description |
|---|---|---|---|
| `Home` | HomeScreen | ✅ | Dashboard with today's tasks, scheduled reminders, personal lists |
| `Planner` | PlannerScreen | ✅ | Day/Week/Month planning views |
| `Stats` | StatsScreen | ✅ | Completion analytics and charts |
| `Settings` | SettingsScreen | ✅ | Preferences, devices, subscription |
| `NewReminder` | NewReminderScreen | ❌ | Create/edit reminder (full editor) |
| `AllReminders` | AllRemindersScreen | ❌ | Filtered reminder list by category |
| `Login` | LoginScreen | ❌ | Google/Apple sign-in |
| `Onboarding` | OnboardingScreen | ❌ | 4-step animated onboarding |
| `Paywall` | PaywallScreen | ❌ | Subscription upgrade |

Navigation uses **AndroidX Navigation 3** with a serializable `Route` sealed interface and `NavBackStack` management.


## License

This project is proprietary software developed by ShinKai Apps. All rights reserved.

