# VoyageKavach

VoyageKavach is a ride-booking web application that connects users with drivers. Users can request rides, and drivers can accept them. Once a ride is accepted, the user is redirected to a payment page to complete the transaction. The platform integrates Firebase for authentication and Firestore for real-time database management, Stripe for secure payments, and Google Maps API for navigation and route management.

## Firebase Integration Steps
1. Go to [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. Navigate to **Authentication** and enable Email/Password authentication.
3. Navigate to **Firestore Database** and create a Firestore database in test mode.
4. Navigate to **Project Settings > General**, and copy your Firebase config.
5. Add the Firebase config to your project:
   ```js
   import { initializeApp } from "firebase/app";
   import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword } from "firebase/auth";
   import { getFirestore, collection, addDoc, getDocs } from "firebase/firestore";
   
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_AUTH_DOMAIN",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_STORAGE_BUCKET",
     messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   
   const app = initializeApp(firebaseConfig);
   export const auth = getAuth(app);
   export const db = getFirestore(app);
   
   // Function to register a new user
   export const registerUser = async (email, password) => {
     try {
       const userCredential = await createUserWithEmailAndPassword(auth, email, password);
       return userCredential.user;
     } catch (error) {
       console.error("Error registering user:", error);
     }
   };
   
   // Function to sign in a user
   export const loginUser = async (email, password) => {
     try {
       const userCredential = await signInWithEmailAndPassword(auth, email, password);
       return userCredential.user;
     } catch (error) {
       console.error("Error signing in user:", error);
     }
   };
   
   // Function to add a ride request to Firestore
   export const addRideRequest = async (userId, pickup, dropoff) => {
     try {
       await addDoc(collection(db, "rides"), {
         userId,
         pickup,
         dropoff,
         status: "pending",
         timestamp: new Date()
       });
     } catch (error) {
       console.error("Error adding ride request:", error);
     }
   };
   
   // Function to fetch all ride requests
   export const fetchRideRequests = async () => {
     try {
       const querySnapshot = await getDocs(collection(db, "rides"));
       return querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
     } catch (error) {
       console.error("Error fetching ride requests:", error);
     }
   };
   ```
6. Initialize Firebase authentication and Firestore in your React app.
7. Set up Firestore rules to control access.

## Stripe Integration Steps
1. Sign up on [Stripe](https://stripe.com/) and get your API keys.
2. Install Stripe in your React project:
   ```sh
   npm install @stripe/react-stripe-js @stripe/stripe-js
   ```
3. Add the Stripe provider in `App.js`:
   ```js
   import { Elements } from "@stripe/react-stripe-js";
   import { loadStripe } from "@stripe/stripe-js";
   
   const stripePromise = loadStripe("YOUR_PUBLISHABLE_KEY");
   
   function App() {
     return (
       <Elements stripe={stripePromise}>
         <CheckoutForm />
       </Elements>
     );
   }
   ```
4. Set up your backend to create a payment intent using Flask:
   ```python
   import stripe
   from flask import Flask, request, jsonify
   
   app = Flask(__name__)
   stripe.api_key = "YOUR_SECRET_KEY"
   
   @app.route("/create-payment-intent", methods=["POST"])
   def create_payment():
       data = request.json
       intent = stripe.PaymentIntent.create(
           amount=int(data["amount"] * 100),
           currency="usd"
       )
       return jsonify({"clientSecret": intent.client_secret})
   ```
5. Use `fetch` in your frontend to call the backend for payment processing.


