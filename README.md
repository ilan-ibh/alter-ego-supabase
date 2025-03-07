# Alter Ego - Supabase Setup

This repository contains the database schema and configuration for the Alter Ego AI Companion app's Supabase backend.

## Setup Instructions

### 1. Create a Supabase Project

1. Go to [Supabase.com](https://supabase.com) and sign up or log in
2. Click "New Project" and fill out the details:
   - Name your project (e.g., "alter-ego-app")
   - Set a secure database password
   - Choose the region closest to your users
   - Select the free plan (or appropriate tier)
   - Click "Create new project"

### 2. Set Up Database Schema

1. Once your project is created, go to the SQL Editor in your Supabase dashboard
2. Copy the SQL from [schema.sql](./schema.sql) in this repository
3. Paste the SQL into the editor and click "Run"
4. Verify that the tables and policies were created successfully

### 3. Configure Authentication

1. Go to Authentication → Settings in your Supabase dashboard
2. Under Email Auth, ensure "Enable Email Signup" is turned on
3. Configure additional settings as needed:
   - Decide whether to require email confirmation
   - Set up password restrictions
   - Configure redirect URLs for password recovery

### 4. Get API Credentials

1. Go to Project Settings → API in your Supabase dashboard
2. Copy the "Project URL" and "anon public" key
3. In your Alter Ego app, update the `SupabaseConfig.swift` file with these values:

```swift
struct SupabaseConfig {
    static let projectURL = "YOUR_SUPABASE_PROJECT_URL"  // from Supabase dashboard
    static let anonKey = "YOUR_SUPABASE_ANON_KEY"        // from Supabase dashboard
}
```

## Database Structure

### Users
The app uses Supabase Auth for user authentication. User metadata is stored in the `profiles` table.

### Profiles Table
Stores additional user information:
- `id`: UUID (references auth.users)
- `email`: Text
- `user_name`: Text (optional)
- `created_at`: Timestamp

### Messages Table
Stores all chat messages:
- `id`: UUID
- `user_id`: UUID (references auth.users)
- `content`: Text
- `is_user_message`: Boolean
- `timestamp`: Timestamp

## Security

The database uses Row Level Security (RLS) to ensure users can only:
- View any user's profile
- Insert and update their own profile
- View, insert and update only their own messages

## Extended Functionality

Future phases may include additional tables for:
- User preferences
- Conversation contexts
- Vector embeddings for RAG
- Knowledge graph relationships