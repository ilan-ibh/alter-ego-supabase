# Supabase Swift Setup Guide

This guide will help you integrate your Supabase project with your Alter Ego iOS app.

## Prerequisites

- Xcode 15.0+
- Swift 5.9+
- A Supabase project (follow the instructions in the main README)

## Step 1: Add the Supabase Swift Package

1. In Xcode, go to File > Add Packages...
2. Enter the URL: `https://github.com/supabase-community/supabase-swift`
3. Select "Up to Next Major Version" for dependency rule
4. Check these packages:
   - `Supabase` (core functionality)
   - `Auth` (for authentication)
   - `Realtime` (optional, for real-time updates)
   - `Storage` (optional, for file storage)
5. Click "Add Package"

## Step 2: Set Up Configuration Files

1. Create a template configuration file:

```swift
// SupabaseConfig-template.swift
import Foundation

// TEMPLATE FILE - Copy to SupabaseConfig.swift and add your Supabase credentials
struct SupabaseConfig {
    // Get these values from your Supabase project dashboard
    static let projectURL = "YOUR_SUPABASE_PROJECT_URL" // e.g. https://abcdefghijklm.supabase.co
    static let anonKey = "YOUR_SUPABASE_ANON_KEY" // Public anon key
}
```

2. Create a real configuration file with your actual credentials:

```swift
// SupabaseConfig.swift
import Foundation

// This file contains sensitive information and should not be committed to git
// See SupabaseConfig-template.swift for an example
struct SupabaseConfig {
    // Get these values from your Supabase project dashboard
    static let projectURL = "https://your-project-id.supabase.co" // Replace with your actual URL
    static let anonKey = "your-anon-key" // Replace with your actual anon key
}
```

3. Update `.gitignore` to exclude your config file:

```
# API Keys and Secrets
**/SupabaseConfig.swift
```

## Step 3: Create the Auth Service

```swift
import Foundation
import Supabase

class AuthService {
    private let supabase: SupabaseClient
    
    init() {
        // Initialize Supabase client
        let supabaseURL = URL(string: SupabaseConfig.projectURL)!
        self.supabase = SupabaseClient(
            supabaseURL: supabaseURL,
            supabaseKey: SupabaseConfig.anonKey
        )
    }
    
    // Get access to the Supabase client for other services
    func getSupabaseClient() -> SupabaseClient {
        return supabase
    }
    
    // Authentication methods (sign up, sign in, sign out, etc.)
    // ...
}
```

## Step 4: Create Message Repository

```swift
import Foundation
import Supabase

class MessageRepository {
    private let supabase: SupabaseClient
    
    init(supabase: SupabaseClient) {
        self.supabase = supabase
    }
    
    // CRUD operations for messages
    // ...
}
```

## Step 5: Initialize Services in Your App

In your `AlterEgoApp.swift`:

```swift
import SwiftUI

@main
struct AlterEgoApp: App {
    // Create all required view models
    @StateObject private var authViewModel = AuthViewModel()
    
    var body: some Scene {
        WindowGroup {
            // Show different views based on authentication state
            Group {
                if authViewModel.isAuthenticated {
                    ContentView()
                        .environmentObject(authViewModel)
                } else {
                    LoginView()
                        .environmentObject(authViewModel)
                }
            }
        }
    }
}
```

## Step 6: Handle Authentication State

In your `AuthViewModel.swift`:

```swift
@MainActor
class AuthViewModel: ObservableObject {
    @Published var currentUser: User?
    @Published var isAuthenticated = false
    
    private let authService = AuthService()
    
    init() {
        Task {
            await checkAuthStatus()
        }
    }
    
    func checkAuthStatus() async {
        // Check if user is already logged in
        // ...
    }
    
    // Expose Supabase client to other view models
    func getSupabaseClient() -> SupabaseClient {
        return authService.getSupabaseClient()
    }
    
    // Authentication methods
    // ...
}
```

## Step 7: Initialize Chat View Model with Dependencies

In your `ContentView.swift` or where you initialize `ChatViewModel`:

```swift
struct ContentView: View {
    @EnvironmentObject private var authViewModel: AuthViewModel
    @StateObject private var chatViewModel: ChatViewModel
    
    init() {
        // This will be initialized when ContentView is created
        // We can't use this directly since we need access to the authViewModel
        // which is passed as an environment object
        _chatViewModel = StateObject(wrappedValue: ChatViewModel(authViewModel: nil))
    }
    
    var body: some View {
        ChatView()
            .environmentObject(chatViewModel)
            .onAppear {
                // This will reinitialize the chatViewModel with the actual authViewModel
                // when the view appears
                if chatViewModel.authViewModel == nil {
                    chatViewModel.initialize(with: authViewModel)
                }
            }
    }
}
```

And in your `ChatViewModel.swift`:

```swift
@MainActor
class ChatViewModel: ObservableObject {
    // Your published properties
    
    var authViewModel: AuthViewModel?
    private var messageRepository: MessageRepository?
    
    init(authViewModel: AuthViewModel?) {
        self.authViewModel = authViewModel
        
        if let authViewModel = authViewModel {
            initialize(with: authViewModel)
        }
    }
    
    func initialize(with authViewModel: AuthViewModel) {
        self.authViewModel = authViewModel
        let supabase = authViewModel.getSupabaseClient()
        self.messageRepository = MessageRepository(supabase: supabase)
        
        // Load messages when initializing
        Task {
            await loadMessages()
        }
    }
    
    // Your methods for handling messages
    // ...
}
```

## Common Issues and Solutions

### 1. Authentication Errors

If you encounter authentication errors:
- Check your Supabase URL and anon key
- Ensure email confirmation settings match your app flow
- Verify that your device has internet connectivity

### 2. Database Access Issues

If you can't access the database:
- Check that you're signed in (authenticate first)
- Verify RLS policies are set up correctly
- Ensure your tables have the correct structure

### 3. Offline Support

For basic offline support:
- Cache messages locally using UserDefaults or Core Data
- Sync when the app comes back online
- Handle offline state in your UI

### 4. Testing

Test the integration:
- Create a test account
- Verify authentication flows
- Test message persistence
- Test across multiple devices

## Next Steps

After setting up basic authentication and message persistence, consider:

1. Implementing real-time updates using Supabase Realtime
2. Adding user profile management and customization
3. Setting up analytics to track app usage
4. Adding more complex data structures for advanced features