# ASP.NET Core Identity Authentication Flows

## 1. User Registration Flow

```
[Browser] ---> [HTTP Request: GET /Account/Register] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> No auth required for registration
       ---> [Authorization Middleware] --> [AllowAnonymous] allows access
       ---> [AccountController.Register(GET)] ---> [Return Register.cshtml View]
       ---> [Browser displays registration form]

User fills form with: Username, Email, Password, ConfirmPassword and submits:

[Browser] ---> [HTTP POST /Account/Register] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> Still no auth required
       ---> [Authorization Middleware] --> [AllowAnonymous] allows access
       ---> [AccountController.Register(POST)]
       ---> [Model Binding] --> Bind RegisterViewModel from form data
       ---> [Model Validation] --> Check DataAnnotations, password strength
            |---> If validation fails: Return Register.cshtml with errors
            |---> If validation passes: Continue
       ---> [UserManager.CreateAsync(user, password)]
            |---> Check if username/email already exists
            |---> Hash password using PBKDF2/Argon2
            |---> Store user in InMemoryDatabase/SQL Database
            |---> If creation fails: Return Register.cshtml with errors
            |---> If creation succeeds: Continue
       ---> [SignInManager.SignInAsync(user, isPersistent: false)]
            |---> Generate authentication claims (UserId, UserName, Email, Roles)
            |---> Create ClaimsIdentity with CookieAuthenticationDefaults.AuthenticationScheme
            |---> Set authentication cookie with encrypted claims
            |---> Cookie expires when browser closes (isPersistent: false)
       ---> [RedirectToAction("Index", "Home")] --> 302 redirect
       ---> [Browser] --> Automatically follows redirect to /Home/Index
```

## 2. User Login Flow

```
[Browser] ---> [HTTP Request: GET /Account/Login] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> No auth required for login page
       ---> [Authorization Middleware] --> [AllowAnonymous] allows access
       ---> [AccountController.Login(GET)] 
       ---> [Check if user already authenticated]
            |---> If User.Identity.IsAuthenticated == true: RedirectToAction("Index", "Home")
            |---> If not authenticated: Return Login.cshtml View
       ---> [Browser displays login form]

User enters credentials and submits:

[Browser] ---> [HTTP POST /Account/Login] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> Still no auth required
       ---> [Authorization Middleware] --> [AllowAnonymous] allows access
       ---> [AccountController.Login(POST)]
       ---> [Model Binding] --> Bind LoginViewModel (Username, Password, RememberMe)
       ---> [Model Validation] --> Check required fields, format validation
            |---> If validation fails: Return Login.cshtml with errors
            |---> If validation passes: Continue
       ---> [SignInManager.PasswordSignInAsync(username, password, rememberMe, lockoutOnFailure)]
            |---> [UserManager.FindByNameAsync(username)] --> Find user in database
            |---> If user not found: Return failed result
            |---> [UserManager.CheckPasswordAsync(user, password)]
                   |---> Hash provided password and compare with stored hash
                   |---> If password incorrect: 
                         |---> Increment failed login attempts
                         |---> If lockoutOnFailure && attempts > threshold: Lock account
                         |---> Return failed result
                   |---> If password correct: Continue
            |---> [Check if account is locked out]
                   |---> If locked: Return lockout result
                   |---> If not locked: Continue
            |---> [Reset failed login attempts to 0]
            |---> [Generate authentication claims and set cookie]
                   |---> Claims: UserId, UserName, Email, SecurityStamp, Roles
                   |---> If rememberMe == true: Cookie persists for weeks/months
                   |---> If rememberMe == false: Cookie expires when browser closes
            |---> Return success result
       ---> [Check SignInResult]
            |---> If result.Succeeded: RedirectToAction("Index", "Home")
            |---> If result.IsLockedOut: Return Login.cshtml with lockout message
            |---> If result.RequiresTwoFactor: RedirectToAction("TwoFactorLogin")
            |---> If failed: Return Login.cshtml with "Invalid credentials" error
```

## 3. Protected Page Access Flow

```
[Browser] ---> [HTTP Request: GET /Home/SecretPage] ---> [Routing Middleware]
       ---> [Authentication Middleware]
            |---> Read authentication cookie from request
            |---> Decrypt and validate cookie
            |---> If cookie valid: Create ClaimsPrincipal and set HttpContext.User
            |---> If cookie invalid/missing: HttpContext.User remains anonymous
       ---> [Authorization Middleware]
            |---> Check if action has [Authorize] attribute
            |---> If [Authorize] present: Check if User.Identity.IsAuthenticated == true
            |---> If authenticated: Continue to controller
            |---> If not authenticated: 
                   |---> Return 302 redirect to /Account/Login?ReturnUrl=%2FHome%2FSecretPage
                   |---> Browser automatically follows redirect to login page
       ---> [HomeController.SecretPage()] --> Only reached if authenticated
       ---> [Return SecretPage.cshtml with user-specific content]
```

## 4. User Logout Flow

```
[Browser] ---> [HTTP Request: GET/POST /Account/Logout] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> Reads current auth cookie
       ---> [Authorization Middleware] --> Usually requires authentication to logout
       ---> [AccountController.Logout()]
       ---> [SignInManager.SignOutAsync()]
            |---> Clear authentication cookie by setting expiry to past date
            |---> Remove all authentication claims from current context
            |---> Clear HttpContext.User (becomes anonymous)
            |---> Raise SignedOut events for any registered handlers
       ---> [RedirectToAction("Index", "Home")] or [RedirectToAction("Login")]
       ---> [Browser] --> Follows redirect, now accessing as anonymous user
```

## 5. Role-Based Authorization Flow

```
[Browser] ---> [HTTP Request: GET /Admin/Dashboard] ---> [Routing Middleware]
       ---> [Authentication Middleware] --> Validates auth cookie, sets User claims
       ---> [Authorization Middleware]
            |---> Check action has [Authorize(Roles = "Admin")] attribute
            |---> If user not authenticated: Redirect to login
            |---> If user authenticated: Check if User.IsInRole("Admin")
                   |---> Get user's role claims from cookie/database
                   |---> If user has "Admin" role: Continue to controller
                   |---> If user lacks "Admin" role: Return 403 Forbidden
       ---> [AdminController.Dashboard()] --> Only reached if user is authenticated Admin
       ---> [Return admin dashboard view]
```

## 6. Password Reset Flow

```
[Browser] ---> [GET /Account/ForgotPassword] ---> [Return ForgotPassword.cshtml form]

User enters email and submits:

[Browser] ---> [POST /Account/ForgotPassword] ---> [AccountController.ForgotPassword(POST)]
       ---> [Model Validation] --> Validate email format
       ---> [UserManager.FindByEmailAsync(email)] --> Find user by email
            |---> If user not found: Return same page (don't reveal if email exists)
            |---> If user found: Continue
       ---> [UserManager.GeneratePasswordResetTokenAsync(user)]
            |---> Generate cryptographically secure token
            |---> Token tied to user and current SecurityStamp
            |---> Store token temporarily (usually in database or cache)
       ---> [Send email with reset link] --> /Account/ResetPassword?token=xyz&email=user@domain.com
       ---> [Return ForgotPasswordConfirmation.cshtml] --> "Check your email" message

User clicks link in email:

[Browser] ---> [GET /Account/ResetPassword?token=xyz&email=user@domain.com]
       ---> [AccountController.ResetPassword(GET)]
       ---> [Return ResetPassword.cshtml form] --> Pre-populate email, hide token

User enters new password and submits:

[Browser] ---> [POST /Account/ResetPassword] ---> [AccountController.ResetPassword(POST)]
       ---> [Model Validation] --> Validate new password strength
       ---> [UserManager.ResetPasswordAsync(user, token, newPassword)]
            |---> Validate token is correct and not expired
            |---> If token invalid: Return error
            |---> Hash new password
            |---> Update user's password in database
            |---> Invalidate all existing tokens by updating SecurityStamp
       ---> [RedirectToAction("Login")] --> User 🔹 1. Request → Response (Downstream)

When a request comes in (e.g., GET /api/users), it flows through the pipeline top to bottom:

    HTTP Request received by Kestrel

        The ASP.NET Core web server (Kestrel) accepts the raw HTTP request.

    Middleware #1 (e.g., Logging / Exception Handling)

        Logs details about the request.

        Can short-circuit and return a response immediately (e.g., 400 Bad Request).

    Middleware #2 (e.g., Authentication)

        Checks if the request has valid authentication tokens/headers.

        If invalid, it may stop and return 401 Unauthorized.

    Middleware #3 (e.g., Authorization)

        Validates whether the authenticated user has permission to access the resource.

        Can stop early with 403 Forbidden.

    Middleware #4 (e.g., Routing)

        Matches the request to the correct route (/api/users → UsersController.Get).

        Passes request details to the endpoint.

    Middleware #5 (MVC / Endpoint Execution)

        Calls your controller action (or minimal API endpoint).

        Example: UsersController.GetUsers() is executed.

        Returns the result (like JSON) → starts the response flow.

🔹 2. Response → Request (Upstream)

After the controller action finishes, the response travels back up the pipeline, each middleware gets a chance to modify it:

    Middleware #5 (MVC / Endpoint Execution)

        Converts the controller result into a proper HTTP response (e.g., JSON serialization).

        Passes it back up.

    Middleware #4 (Routing)

        Adds routing metadata (if needed).

        Doesn’t usually modify the response.

    Middleware #3 (Authorization)

        Response goes through unchanged unless policies require modifications.

    Middleware #2 (Authentication)

        May add authentication headers/tokens in response.

    Middleware #1 (Logging / Exception Handling)

        Logs response details (status code, duration).

        Can also catch exceptions thrown by downstream.

    Kestrel sends response back to client

        Final HTTP response is written to the socket and sent to the caller.
must log in with new password
```

## 7. Two-Factor Authentication Flow

```
[Browser] ---> [POST /Account/Login] --> Standard login flow
       ---> [SignInManager.PasswordSignInAsync()] --> Password correct
       ---> [Check if 2FA enabled for user]
            |---> If 2FA disabled: Complete signin and redirect to Home
            |---> If 2FA enabled: Return result.RequiresTwoFactor == true
       ---> [RedirectToAction("TwoFactorLogin")]
       ---> [Browser] --> GET /Account/TwoFactorLogin

Two-Factor Login Page:

[Browser] ---> [AccountController.TwoFactorLogin(GET)]
       ---> [SignInManager.GetTwoFactorAuthenticationUserAsync()]
            |---> Retrieve user from temporary signin (not fully signed in yet)
            |---> If no temp user: Redirect to Login
       ---> [Generate and send 2FA code] --> SMS/Email/Authenticator app
       ---> [Return TwoFactorLogin.cshtml form]

User enters 2FA code:

[Browser] ---> [POST /Account/TwoFactorLogin] --> [AccountController.TwoFactorLogin(POST)]
       ---> [Model Validation] --> Validate code format
       ---> [SignInManager.TwoFactorAuthenticatorSignInAsync(code, rememberMe, rememberMachine)]
            |---> Validate the provided code against expected code
            |---> If code invalid: Return error
            |---> If code valid: Complete the signin process
                   |---> Set full authentication cookie
                   |---> If rememberMachine: Set cookie to bypass 2FA on this device
       ---> [RedirectToAction("Index", "Home")] --> User now fully authenticated
```

## Key Components

- **UserManager**: Manages user operations (create, find, password hashing, etc.)
- **SignInManager**: Handles signin/signout operations and cookie management
- **Authentication Middleware**: Reads and validates authentication cookies
- **Authorization Middleware**: Enforces access control based on user claims/roles
- **Identity Database**: Stores users, roles, claims (can be SQL Server, InMemory, etc.)
- **Authentication Cookies**: Encrypted cookies containing user claims and identity info
