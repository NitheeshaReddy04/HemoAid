# HemoAid

HemoAid is a real-time blood donor assistance platform designed to connect hospitals with nearby compatible donors during emergencies.

## Problem
Hospitals often face delays in locating compatible blood donors during critical situations, leading to loss of valuable time.

## Solution
HemoAid enables hospitals to send verified emergency blood requests to nearby donors. Donors can view request details, proof, and hospital location before accepting.

## Features
- Blood group–based donor registration
- Hospital-verified emergency requests
- Real-time notifications
- Proof and location visibility
- Privacy-first design using Google Cloud technologies

## Tech Stack (Planned)
- Frontend: Android / Web (Flutter / React)
- Backend: Firebase, Google Cloud
- Database: Firestore
- Maps & Location: Google Maps API

## MVP
- Figma Prototype: https://down-alert-88241203.figma.site
- Demo Video: https://drive.google.com/file/d/1yBG-p4v6kfokHYTnJJT-wVK2fZhk3lpC/view

> Note: This repository currently contains documentation only. Implementation will be added in future development.




Donar details:







# Homoaid - Blood Donation Emergency Application
## Technology Integration Guide

### 🎯 One-Line Pitch
**"Homoaid is a human-centric blood donation app that uses Google and Firebase technologies to connect donors and patients instantly during emergencies."**

---

## 🏗️ Architecture Overview

### Frontend
- **React** - Modern UI framework
- **Tailwind CSS** - Utility-first styling
- **Mobile-First Design** - Optimized for smartphones

### Backend Services (Google Cloud Platform)

#### 1. **Firebase Authentication**
```javascript
// Phone Number Authentication with OTP
import { getAuth, signInWithPhoneNumber, RecaptchaVerifier } from 'firebase/auth';

const auth = getAuth();
const appVerifier = new RecaptchaVerifier('recaptcha-container', {}, auth);

// Send OTP
const confirmationResult = await signInWithPhoneNumber(
  auth, 
  phoneNumber, // e.g., "+1234567890"
  appVerifier
);

// Verify OTP
const result = await confirmationResult.confirm(otpCode);
const user = result.user;
```

**Security Features:**
- Multi-factor authentication
- Phone number verification
- Session management
- Token-based authentication

---

#### 2. **Firebase Cloud Firestore**
```javascript
// Firestore Data Structure
import { getFirestore, collection, addDoc, updateDoc, doc } from 'firebase/firestore';

const db = getFirestore();

// Donor Registration
const donorData = {
  uid: user.uid,
  fullName: "John Doe",
  phoneNumber: "+1234567890",
  bloodGroup: "O+",
  age: 28,
  gender: "male",
  location: {
    address: "123 Medical Center, Downtown",
    coordinates: {
      latitude: 40.7128,
      longitude: -74.0060
    }
  },
  lastDonationDate: "2025-12-15",
  isAvailable: true,
  registeredAt: new Date(),
  fcmToken: "device-fcm-token" // For push notifications
};

// Save to Firestore
await addDoc(collection(db, "donors"), donorData);

// Blood Request Data Structure
const requestData = {
  requestId: "REQ-12345",
  patientName: "Emergency Patient",
  bloodGroup: "O+",
  unitsNeeded: 2,
  urgency: "critical", // critical | high | medium
  hospitalName: "City General Hospital",
  location: {
    address: "456 Hospital St",
    coordinates: { latitude: 40.7580, longitude: -73.9855 }
  },
  requestedAt: new Date(),
  status: "active", // active | fulfilled | cancelled
  matchedDonors: []
};

await addDoc(collection(db, "bloodRequests"), requestData);
```

**Collections:**
- `donors` - Registered blood donors
- `bloodRequests` - Emergency blood requests
- `donations` - Donation history
- `notifications` - Notification logs

---

#### 3. **Firebase Cloud Messaging (FCM)**
```javascript
// Push Notification for Emergency Requests
import { getMessaging, getToken, onMessage } from 'firebase/messaging';

const messaging = getMessaging();

// Get FCM Token (on donor registration)
const fcmToken = await getToken(messaging, {
  vapidKey: 'YOUR_VAPID_KEY'
});

// Save FCM token to user profile
await updateDoc(doc(db, "donors", userId), { fcmToken });

// Backend: Send emergency alert to nearby donors
// This runs on Google Cloud Functions
exports.sendBloodRequestAlert = functions.firestore
  .document('bloodRequests/{requestId}')
  .onCreate(async (snap, context) => {
    const request = snap.data();
    
    // Find matching donors (same blood group, available, nearby)
    const matchingDonors = await findNearbyDonors({
      bloodGroup: request.bloodGroup,
      location: request.location,
      radius: 10 // km
    });
    
    // Send push notification to all matching donors
    const notifications = matchingDonors.map(donor => ({
      token: donor.fcmToken,
      notification: {
        title: '🩸 Emergency Blood Donation Request',
        body: `${request.bloodGroup} needed urgently at ${request.hospitalName}`,
        icon: '/blood-icon.png'
      },
      data: {
        requestId: request.requestId,
        bloodGroup: request.bloodGroup,
        urgency: request.urgency,
        location: JSON.stringify(request.location)
      }
    }));
    
    await admin.messaging().sendEach(notifications);
  });
```

---

#### 4. **Google Maps API**
```javascript
// Location Services Integration
import { Loader } from '@googlemaps/js-api-loader';

const loader = new Loader({
  apiKey: 'YOUR_GOOGLE_MAPS_API_KEY',
  version: 'weekly',
  libraries: ['places', 'geometry']
});

// Auto-detect user location
async function detectLocation() {
  if (navigator.geolocation) {
    return new Promise((resolve, reject) => {
      navigator.geolocation.getCurrentPosition(
        position => {
          resolve({
            latitude: position.coords.latitude,
            longitude: position.coords.longitude
          });
        },
        error => reject(error)
      );
    });
  }
}

// Render map with draggable pin
const { Map } = await loader.importLibrary('maps');
const { Marker } = await loader.importLibrary('marker');

const map = new Map(document.getElementById('map'), {
  center: { lat: 40.7128, lng: -74.0060 },
  zoom: 15
});

const marker = new Marker({
  position: { lat: 40.7128, lng: -74.0060 },
  map: map,
  draggable: true,
  title: 'Your Location'
});

marker.addListener('dragend', (event) => {
  const location = {
    latitude: event.latLng.lat(),
    longitude: event.latLng.lng()
  };
  // Update form with new location
});

// Find nearby donors using Geometry library
function calculateDistance(coord1, coord2) {
  const R = 6371; // Earth's radius in km
  const dLat = (coord2.lat - coord1.lat) * Math.PI / 180;
  const dLon = (coord2.lng - coord1.lng) * Math.PI / 180;
  const a = 
    Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.cos(coord1.lat * Math.PI / 180) * Math.cos(coord2.lat * Math.PI / 180) *
    Math.sin(dLon/2) * Math.sin(dLon/2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return R * c;
}
```

---

#### 5. **Google Cloud Security**

**Security Measures:**

```javascript
// Firestore Security Rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Donors can only read/write their own data
    match /donors/{donorId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == donorId;
    }
    
    // Blood requests can be read by authenticated users
    // Only verified hospitals can create requests
    match /bloodRequests/{requestId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && 
        request.auth.token.hospitalVerified == true;
      allow update: if request.auth != null && 
        (request.auth.uid == resource.data.createdBy || 
         request.auth.token.admin == true);
    }
  }
}

// Firebase Cloud Functions for data validation
exports.validateDonorRegistration = functions.https.onCall((data, context) => {
  // Ensure user is authenticated
  if (!context.auth) {
    throw new functions.https.HttpsError(
      'unauthenticated',
      'User must be authenticated'
    );
  }
  
  // Validate data
  if (!data.bloodGroup || !isValidBloodGroup(data.bloodGroup)) {
    throw new functions.https.HttpsError(
      'invalid-argument',
      'Invalid blood group'
    );
  }
  
  // Verify phone number matches authenticated user
  if (data.phoneNumber !== context.auth.token.phone_number) {
    throw new functions.https.HttpsError(
      'permission-denied',
      'Phone number mismatch'
    );
  }
  
  return { success: true };
});

// Data encryption at rest (automatic with Firestore)
// SSL/TLS for data in transit (automatic with Firebase)
```

---

## 🔄 Complete User Flow

### Donor Registration Flow
1. User opens app → Donor Registration Screen
2. Enters phone number → Firebase sends OTP
3. User enters OTP → Firebase Authentication validates
4. User fills registration form (name, blood group, age, gender)
5. Google Maps auto-detects location → User confirms or adjusts pin
6. User selects last donation date → Calendar picker
7. User toggles availability status → Switch ON/OFF
8. Clicks "Register as Donor" → Data saved to Firestore
9. FCM token generated and saved → User subscribed to emergency alerts

### Emergency Blood Request Flow
1. Hospital creates emergency blood request → Saved to Firestore
2. Firestore onCreate trigger → Cloud Function executes
3. Function queries nearby available donors (blood group match, location radius)
4. FCM sends push notifications to all matching donors
5. Donors receive real-time alert → Can accept or decline
6. Accepting donor's location shown to hospital → Direct contact established
7. Donation completed → Status updated in Firestore

---

## 📊 Key Features

✅ **Real-time Matching** - Instant donor-patient connection  
✅ **Location-based Search** - Find nearest donors within radius  
✅ **Push Notifications** - Emergency alerts via FCM  
✅ **Secure Authentication** - Phone OTP verification  
✅ **Data Privacy** - Google Cloud security & encryption  
✅ **Scalability** - Google Cloud Platform infrastructure  
✅ **Offline Support** - Firestore offline persistence  
✅ **Analytics** - Firebase Analytics for insights  

---

## 🚀 Deployment

### Firebase Setup
```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login to Firebase
firebase login

# Initialize Firebase project
firebase init

# Deploy Cloud Functions
firebase deploy --only functions

# Deploy Hosting
firebase deploy --only hosting
```

### Environment Variables
```env
VITE_FIREBASE_API_KEY=your-api-key
VITE_FIREBASE_AUTH_DOMAIN=your-app.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-app.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abcdef
VITE_GOOGLE_MAPS_API_KEY=your-maps-api-key
```

---

## 🎤 Presentation Talking Points

### For Judges/Stakeholders:

1. **Problem Statement**
   - "Blood emergencies require immediate donor availability. Traditional systems are slow and inefficient."

2. **Solution**
   - "Homoaid connects donors and hospitals in real-time using Google Cloud and Firebase technologies."

3. **Technology Stack**
   - "Built on enterprise-grade Google Cloud Platform for security, scalability, and reliability."

4. **Key Differentiators**
   - "Real-time push notifications, location-based matching, and secure OTP authentication."

5. **Impact**
   - "Reduces blood procurement time from hours to minutes, potentially saving thousands of lives."

6. **Scalability**
   - "Firebase can handle millions of users with automatic scaling. No server management required."

7. **Security & Privacy**
   - "Medical data protected by Google Cloud Security, HIPAA-ready infrastructure, and encrypted communication."

---

## 🎨 Design Philosophy

**Medical + Humanitarian Theme**
- Primary color: Red (#dc2626) - Universal symbol of blood/healthcare
- White (#ffffff) - Cleanliness, trust, medical professionalism
- Soft gray (#f5f5f5) - Minimal, accessible background
- Rounded inputs (0.75rem radius) - Friendly, approachable
- Clear icons - Intuitive, accessible for all users
- High contrast - Readable in emergency situations

---

## 📱 Next Screens to Build

1. **Login / OTP Verification Screen**
   - Phone number input
   - OTP code entry
   - Resend OTP functionality

2. **Blood Request Screen**
   - Hospital interface
   - Urgency level selector
   - Units needed input
   - Submit emergency request

3. **Nearby Donors Map**
   - Google Maps with donor pins
   - Filter by blood group
   - Distance calculation
   - Direct contact buttons

4. **Donor Dashboard**
   - Donation history
   - Availability toggle
   - Upcoming requests
   - Profile management

---

## 📝 Notes

- This is a **concept design** showcasing UI/UX and technology integration
- Real implementation requires:
  - Valid Google Cloud Platform account
  - Firebase project setup
  - Google Maps API key with billing enabled
  - Healthcare compliance certifications (HIPAA, etc.)
  - Legal agreements for medical data handling

**⚠️ Disclaimer:** This application concept is for demonstration purposes. Real-world deployment of medical applications requires proper healthcare certifications, legal compliance, and data protection measures.

---

## 🩸 Mission

*"Helping humans help humans - because every drop counts."*

**Homoaid** - Making blood donation accessible, instant, and life-saving through technology.
//patient details
import { PatientDetailsForm } from "./components/PatientDetailsForm";
import { Toaster } from "./components/ui/sonner";

export default function App() {
  return (
    <>
      <PatientDetailsForm />
      <Toaster position="top-center" />
    </>
  );
}

