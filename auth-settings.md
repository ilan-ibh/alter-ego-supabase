# Authentication Settings for Alter Ego

This guide outlines the recommended authentication settings for your Supabase project to work with the Alter Ego app.

## Email Authentication Settings

1. Go to Authentication → Settings → Email Auth in your Supabase dashboard

2. Configure the following:

   - **Email Auth**: Enabled
   - **Confirm Email**: Optional (Enable for production)
   - **Secure Email Change**: Enabled
   - **Custom SMTP Server**: 
     - Use your own SMTP server for production
     - For testing, Supabase's default server is fine

3. Email Templates:
   - Customize the email templates to match your brand
   - Update the confirmation, invitation, and magic link templates

## Password Settings

1. Go to Authentication → Settings → Auth Providers

2. Configure the following:
   - **Minimum Password Length**: 8
   - **Require Strong Password**: Enabled (for production)
   - **Password Protection for Email Signups**: Enabled

## External OAuth Providers (Optional)

For social login functionality, you can enable:

1. **Apple**
   - Set up an Apple Developer account
   - Configure the service ID and secret key

2. **Google**
   - Create a project in Google Cloud Console
   - Set up OAuth credentials

3. **Other Providers**
   - Facebook, Twitter, GitHub, etc.

## Custom Configuration (Swift)

In your Swift code, ensure your Supabase client is properly configured:

```swift
import Supabase

// Initialize Supabase client
let supabase = SupabaseClient(
    supabaseURL: URL(string: SupabaseConfig.projectURL)!,
    supabaseKey: SupabaseConfig.anonKey
)

// Sign up
func signUp(email: String, password: String) async throws {
    try await supabase.auth.signUp(
        email: email,
        password: password
    )
}

// Sign in
func signIn(email: String, password: String) async throws {
    try await supabase.auth.signIn(
        email: email,
        password: password
    )
}

// Sign out
func signOut() async throws {
    try await supabase.auth.signOut()
}
```

## Security Recommendations

1. **Enable MFA** (Multi-Factor Authentication) for additional security
2. **Set up Rate Limiting** to prevent brute force attacks
3. **Configure Trusted Browsers** to require email confirmation when signing in from a new device
4. **Set Reasonable Session Lengths** (default: 1 week)
5. **Create Custom Claims** for role-based access control if needed

## Testing Authentication

1. Use test credentials during development
2. Verify email flows by registering real test accounts
3. Test password recovery functionality
4. Ensure proper error handling for invalid credentials

## Troubleshooting

Common issues:

- **CORS errors**: Check your site URL configuration
- **Redirect issues**: Ensure your redirect URLs are properly set
- **Email delivery problems**: Verify SMTP settings or switch to a custom provider