# UniRide

Android mobile application for coordinating shared rides among university students. Implements real-time communication between drivers and passengers, with location tracking via Google Maps and push notifications.

## Purpose

UniRide enables university students to:
- **Passengers**: Search and request seats on available trips, receive notifications about request status.
- **Drivers**: Publish trips with defined routes, manage passenger requests, and start/end trips with real-time tracking.

The application uses a role-based architecture that allows the same user to switch between passenger mode and driver mode within the same session.

---

## Architecture and Patterns

### Model-View-ViewModel (MVVM)

The application implements the Model-View-ViewModel pattern using the following components:

```
com.example.uniride/
├── domain/
│   ├── adapter/         # RecyclerView adapters
│   └── model/           # Data classes (Trip, User, Vehicle, etc.)
├── messaging/           # FCM services and notification sending
├── ui/
│   ├── auth/            # Authentication flow
│   ├── driver/          # Driver fragments and activities
│   ├── passenger/       # Passenger fragments and activities
│   └── shared/          # Shared components
└── utility/             # Utility classes
```

### Data Flow

1. **ViewModels** expose `StateFlow<T>` for reactive observation from views.
2. **Fragments** use `lifecycleScope.launch` with `collectLatest` to observe state changes.
3. **Navigation Component** manages navigation with Safe Args for type-safe parameter passing.
4. **Dagger Hilt** provides application-level dependency injection.

### Role-Based Navigation

The application uses separate navigation graphs:
- `passenger_nav_graph.xml`: Passenger flows
- `driver_nav_graph.xml`: Driver flows

Role transitions are performed dynamically via `MainViewModel.switchToPassenger()` and `MainViewModel.switchToDriver()`.

---

## Technology Stack

| Category | Technology | Version |
|----------|------------|---------|
| Language | Kotlin | 2.1.0 |
| Minimum SDK | Android API | 26 (Oreo) |
| Target SDK | Android API | 35 |
| Build System | Gradle Kotlin DSL | 8.8.0 |
| Dependency Injection | Dagger Hilt | 2.48 |
| Navigation | Navigation Component + Safe Args | 2.7.1 |
| UI Binding | View Binding | - |

### Main Dependencies

```kotlin
// Firebase
firebase-auth: 23.2.1
firebase-firestore: 25.1.4
firebase-database-ktx: 21.0.0
firebase-storage: 21.0.2
firebase-messaging-ktx: 24.1.1

// Google Maps
play-services-maps: 19.1.0
play-services-location: 21.3.0

// Network
ktor-client-android: 3.1.2
okhttp: 4.12.0

// Security
security-crypto: 1.1.0-alpha06
biometric: 1.2.0-alpha05
```

---

## Technical Integrations

### Firebase

#### Authentication
- Sign-in and registration via `FirebaseAuth`.
- FCM token management linked to user documents in Firestore.

#### Firestore
Collections identified in the source code:
- `users`: User profiles with FCM tokens
- `trips`: Published trips with status (PENDING, ACTIVE, FINISHED)
- `locations`: Origin/destination coordinates
- `stops`: Intermediate stops on routes

#### Realtime Database
- Driver location synchronization during active trips.
- The `updateDriverLocationInFirestore()` function updates coordinates on each location change.

#### Cloud Functions
`sendCustomNotification` function deployed in Node.js (`functions/index.js`):

```javascript
// Supported notification types:
- solicitud_cupo   // New passenger request
- aceptado         // Request accepted
- rechazado        // Request rejected
- mensaje          // New chat message
- viaje_iniciado   // Driver started the trip
- viaje_terminado  // Trip finished
- viaje_cancelado  // Trip cancelled
```

The Android client invokes this function via `NotificationSender.enviar()` using OkHttp.

#### Cloud Messaging (FCM)
- `MyFirebaseMessagingService`: Receives notifications and generates PendingIntents with deep linking to the corresponding fragment.
- Automatic token refresh in `onNewToken()`.



### Google Maps Platform

#### Maps SDK
- `OnMapReadyCallback` implementation in `DriverHomeFragment`.
- API Key configuration via secrets-gradle-plugin in `local.properties`.

#### Directions API
Direct REST API consumption for route retrieval:
```kotlin
fun getDirectionsData(origin: LatLng, destination: LatLng, waypoints: List<LatLng>)
```
- Polyline construction via `decodePoly(encoded: String)`.
- Dynamic route updates based on driver's current location.

#### Location Services
- `FusedLocationProviderClient` for high-accuracy location retrieval.
- `LocationCallback` updates position and verifies proximity to destination via `checkDistanceToDestination()`.

---

## Sensor Implementation

### Accelerometer (Motion Detection)

Implemented in `MainActivity.kt`:

```kotlin
val acceleration = sqrt(x*x + y*y + z*z) - SensorManager.GRAVITY_EARTH
if (acceleration > 12) {
    // Passenger: Navigate to trip search
    // Driver: Navigate to chats
}
```

- Activation threshold: 12 m/s² above Earth's gravity.
- 10ms debounce between detections.
- Context-aware behavior based on active role.

### Light Sensor

Implemented in `DriverHomeFragment.kt`:

```kotlin
fun updateMapStyle(lightValue: Float) {
    // Adjusts map style based on ambient lighting
}
```

- Automatically switches between light/dark map styles.
- Uses `Sensor.TYPE_LIGHT` with `SensorManager.SENSOR_DELAY_UI`.

---

## Demo

<!-- Insert emulator synchronization GIF here -->
![Real-time synchronization](./docs/demo.gif)

*Real-time synchronization demonstration across multiple emulators: driver location communication and push notifications.*

---

## Development Environment Setup

### Prerequisites

- Android Studio Ladybug (2024.x) or higher
- JDK 11
- Firebase account with configured project
- Google Maps Platform API Key (Maps SDK + Directions API)

### Configuration Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/juliancramos/UniRide.git
   cd uniRide
   ```

2. **Configure credentials**
   
   Create `local.properties` with:
   ```properties
   MAPS_API_KEY=<your-google-maps-api-key>
   ```

3. **Configure Firebase**
   - Download `google-services.json` from Firebase Console
   - Place in `app/google-services.json`
   - Enable: Authentication, Firestore, Realtime Database, Cloud Messaging, Storage

4. **Deploy Cloud Functions**
   ```bash
   cd functions
   npm install
   firebase deploy --only functions
   ```

5. **Build and run**
   ```bash
   ./gradlew assembleDebug
   ```

### Required Permissions

The `AndroidManifest.xml` file declares the following permissions:
- `INTERNET`: Network communication
- `ACCESS_FINE_LOCATION`: High-precision GPS
- `ACCESS_COARSE_LOCATION`: Approximate location
- `RECORD_AUDIO`: Audio functionality (voice messages)
- `POST_NOTIFICATIONS`: Push notifications (Android 13+)

---

## Additional Documentation

- [Architecture Diagram](./docs/Diagrama%20de%20arquitectura.pdf)
- [Database Diagram](./docs/Diagrama%20de%20base%20de%20datos.pdf)
- [Use Case Diagram](./docs/Diagrama%20de%20casos%20de%20uso%20UniRide.pdf)

---

## Development Team

| Member              | Username       | Role        |
|:------------------:|:-------------:|:----------:|
| Julian Ramos        | juliancramos  | Development|
| Samuel Osorio       | SamuOsorio    | Development|
| Sergio Ortiz  | SergioOrtiz145| Development|


