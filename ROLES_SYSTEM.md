# User Roles System - Admin, Organizer, Volunteer

## 🎯 Three User Types Now Available

### 1. **Volunteer** (Default)
```javascript
role: "volunteer"
```
**What they can do:**
- Register for environmental events
- Earn points for volunteering
- Collect badges
- View their profile
- Track volunteer hours

**Use case:** Regular users joining volunteer activities

---

### 2. **Organizer** (Event Creator)
```javascript
role: "organizer"
```
**What they can do:**
- Create environmental events
- Manage event details
- View volunteer registrations
- Track event participation
- Send announcements
- View reports

**Use case:** NGOs, environmental groups creating events

---

### 3. **Admin** (System Manager)
```javascript
role: "admin"
```
**What they can do:**
- Manage all users
- Approve organizers
- Delete inappropriate content
- View system statistics
- Manage badges and rewards
- Access all data

**Use case:** Project managers, system administrators

---

## 📊 Database Schema Changes

### User Collection Now Stores:
```javascript
{
  _id: ObjectId,
  name: "John Doe",
  email: "john@test.com",
  passwordHash: "$2b$10$hashed...",
  role: "organizer",              // ← NEW FIELD
  city: "Hyderabad",
  points: 100,
  badges: ["b1", "b2"],
  isVerified: true,               // ← NEW FIELD
  verificationCode: null,         // ← NEW FIELD
  joinedEventIds: [...],
  interests: ["wildlife", "climate"],
  createdAt: ISODate("2025-02-28T10:30:00Z"),
  updatedAt: ISODate("2025-02-28T10:30:00Z")
}
```

---

## 🔧 Updated Code Files

### 1. `server/src/models/User.js`

**New Fields Added:**
```javascript
role: { 
  type: String, 
  enum: ["volunteer", "organizer", "admin"],  // Only these values allowed
  default: "volunteer"                        // Default if not specified
}

isVerified: { 
  type: Boolean, 
  default: false 
}                                             // For email verification

verificationCode: { 
  type: String, 
  default: null 
}                                             // For 2FA or email verification
```

**Why enum?**
- Ensures only valid roles are stored
- MongoDB rejects invalid values
- Prevents typos like "administator" or "organiser"

---

### 2. `server/src/routes/auth.js`

**Register Endpoint - Role Handling:**
```javascript
const { name, email, password, city, role } = req.body;

// Validate role
const validRoles = ["volunteer", "organizer", "admin"];
const userRole = validRoles.includes(role) ? role : "volunteer";
// If invalid role sent, defaults to "volunteer"

const user = await User.create({
  name,
  email,
  passwordHash,
  role: userRole,        // ← Save role
  city,
  points: 0,
  badges: ["b1"],
  isVerified: userRole === "admin" ? true : false  // Auto-verify admins
});

// Include role in JWT token
const token = jwt.sign({ 
  userId: user._id, 
  role: user.role        // ← NEW: Role in token
}, process.env.JWT_SECRET, { expiresIn: "7d" });

// Return role to frontend
res.json({
  token,
  user: {
    _id, name, email, city,
    role: user.role,      // ← NEW: Return role
    isVerified: user.isVerified
  }
});
```

**Login Endpoint:**
- Returns role in response
- Includes role in JWT token
- Frontend can check role to show different UI

---

### 3. `src/pages/Auth.jsx`

**Frontend Role Selection:**
```jsx
// State now includes role
const [formData, setFormData] = useState({
  name: '',
  email: '',
  password: '',
  role: 'volunteer'  // ← NEW: Default role
})

// Registration form includes role dropdown
{!isLogin && (
  <div className="form-group">
    <label htmlFor="role">Select Your Role</label>
    <select
      id="role"
      name="role"
      value={formData.role}
      onChange={handleChange}
    >
      <option value="volunteer">Volunteer</option>
      <option value="organizer">Event Organizer</option>
      <option value="admin">Administrator</option>
    </select>
  </div>
)}

// Send role to backend
const payload = {
  name: formData.name,
  email: formData.email,
  password: formData.password,
  role: formData.role    // ← NEW: Include role
}

// Show role in success message
alert(`Welcome ${data.user.name}!\nRole: ${data.user.role}`)
```

---

## 🧪 Testing Scenarios

### Test 1: Register as Volunteer
1. Go to http://localhost:5173
2. Click "Register"
3. Fill: Name="John", Email="john@test.com", Password="pass123", Role="Volunteer"
4. Click "Create Account"
5. Check MongoDB:
   ```bash
   mongosh
   db.users.find({ email: "john@test.com" })
   # Should show: role: "volunteer"
   ```

### Test 2: Register as Organizer
1. Go to http://localhost:5173
2. Click "Register"
3. Fill: Name="Sarah", Email="sarah@test.com", Password="pass123", Role="Event Organizer"
4. Click "Create Account"
5. Check MongoDB:
   ```bash
   db.users.find({ email: "sarah@test.com" })
   # Should show: role: "organizer"
   ```

### Test 3: Register as Admin
1. Go to http://localhost:5173
2. Click "Register"
3. Fill: Name="Admin", Email="admin@test.com", Password="pass123", Role="Administrator"
4. Click "Create Account"
5. Check MongoDB:
   ```bash
   db.users.find({ email: "admin@test.com" })
   # Should show: role: "admin", isVerified: true (auto-verified)
   ```

### Test 4: Login with Role
1. Click "Login"
2. Enter: Email="sarah@test.com", Password="pass123"
3. Click "Sign In"
4. Success message shows: "Role: organizer"
5. Check localStorage:
   ```javascript
   // F12 → Application → Local Storage
   localStorage.getItem('user')
   // Should show: { role: "organizer" }
   ```

---

## 🔐 Security Implications

### Role-Based Access Control (RBAC)
Currently, users can SELECT any role when registering. In production:

```javascript
// Backend should validate role assignment:
router.post("/register", async (req, res) => {
  let { role } = req.body;
  
  // Only allow volunteer role for registrations
  // Admins & organizers created by existing admins only
  if (role !== "volunteer") {
    role = "volunteer";  // Force volunteer role
  }
  
  // ... rest of code
});
```

**Recommended future improvements:**
- Only allow "volunteer" role in public registration
- Admin creates organizer accounts after verification
- Admin creates admin accounts only (restricted)
- Email verification before account activation
- Role-based route protection on backend

---

## 📝 API Endpoints Updated

### Register with Role
```bash
POST http://localhost:4000/api/auth/register
Content-Type: application/json

{
  "name": "Sarah",
  "email": "sarah@test.com",
  "password": "password123",
  "role": "organizer"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Sarah",
    "email": "sarah@test.com",
    "role": "organizer",
    "city": "Hyderabad",
    "isVerified": false
  }
}
```

### Login (Role Returned)
```bash
POST http://localhost:4000/api/auth/login
Content-Type: application/json

{
  "email": "sarah@test.com",
  "password": "password123"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Sarah",
    "email": "sarah@test.com",
    "role": "organizer",        ← Role included
    "city": "Hyderabad",
    "isVerified": false
  }
}
```

---

## 🎯 JWT Token Structure

### Token Payload Now Includes Role
```javascript
// Before:
jwt.sign({ userId: user._id }, secret)

// After:
jwt.sign({ 
  userId: user._id, 
  role: user.role     // ← NEW
}, secret)

// Decoded token:
{
  userId: "507f1f77bcf86cd799439011",
  role: "organizer",
  iat: 1709123456,      // issued at
  exp: 1709728256       // expires in 7 days
}
```

Backend can use role in token for route protection:
```javascript
// Middleware example (to implement):
function requireAdmin(req, res, next) {
  if (req.user.role !== "admin") {
    return res.status(403).json({ message: "Admin access required" });
  }
  next();
}

app.delete("/api/users/:id", requireAdmin, deleteUserHandler);
```

---

## 📊 Sample Data

After testing all three roles:

```bash
mongosh
use eco-volunteer-match
db.users.find().pretty()
```

Output:
```json
[
  {
    "_id": ObjectId("..."),
    "name": "John Doe",
    "email": "john@test.com",
    "role": "volunteer",
    "isVerified": false,
    "city": "Hyderabad",
    "points": 0
  },
  {
    "_id": ObjectId("..."),
    "name": "Sarah Smith",
    "email": "sarah@test.com",
    "role": "organizer",
    "isVerified": false,
    "city": "Hyderabad",
    "points": 0
  },
  {
    "_id": ObjectId("..."),
    "name": "Admin User",
    "email": "admin@test.com",
    "role": "admin",
    "isVerified": true,        ← Auto-verified
    "city": "Hyderabad",
    "points": 0
  }
]
```

---

## ✅ Checklist Before Demo

- [ ] Backend updated with role field
- [ ] Frontend has role dropdown
- [ ] Test register as volunteer
- [ ] Test register as organizer
- [ ] Test register as admin
- [ ] Test login shows role
- [ ] MongoDB shows role field
- [ ] localStorage shows user role
- [ ] Token is generated with role

---

## 🚀 Next Steps (Future Features)

1. **Admin Dashboard:**
   - View all users by role
   - Promote/demote users
   - Manage organizers

2. **Organizer Dashboard:**
   - Create events
   - Manage volunteers
   - View registrations

3. **Volunteer Dashboard:**
   - View available events
   - Register for events
   - Track points/badges

4. **Role-Based Routes:**
   - `/admin/*` - Admin only
   - `/organizer/*` - Organizer+ only
   - `/volunteer/*` - Volunteer+ only

5. **Permissions Matrix:**
   ```
   Feature              | Volunteer | Organizer | Admin
   ─────────────────────┼───────────┼───────────┼──────
   Browse Events        |     ✓     |     ✓     |   ✓
   Register for Event   |     ✓     |     ✓     |   ✓
   Create Events        |           |     ✓     |   ✓
   Manage Organizers    |           |           |   ✓
   Delete Users         |           |           |   ✓
   View All Data        |           |           |   ✓
   ```

---

**Your system now has a complete role-based user management system! 🎉**
