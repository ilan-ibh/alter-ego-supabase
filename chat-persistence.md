# Chat Persistence Implementation Guide

This guide explains how to implement chat persistence in the Alter Ego app using Supabase.

## Database Structure

The `messages` table in Supabase stores all chat messages:

```sql
CREATE TABLE IF NOT EXISTS messages (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users NOT NULL,
  content TEXT NOT NULL,
  is_user_message BOOLEAN NOT NULL,
  timestamp TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

## Swift Implementation

### 1. Create a Message Repository

First, create a `MessageRepository` class to handle database operations:

```swift
import Foundation
import Supabase

class MessageRepository {
    private let supabase: SupabaseClient
    
    init(supabase: SupabaseClient) {
        self.supabase = supabase
    }
    
    // Save a message to the database
    func saveMessage(_ message: Message, userId: String) async throws {
        let messageData: [String: Any] = [
            "user_id": userId,
            "content": message.content,
            "is_user_message": message.isUserMessage,
            "timestamp": ISO8601DateFormatter().string(from: message.timestamp)
        ]
        
        let query = supabase.database
            .from("messages")
            .insert(values: messageData)
        
        try await query.execute()
    }
    
    // Fetch all messages for a user
    func fetchMessages(userId: String) async throws -> [Message] {
        let query = supabase.database
            .from("messages")
            .select()
            .eq("user_id", value: userId)
            .order("timestamp", ascending: true)
        
        let response = try await query.execute()
        let decoder = JSONDecoder()
        decoder.dateDecodingStrategy = .iso8601
        
        // Use your existing Message model with these extra properties to decode
        struct MessageDTO: Codable {
            let id: String
            let user_id: String
            let content: String
            let is_user_message: Bool
            let timestamp: Date
        }
        
        let messageDTOs = try decoder.decode([MessageDTO].self, from: response.data)
        
        // Convert DTOs to your app's Message model
        return messageDTOs.map { dto in
            Message(
                id: dto.id,
                content: dto.content,
                isUserMessage: dto.is_user_message,
                timestamp: dto.timestamp
            )
        }
    }
    
    // Delete all messages for a user
    func clearMessages(userId: String) async throws {
        let query = supabase.database
            .from("messages")
            .delete()
            .eq("user_id", value: userId)
        
        try await query.execute()
    }
}
```

### 2. Update the ChatViewModel

Next, update your ChatViewModel to use the MessageRepository:

```swift
@MainActor
class ChatViewModel: ObservableObject {
    @Published var messages: [Message] = []
    @Published var inputMessage: String = ""
    @Published var isProcessing: Bool = false
    @Published var errorMessage: String? = nil
    
    private let openAIService = OpenAIService()
    private let messageRepository: MessageRepository
    private let authViewModel: AuthViewModel
    
    init(authViewModel: AuthViewModel) {
        self.authViewModel = authViewModel
        
        // Get the Supabase instance from the AuthService
        let supabase = authViewModel.getSupabaseClient()
        self.messageRepository = MessageRepository(supabase: supabase)
        
        // Load messages when initializing
        Task {
            await loadMessages()
        }
    }
    
    // Load messages from the database
    private func loadMessages() async {
        guard let userId = authViewModel.currentUser?.id else { return }
        
        do {
            let fetchedMessages = try await messageRepository.fetchMessages(userId: userId)
            self.messages = fetchedMessages
        } catch {
            errorMessage = "Failed to load messages: \(error.localizedDescription)"
        }
    }
    
    // Save a message to the database
    private func saveMessage(_ message: Message) async {
        guard let userId = authViewModel.currentUser?.id else { return }
        
        do {
            try await messageRepository.saveMessage(message, userId: userId)
        } catch {
            errorMessage = "Failed to save message: \(error.localizedDescription)"
        }
    }
    
    // Modified sendMessage to persist messages
    func sendMessage() async {
        guard !inputMessage.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else {
            return
        }
        
        // Create and add user message
        let userMessage = Message(content: inputMessage, isUserMessage: true)
        messages.append(userMessage)
        
        // Save user message to database
        await saveMessage(userMessage)
        
        // Clear input field
        inputMessage = ""
        
        // Set processing state
        isProcessing = true
        
        // Add delays and processing as in your original implementation
        
        do {
            // Send message to OpenAI and get response
            let response = try await openAIService.sendMessage(messages: messages)
            
            // Create and add AI response
            let aiMessage = Message(content: response, isUserMessage: false)
            messages.append(aiMessage)
            
            // Save AI message to database
            await saveMessage(aiMessage)
            
        } catch {
            // Error handling as in your original implementation
            // Also save the error message to the database if needed
        }
        
        // Reset processing state
        isProcessing = false
    }
    
    // Clear chat and delete from database
    func clearChat() async {
        guard let userId = authViewModel.currentUser?.id else { return }
        
        do {
            try await messageRepository.clearMessages(userId: userId)
            messages.removeAll()
            errorMessage = nil
        } catch {
            errorMessage = "Failed to clear messages: \(error.localizedDescription)"
        }
    }
}
```

### 3. Modify the Supabase Client Access

Update your AuthViewModel to provide access to the Supabase client:

```swift
@MainActor
class AuthViewModel: ObservableObject {
    // Existing properties and methods
    
    // Get access to the Supabase client
    func getSupabaseClient() -> SupabaseClient {
        return authService.getSupabaseClient()
    }
}

// And in AuthService:
class AuthService {
    private let supabase: SupabaseClient
    
    // Existing methods
    
    // Allow access to the Supabase client
    func getSupabaseClient() -> SupabaseClient {
        return supabase
    }
}
```

## Testing

1. Test saving messages
2. Test loading messages when the app starts
3. Test clearing messages
4. Verify that messages persist across app restarts
5. Ensure message order is preserved

## Privacy Considerations

- Messages are protected with Row Level Security (RLS)
- Users can only see their own messages
- Consider implementing message encryption for sensitive content