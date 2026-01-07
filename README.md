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


Login:










{
  "name": "@figma/my-make-file",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "build": "vite build"
  },
  "dependencies": {
    "@emotion/react": "11.14.0",
    "@emotion/styled": "11.14.1",
    "@mui/icons-material": "7.3.5",
    "@mui/material": "7.3.5",
    "@popperjs/core": "2.11.8",
    "@radix-ui/react-accordion": "1.2.3",
    "@radix-ui/react-alert-dialog": "1.1.6",
    "@radix-ui/react-aspect-ratio": "1.1.2",
    "@radix-ui/react-avatar": "1.1.3",
    "@radix-ui/react-checkbox": "1.1.4",
    "@radix-ui/react-collapsible": "1.1.3",
    "@radix-ui/react-context-menu": "2.2.6",
    "@radix-ui/react-dialog": "1.1.6",
    "@radix-ui/react-dropdown-menu": "2.1.6",
    "@radix-ui/react-hover-card": "1.1.6",
    "@radix-ui/react-label": "2.1.2",
    "@radix-ui/react-menubar": "1.1.6",
    "@radix-ui/react-navigation-menu": "1.2.5",
    "@radix-ui/react-popover": "1.1.6",
    "@radix-ui/react-progress": "1.1.2",
    "@radix-ui/react-radio-group": "1.2.3",
    "@radix-ui/react-scroll-area": "1.2.3",
    "@radix-ui/react-select": "2.1.6",
    "@radix-ui/react-separator": "1.1.2",
    "@radix-ui/react-slider": "1.2.3",
    "@radix-ui/react-slot": "1.1.2",
    "@radix-ui/react-switch": "1.1.3",
    "@radix-ui/react-tabs": "1.1.3",
    "@radix-ui/react-toggle-group": "1.1.2",
    "@radix-ui/react-toggle": "1.1.2",
    "@radix-ui/react-tooltip": "1.1.8",
    "class-variance-authority": "0.7.1",
    "clsx": "2.1.1",
    "cmdk": "1.1.1",
    "date-fns": "3.6.0",
    "embla-carousel-react": "8.6.0",
    "input-otp": "1.4.2",
    "lucide-react": "0.487.0",
    "motion": "12.23.24",
    "next-themes": "0.4.6",
    "react-day-picker": "8.10.1",
    "react-dnd": "16.0.1",
    "react-dnd-html5-backend": "16.0.1",
    "react-hook-form": "7.55.0",
    "react-popper": "2.3.0",
    "react-resizable-panels": "2.1.7",
    "react-responsive-masonry": "2.7.1",
    "react-slick": "0.31.0",
    "recharts": "2.15.2",
    "sonner": "2.0.3",
    "tailwind-merge": "3.2.0",
    "tw-animate-css": "1.3.8",
    "vaul": "1.1.2"
  },
  "devDependencies": {
    "@tailwindcss/vite": "4.1.12",
    "@vitejs/plugin-react": "4.7.0",
    "tailwindcss": "4.1.12",
    "vite": "6.3.5"
  },
  "peerDependencies": {
    "react": "18.3.1",
    "react-dom": "18.3.1"
  },
  "peerDependenciesMeta": {
    "react": {
      "optional": true
    },
    "react-dom": {
      "optional": true
    }
  },
  "pnpm": {
    "overrides": {
      "vite": "6.3.5"
    }
  }

  ///////////////////
  ///patient details code
  import { useState } from "react";
import { Droplet, MapPin, Phone, Hospital, User, AlertCircle, FileText, X } from "lucide-react";
import { Input } from "./ui/input";
import { Label } from "./ui/label";
import { Button } from "./ui/button";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "./ui/select";
import { RadioGroup, RadioGroupItem } from "./ui/radio-group";
import { toast } from "sonner";

interface PatientFormData {
  patientName: string;
  bloodGroup: string;
  requiredUnits: string;
  urgencyLevel: string;
  hospitalName: string;
  location: string;
  contactNumber: string;
}

export function PatientDetailsForm() {
  const [formData, setFormData] = useState<PatientFormData>({
    patientName: "",
    bloodGroup: "",
    requiredUnits: "",
    urgencyLevel: "normal",
    hospitalName: "",
    location: "",
    contactNumber: "",
  });

  const [uploadedFiles, setUploadedFiles] = useState<File[]>([]);

  const bloodGroups = ["A+", "A-", "B+", "B-", "O+", "O-", "AB+", "AB-"];

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // Basic validation
    if (!formData.patientName || !formData.bloodGroup || !formData.requiredUnits || 
        !formData.hospitalName || !formData.location || !formData.contactNumber) {
      toast.error("Please fill in all required fields");
      return;
    }

    // Check if documents are uploaded
    if (uploadedFiles.length === 0) {
      toast.error("Please upload at least one medical document");
      return;
    }

    // Show success message
    if (formData.urgencyLevel === "emergency") {
      toast.success("Emergency blood request submitted successfully! We'll notify donors immediately.", {
        duration: 5000,
      });
    } else {
      toast.success("Blood request submitted successfully!", {
        duration: 4000,
      });
    }

    // Log form data (in real app, this would be sent to backend)
    console.log("Blood Request Submitted:", formData);
    console.log("Uploaded Documents:", uploadedFiles.map(f => f.name));
    
    // Reset form
    setFormData({
      patientName: "",
      bloodGroup: "",
      requiredUnits: "",
      urgencyLevel: "normal",
      hospitalName: "",
      location: "",
      contactNumber: "",
    });
    setUploadedFiles([]);
  };

  const handleInputChange = (field: keyof PatientFormData, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
  };

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = e.target.files;
    if (files) {
      setUploadedFiles(Array.from(files));
    }
  };

  const handleRemoveFile = (index: number) => {
    const newFiles = uploadedFiles.filter((_, i) => i !== index);
    setUploadedFiles(newFiles);
  };

  return (
    <div className="w-full min-h-screen bg-gradient-to-b from-red-50 to-white p-4 sm:p-6">
      <div className="max-w-md mx-auto">
        {/* Header */}
        <div className="text-center mb-8">
          <div className="flex items-center justify-center gap-2 mb-3">
            <Droplet className="w-10 h-10 text-red-600 fill-red-600" />
            <h1 className="text-3xl text-red-600">HomoAid</h1>
          </div>
          <p className="text-gray-600">Emergency Blood Request System</p>
        </div>

        {/* Form Card */}
        <div className="bg-white rounded-2xl shadow-lg p-6 sm:p-8 border-2 border-red-100">
          <form onSubmit={handleSubmit} className="space-y-6">
            {/* Patient Name */}
            <div className="space-y-2">
              <Label htmlFor="patientName" className="text-base flex items-center gap-2">
                <User className="w-4 h-4 text-red-600" />
                Patient Name
              </Label>
              <Input
                id="patientName"
                type="text"
                value={formData.patientName}
                onChange={(e) => handleInputChange("patientName", e.target.value)}
                placeholder="Enter patient's full name"
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500"
                required
              />
            </div>

            {/* Blood Group */}
            <div className="space-y-2">
              <Label htmlFor="bloodGroup" className="text-base flex items-center gap-2">
                <Droplet className="w-4 h-4 text-red-600" />
                Blood Group
              </Label>
              <Select value={formData.bloodGroup} onValueChange={(value) => handleInputChange("bloodGroup", value)}>
                <SelectTrigger className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500">
                  <SelectValue placeholder="Select blood group" />
                </SelectTrigger>
                <SelectContent>
                  {bloodGroups.map((group) => (
                    <SelectItem key={group} value={group} className="text-base">
                      {group}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>

            {/* Required Units */}
            <div className="space-y-2">
              <Label htmlFor="requiredUnits" className="text-base">
                Required Units of Blood
              </Label>
              <Input
                id="requiredUnits"
                type="number"
                min="1"
                value={formData.requiredUnits}
                onChange={(e) => handleInputChange("requiredUnits", e.target.value)}
                placeholder="Enter number of units"
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500"
                required
              />
            </div>

            {/* Urgency Level */}
            <div className="space-y-3">
              <Label className="text-base flex items-center gap-2">
                <AlertCircle className="w-4 h-4 text-red-600" />
                Urgency Level
              </Label>
              <RadioGroup
                value={formData.urgencyLevel}
                onValueChange={(value) => handleInputChange("urgencyLevel", value)}
                className="flex gap-4"
              >
                <div className="flex items-center space-x-2 flex-1">
                  <RadioGroupItem value="normal" id="normal" className="border-gray-400" />
                  <Label htmlFor="normal" className="text-base cursor-pointer">
                    Normal
                  </Label>
                </div>
                <div className={`flex items-center space-x-2 flex-1 p-3 rounded-lg border-2 transition-all ${
                  formData.urgencyLevel === "emergency" 
                    ? "border-red-500 bg-red-50" 
                    : "border-transparent"
                }`}>
                  <RadioGroupItem value="emergency" id="emergency" className="border-red-500" />
                  <Label htmlFor="emergency" className="text-base cursor-pointer text-red-700">
                    Emergency
                  </Label>
                </div>
              </RadioGroup>
            </div>

            {/* Hospital Name */}
            <div className="space-y-2">
              <Label htmlFor="hospitalName" className="text-base flex items-center gap-2">
                <Hospital className="w-4 h-4 text-red-600" />
                Hospital Name
              </Label>
              <Input
                id="hospitalName"
                type="text"
                value={formData.hospitalName}
                onChange={(e) => handleInputChange("hospitalName", e.target.value)}
                placeholder="Enter hospital name"
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500"
                required
              />
            </div>

            {/* Location */}
            <div className="space-y-2">
              <Label htmlFor="location" className="text-base flex items-center gap-2">
                <MapPin className="w-4 h-4 text-red-600" />
                Location
              </Label>
              <Input
                id="location"
                type="text"
                value={formData.location}
                onChange={(e) => handleInputChange("location", e.target.value)}
                placeholder="Enter city or area"
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500"
                required
              />
            </div>

            {/* Contact Number */}
            <div className="space-y-2">
              <Label htmlFor="contactNumber" className="text-base flex items-center gap-2">
                <Phone className="w-4 h-4 text-red-600" />
                Contact Number
              </Label>
              <Input
                id="contactNumber"
                type="tel"
                value={formData.contactNumber}
                onChange={(e) => handleInputChange("contactNumber", e.target.value)}
                placeholder="Enter contact number"
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500"
                required
              />
            </div>

            {/* Upload Files */}
            <div className="space-y-2">
              <Label htmlFor="files" className="text-base flex items-center gap-2">
                <FileText className="w-4 h-4 text-red-600" />
                Upload Documents
              </Label>
              <Input
                id="files"
                type="file"
                multiple
                accept=".pdf,.jpg,.jpeg,.png,.doc,.docx"
                onChange={handleFileChange}
                className="h-12 text-base border-gray-300 focus:border-red-500 focus:ring-red-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:text-sm file:bg-red-50 file:text-red-700 hover:file:bg-red-100"
              />
              <p className="text-xs text-gray-500">Upload medical reports, prescriptions, or doctor's notes (PDF, Images, DOC) - Required</p>
              
              {uploadedFiles.length > 0 && (
                <div className="mt-3 space-y-2 bg-gray-50 p-3 rounded-lg border border-gray-200">
                  <p className="text-sm text-gray-700 mb-2">Uploaded Documents ({uploadedFiles.length}):</p>
                  {uploadedFiles.map((file, index) => (
                    <div key={index} className="flex items-center justify-between bg-white p-2 rounded border border-gray-200">
                      <div className="flex items-center gap-2 flex-1">
                        <FileText className="w-4 h-4 text-red-600 flex-shrink-0" />
                        <span className="text-sm text-gray-700 truncate">{file.name}</span>
                        <span className="text-xs text-gray-400">({(file.size / 1024).toFixed(1)} KB)</span>
                      </div>
                      <Button
                        type="button"
                        variant="ghost"
                        size="sm"
                        onClick={() => handleRemoveFile(index)}
                        className="text-red-600 hover:text-red-700 hover:bg-red-50 h-8 w-8 p-0"
                      >
                        <X className="w-4 h-4" />
                      </Button>
                    </div>
                  ))}
                </div>
              )}
            </div>

            {/* Submit Button */}
            <Button
              type="submit"
              className={`w-full h-14 text-lg transition-all ${
                formData.urgencyLevel === "emergency"
                  ? "bg-red-600 hover:bg-red-700 animate-pulse"
                  : "bg-red-600 hover:bg-red-700"
              }`}
            >
              {formData.urgencyLevel === "emergency" ? "🚨 Submit Emergency Request" : "Submit Blood Request"}
            </Button>
          </form>
        </div>

        {/* Footer Info */}
        <div className="mt-6 text-center text-sm text-gray-500">
          <p>Your request will be shared with verified donors in your area</p>
        </div>
      </div>
    </div>
  );
}
}
///code for the location sharing of the patient
import { useState } from "react";
import { PatientDetailsForm } from "./components/PatientDetailsForm";
import { LocationDetails } from "./components/LocationDetails";
import { Toaster } from "./components/ui/sonner";

export default function App() {
  const [activeScreen, setActiveScreen] = useState<"patient" | "location">("patient");

  return (
    <>
      {/* Navigation Tabs */}
      <div className="sticky top-0 z-10 bg-white border-b-2 border-red-100 shadow-sm">
        <div className="max-w-md mx-auto flex">
          <button
            onClick={() => setActiveScreen("patient")}
            className={`flex-1 py-4 text-base font-medium transition-all ${
              activeScreen === "patient"
                ? "text-red-600 border-b-4 border-red-600 bg-red-50"
                : "text-gray-500 hover:text-gray-700 hover:bg-gray-50"
            }`}
          >
            Patient Details
          </button>
          <button
            onClick={() => setActiveScreen("location")}
            className={`flex-1 py-4 text-base font-medium transition-all ${
              activeScreen === "location"
                ? "text-red-600 border-b-4 border-red-600 bg-red-50"
                : "text-gray-500 hover:text-gray-700 hover:bg-gray-50"
            }`}
          >
            Location Details
          </button>
        </div>
      </div>

      {/* Screen Content */}
      {activeScreen === "patient" ? <PatientDetailsForm /> : <LocationDetails />}
      
      <Toaster position="top-center" />
    </>
  );
}


