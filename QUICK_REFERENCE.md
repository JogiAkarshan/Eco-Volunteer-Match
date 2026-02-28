# 🎯 Quick Reference: User Roles System

## 3️⃣ User Types

### 1️⃣ Volunteer (Default)
```
Role: "volunteer"
Auto-verified: ❌ No
Default selection: ✅ Yes
Use case: Regular users
```

### 2️⃣ Organizer
```
Role: "organizer"
Auto-verified: ❌ No
Default selection: ❌ No
Use case: Event creators
```

### 3️⃣ Admin
```
Role: "admin"
Auto-verified: ✅ Yes
Default selection: ❌ No
Use case: System managers
```

---

## 🔄 Registration Flow

```
User Opens App
    ↓
Clicks "Register"
    ↓
Fills Form:
├─ Name
├─ Email
├─ Password
└─ Role (Dropdown)
    ↓
Clicks "Create Account"
    ↓
Frontend sends to Backend
    ↓
Backend validates role
    ├─ If volunteer: Create user, isVerified=false
    ├─ If organizer: Create user, isVerified=false
    └─ If admin: Create user, isVerified=true
    ↓
Backend creates JWT with role
    ↓
Frontend stores token + user data
    ↓
Shows: "Welcome [Name]! Role: [Role]"
```

---

## 📱 Frontend Changes

### Register Form - Before
```
┌─────────────────────┐
│ Eco Volunteer Match │
│                     │
│ Name: ___________   │
│ Email: __________   │
│ Password: _______   │
│ [Create Account]    │
└─────────────────────┘
```

### Register Form - After
```
┌─────────────────────┐
│ Eco Volunteer Match │
│                     │
│ Name: ___________   │
│ Email: __________   │
│ Password: _______   │
│ Role:           ▼   │  ← NEW DROPDOWN
│ [Volunteer      ▲]  │
│ [Organizer    ]     │
│ [Admin        ]     │
│ [Create Account]    │
└─────────────────────┘
```

---

## 🗄️ Database Schema

### User Document Structure
```javascript
{
  _id: ObjectId,
  name: String,              // User's name
  email: String,             // User's email (unique)
  passwordHash: String,      // Hashed password
  
  role: String,              // ← NEW: "volunteer" | "organizer" | "admin"
  isVerified: Boolean,       // ← NEW: Auto true for admin
  verificationCode: String,  // ← NEW: For 2FA
  
  city: String,              // Location
  points: Number,            // Reward points
  badges: [String],          // Achievements
  interests: [String],       // Interests
  joinedEventIds: [ObjectId], // Events joined
  
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🔑 JWT Token

### Token Payload
```javascript
{
  userId: "507f1f77bcf86cd799439011",
  role: "organizer",        // ← NEW
  iat: 1709123456,
  exp: 1709728256
}
```

---

## 📡 API Responses

### Register Response
```json
{
  "token": "eyJhbGciOi...",
  "user": {
    "_id": "507f1f77bcf86cd7...",
    "name": "Sarah",
    "email": "sarah@test.com",
    "city": "Hyderabad",
    "role": "organizer",        ← NEW
    "isVerified": false         ← NEW
  }
}
```

### Login Response
```json
{
  "token": "eyJhbGciOi...",
  "user": {
    "_id": "507f1f77bcf86cd7...",
    "name": "Sarah",
    "email": "sarah@test.com",
    "city": "Hyderabad",
    "role": "organizer",        ← RETURNED
    "isVerified": false         ← RETURNED
  }
}
```

---

## 💾 LocalStorage After Login

```javascript
// Key: user
// Value:
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "Sarah",
  "email": "sarah@test.com",
  "city": "Hyderabad",
  "role": "organizer",
  "isVerified": false
}

// Key: token
// Value:
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## 🧪 Test Data

### Sample Users in MongoDB

#### Volunteer
```javascript
db.users.findOne({ role: "volunteer" })
{
  name: "John Doe",
  email: "john@test.com",
  role: "volunteer",
  isVerified: false,
  points: 0
}
```

#### Organizer
```javascript
db.users.findOne({ role: "organizer" })
{
  name: "Sarah Smith",
  email: "sarah@test.com",
  role: "organizer",
  isVerified: false,
  points: 0
}
```

#### Admin
```javascript
db.users.findOne({ role: "admin" })
{
  name: "Admin User",
  email: "admin@test.com",
  role: "admin",
  isVerified: true,        ← AUTO-VERIFIED
  points: 0
}
```

---

## ✅ Verification Checklist

After registering 3 users:

```
✓ Volunteer registered
✓ Organizer registered
✓ Admin registered

✓ Role appears in success message
✓ Role stored in localStorage
✓ Role stored in MongoDB
✓ Role in JWT token
✓ Admin has isVerified: true
✓ Others have isVerified: false
```

---

## 📊 Code Files Quick Reference

| File | What Changed |
|------|--------------|
| `server/src/models/User.js` | Added role, isVerified fields |
| `server/src/routes/auth.js` | Added role validation, JWT role, response role |
| `src/pages/Auth.jsx` | Added role dropdown, send role, show role |
| `src/styles.css` | Added select dropdown styling |

---

## 🎓 Person 8's Explanation Topics

1. **Why three roles?**
   - Different user types need different features
   - Volunteers browse events
   - Organizers create events
   - Admins manage system

2. **Role validation (enum)?**
   - Ensures only valid values stored
   - Prevents database corruption
   - MongoDB rejects invalid roles

3. **Auto-verified for admin?**
   - Security: admins need immediate access
   - Others need email verification first
   - Prevents admin lockout

4. **Role in JWT token?**
   - Fast permission checks (no DB lookup)
   - Token is cryptographically signed
   - Can't be forged

---

## 📱 Future Frontend Uses

```javascript
// Show different UI based on role:
if (user.role === "volunteer") {
  // Show: Browse Events, Register buttons
}

if (user.role === "organizer") {
  // Show: Create Event, Manage Events buttons
}

if (user.role === "admin") {
  // Show: Admin Panel, Manage Users buttons
}

// Protected routes:
<Route path="/admin/*" element={
  user.role === "admin" ? <AdminDash/> : <NotAuthorized/>
} />
```

---

## 🔐 Security Notes

**Current:** Users can select any role  
**Production:** Restrict to volunteer only, admins create others

**Frontend:** Role-based UI display  
**Backend:** Role-based route protection (not yet implemented)

---

## 📞 Quick Demo Script

```
"We've implemented a role-based user system with three types:

1. Volunteer - regular users
2. Organizer - creates events
3. Admin - manages system

Watch me register three users:

[Register as Volunteer]
✓ Role: volunteer, isVerified: false

[Register as Organizer]
✓ Role: organizer, isVerified: false

[Register as Admin]
✓ Role: admin, isVerified: true [Auto-verified!]

All roles stored in MongoDB.
Role included in JWT token.
Frontend knows user type for permission control.

This enables role-based access control for the entire app."
```

---

## ✨ Summary

| Feature | Status |
|---------|--------|
| 3 user types | ✅ Implemented |
| Role selection | ✅ Implemented |
| Role validation | ✅ Implemented |
| Role in database | ✅ Implemented |
| Role in JWT | ✅ Implemented |
| Admin auto-verify | ✅ Implemented |
| Role-based UI | ⏳ Ready to implement |
| Route protection | ⏳ Ready to implement |

---

**You now have a complete role-based system ready! 🚀**
