
## Overview

This document explains the login flow in the application, with specific focus on the conditional redirection based on whether OTP (One-Time Password) verification is required, the subsequent OTP verification process, and the OTP resend mechanism.

## Authentication Functions

### Login Function

The `login` function is responsible for authenticating users and handling appropriate redirections based on the authentication response.

```dart
Future<void> login({required BuildContext context, required VoidCallback redirectTo}) async {
  // Function implementation
}
```

### OTP Verification Function

The `verifyOtp` function handles the verification of the One-Time Password after redirection to the MFA page.

```dart
Future<void> verifyOtp({required String otp, required BuildContext context}) async {
  // Function implementation
}
```

### OTP Resend Widget

The `ResendWithStreamTimer` widget allows users to request a new OTP code when needed, with a cooldown timer to prevent abuse.

```dart
class ResendWithStreamTimer extends StatefulWidget {
  const ResendWithStreamTimer({super.key, required this.state});
  final LoginViewModel state;
  // Implementation
}
```

#### Methods

- `_startTimer()`: Initializes a 120-second countdown timer and updates the UI
- `_timerText`: Getter that formats the remaining time as "MM:SS"
- `dispose()`: Cancels the timer when the widget is disposed to prevent memory leaks


## Flow Diagram

### Login Flow

```
┌─────────────────┐
│  Start Login    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Validate      │
│ Organization URL│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Handle Remember │
│      Me         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Validate      │
│   Form Input    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Send Login      │
│    Request      │
└────────┬────────┘
         │
         ▼
     ┌───┴───┐
     │       │
     ▼       ▼
┌─────────┐  ┌─────────┐
│ OTP     │  │ Regular │
│ Required│  │ Login   │
└────┬────┘  └────┬────┘
     │           │
     ▼           ▼
┌─────────┐  ┌─────────┐
│Redirect │  │Set User │
│  to     |
| OTPPage │  │ Details │
└────┬────┘  └─────────┘

```

### OTP Verification Flow

```
┌─────────────────┐
│ Start OTP       │
│ Verification    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Show Loader     │
│ Set Loading     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Send OTP        │
│ Verification    │
└────────┬────────┘
         │
         ▼
     ┌───┴───┐
     │       │
     ▼       ▼
┌─────────┐  ┌─────────┐
│ Success │  │ Error   │
└────┬────┘  └────┬────┘
     │           │
     ▼           ▼
┌─────────┐  ┌─────────┐
│Set User │  │ Show    │
│ Details │  │ Error   │
└─────────┘  └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ Check   │
             │ if Login│
             │ Required│
             └─────────┘
```

### OTP Resend Flow

```
┌─────────────────┐
│ User Interface  │
└────────┬────────┘
         │
         ▼
     ┌───┴───┐
     │       │
     ▼       ▼
┌─────────┐  ┌─────────┐
│ Timer   │  │ Resend  │
│ Active  │  │ Option  │
└────┬────┘  └────┬────┘
     │           │
     ▼           ▼
┌─────────┐  ┌─────────┐
│ Show    │  │ Resend  │
│ Timer   │  │ Clicked │
└─────────┘  └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ Request │
             │ New OTP │
             └────┬────┘
                  │
                  ▼
             ┌─────────┐
             │ Start   │
             │ Timer   │
             └─────────┘
```

## Authentication Flow

### 1. Initial Login Process

#### 1.1 Initial Validation

- The function begins by showing a loader and retrieving the organization URL from shared preferences
- Validates if the organization URL is properly set
- If URL is invalid, shows an error message and stops the process

#### 1.2 Remember Me Handling

- If "Remember Me" is enabled:
  - Stores username and password in shared preferences
- If "Remember Me" is disabled:
  - Removes stored credentials from shared preferences

#### 1.3 Form Validation

- Validates the login form fields
- Constructs the login request body with email, password, and username

#### 1.4 Authentication Request

- Sends authentication request to the server using the `AuthService().login()` method
- Processes the response from the server

#### 1.5 Conditional Flow Based on OTP Requirement

##### 1.5.1 MFA Required Flow (OTP Needed)

If the server response indicates that OTP verification is needed:
```dart
if (userDetails.nextStep != null && userDetails.otpKey != null) {
  otpKey = userDetails.otpKey;
  if (context.mounted) {
    Loader.of(context).hide();
    redirectTo();  // Redirects to MFA page
  }
  return;
}
```

- The function saves the OTP key received from the server
- Hides the loader
- Calls the `redirectTo` callback to navigate to the MFA/OTP entry page
- Returns early from the function

##### 1.5.2 Regular Login Flow (No OTP Needed)

If the server response contains a valid token:
```dart
if (userDetails.token != null && userDetails.token?.access != null) {
  setUserDeatils(userDetails: userDetails);
} else {
  throw 'Access token not available';
}
```

- The function calls `setUserDeatils()` to save user information
- The application proceeds with normal login flow





## Complete Authentication Flow

1. **Initial Login**
   - User enters credentials and organization URL
   - System validates inputs and sends authentication request
   - Server determines if OTP verification is required

2. **Conditional Path**
   - **Path A (MFA Required)**: 
     - System stores OTP key
     - User is redirected to MFA page
     - User enters OTP code
     - System verifies OTP with server
     - On success, user details are set
   
   - **Path B (No MFA Required)**:
     - User details and tokens are set directly
     - User proceeds to application

3. **OTP Resend Mechanism**
   - If user doesn't receive the OTP code, they can request a new one
   - "Resend" button appears when no timer is active
   - After clicking "Resend", a 120-second countdown timer starts
   - During countdown, user sees timer instead of "Resend" option
   - After timer completes, "Resend" button reappears
   - Process repeats as needed until successful verification

4. **Error Handling**
   - Input validation errors are shown to the user
   - Authentication errors are handled gracefully
   - Session expiry redirects user to login
   - Network errors inform the user to try again

