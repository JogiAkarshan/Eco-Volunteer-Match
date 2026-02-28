# Person 8: UPDATED - Database & Authentication with Roles

## Updated Code Files Person 8 Should Know

Now that we've added the role system, Person 8's code has MORE to explain!

---

## 📝 Person 8's Updated Code Explanation

### **File 1: `server/src/models/User.js` (UPDATED)**

```javascript
const UserSchema = new mongoose.Schema(
  {
    name: { type: String, required: true, default: "Volunteer" },
    email: { type: String, required: true, unique: true, index: true },
    passwordHash: { type: String, required: true },
    
    // ✨ NEW: Role field with three options
    role: { 
      type: String, 
      enum: ["volunteer", "organizer", "admin"],  // Only these allowed
      default: "volunteer"                        // Default if not provided
    },
    
    city: { type: String, default: "Hyderabad" },
    points: { type: Number, default: 0 },
    badges: { type: [String], default: [] },

    // ✨ NEW: Verification tracking
    isVerified: { type: Boolean, default: false },
    verificationCode: { type: String, default: null },

    joinedEventIds: [{ type: mongoose.Schema.Types.ObjectId, ref: "Event" }],
    interests: { type: [String], default: [] }
  },
  { timestamps: true }
);

module.exports = mongoose.model("User", UserSchema);
```

**What to Say:**

```
"We updated the User schema to include a role field for three user types.

NEW FIELD: role
- type: String
- enum: ["volunteer", "organizer", "admin"]
  This means ONLY these three values are allowed
  MongoDB rejects any other value
- default: "volunteer"
  If user doesn't specify role, defaults to volunteer

Why enum?
- Prevents typos and invalid data
- Ensures consistency across database
- Makes queries easier: db.users.find({ role: "organizer" })

NEW FIELD: isVerified
- type: Boolean
- default: false
- Tracks if email or user is verified
- Admins are automatically set to true

NEW FIELD: verificationCode
- For 2-factor authentication or email verification
- Stored temporarily, then deleted after verification

Example documents with different roles:

VOLUNTEER:
{
  name: "John",
  email: "john@test.com",
  role: "volunteer",      ← Regular user
  isVerified: false
}

ORGANIZER:
{
  name: "Sarah",
  email: "sarah@test.com",
  role: "organizer",      ← Can create events
  isVerified: false
}

ADMIN:
{
  name: "Admin",
  email: "admin@test.com",
  role: "admin",          ← System manager
  isVerified: true        ← Auto-verified
}

This schema now supports role-based access control (RBAC)."
```

---

### **File 2: `server/src/routes/auth.js` (UPDATED)**

#### **Register Endpoint - NEW Role Handling**

```javascript
router.post("/register", async (req, res, next) => {
  try {
    // ✨ NEW: Extract role from request
    const { name, email, password, city, role } = req.body;
    
    if (!email || !password) 
      return res.status(400).json({ message: "email/password required" });

    const exists = await User.findOne({ email: email.toLowerCase() });
    if (exists) return res.status(409).json({ message: "Email already exists" });

    // ✨ NEW: Validate role
    const validRoles = ["volunteer", "organizer", "admin"];
    const userRole = validRoles.includes(role) ? role : "volunteer";
    // If someone sends an invalid role, force it to "volunteer"

    const passwordHash = await bcrypt.hash(password, 10);
    const user = await User.create({
      name: name || "Volunteer",
      email: email.toLowerCase(),
      passwordHash,
      role: userRole,  // ✨ NEW: Save chosen role
      city: city || "Hyderabad",
      points: 0,
      badges: ["b1"],
      interests: [],
      isVerified: userRole === "admin" ? true : false  // ✨ NEW: Auto-verify admins
    });

    // ✨ NEW: Include role in JWT token
    const token = jwt.sign(
      { userId: user._id, role: user.role },  // Include role here
      process.env.JWT_SECRET,
      { expiresIn: "7d" }
    );

    res.json({ 
      token, 
      user: { 
        _id: user._id, 
        name: user.name, 
        email: user.email, 
        city: user.city,
        role: user.role,              // ✨ NEW: Return role
        isVerified: user.isVerified   // ✨ NEW: Return verification status
      } 
    });
  } catch (error) {
    next(error);
  }
});
```

**What to Say:**

```
"The register endpoint now handles roles.

Step 1: Extract role from request
- Frontend sends: { name, email, password, role: "organizer" }
- Backend receives and validates it

Step 2: Validate role
- Check if role is "volunteer", "organizer", or "admin"
- If invalid (like "superuser"), default to "volunteer"
- This prevents data corruption from bad inputs

Step 3: Create user with role
- new User({ ..., role: userRole })
- Role is saved to MongoDB

Step 4: Special handling for ADMINS
- If role === "admin": isVerified = true
- Only admins are auto-verified
- Other roles: isVerified = false (need email verification)

Step 5: Generate token WITH ROLE
- jwt.sign({ userId, role }, secret)
- Token now includes role!
- Backend can check token to know user type

Step 6: Return user data with role
- Response includes: role, isVerified
- Frontend knows user type immediately

Security note:
- Frontend can select any role
- In production, restrict role assignment:
  * Only "volunteer" allowed in public registration
  * Admins create "organizer" accounts after verification
  * Only super-admin creates admin accounts

Flow example (Organizer registration):
Frontend sends:
  POST /api/auth/register
  { name: 'Sarah', email: 'sarah@test.com', password: 'pass123', role: 'organizer' }

Backend:
  1. Validates role: 'organizer' ✓ Valid
  2. Hashes password
  3. Creates user: { name, email, passwordHash, role: 'organizer' }
  4. Generates token: jwt.sign({ userId, role: 'organizer' })
  5. Returns: { token, user: { name, email, role: 'organizer' } }

Frontend:
  1. Stores token in localStorage
  2. Stores user data including role
  3. Shows: 'Welcome Sarah! Role: organizer'
"
```

---

#### **Login Endpoint - NEW Role in Token**

```javascript
router.post("/login", async (req, res, next) => {
  try {
    const { email, password } = req.body;
    if (!email || !password) 
      return res.status(400).json({ message: "email/password required" });

    const user = await User.findOne({ email: email.toLowerCase() });
    if (!user) return res.status(401).json({ message: "Invalid credentials" });

    const ok = await bcrypt.compare(password, user.passwordHash);
    if (!ok) return res.status(401).json({ message: "Invalid credentials" });

    // ✨ NEW: Include role in JWT token
    const token = jwt.sign(
      { userId: user._id, role: user.role },  // Include role here
      process.env.JWT_SECRET,
      { expiresIn: "7d" }
    );

    res.json({ 
      token, 
      user: { 
        _id: user._id, 
        name: user.name, 
        email: user.email, 
        city: user.city,
        role: user.role,              // ✨ NEW: Return role
        isVerified: user.isVerified   // ✨ NEW: Return verification status
      } 
    });
  } catch (error) {
    next(error);
  }
});
```

**What to Say:**

```
"Login endpoint now returns role.

Process:
1. Find user by email
2. Verify password with bcrypt.compare()
3. ✨ NEW: Generate token WITH ROLE
   - Token includes: { userId, role }
4. ✨ NEW: Return role in response
5. Frontend knows user type immediately

Why include role in token?
- Backend can verify role without database lookup
- Speed: Token decode is faster than DB query
- Security: Token is signed, can't be forged
- Middleware can check role directly from token

Example token payload:
{
  userId: "507f1f77bcf86cd799439011",
  role: "organizer",      ← NEW
  iat: 1709123456,
  exp: 1709728256
}

Frontend can extract role:
- JWT decoder: jwt_decode(token)
- Show different dashboard based on role
- Organizer sees: Create Event button
- Admin sees: Manage Users button
- Volunteer sees: Browse Events button

After login:
Frontend stores:
- Token in localStorage (for API requests)
- User role in localStorage (for UI decisions)

Backend can use role for:
- Route protection: if (req.user.role !== 'admin') reject
- Query filtering: find events created by organizer
- Logging/audit: track what each role does
"
```

---

## 📊 What Changed Summary

| Aspect | Before | After |
|--------|--------|-------|
| **User Types** | 1 (Volunteer) | 3 (Volunteer, Organizer, Admin) |
| **Role Field** | ❌ None | ✅ enum["volunteer", "organizer", "admin"] |
| **Default Role** | N/A | "volunteer" |
| **Auto-verified** | ❌ No | ✅ Admins only |
| **JWT Token** | { userId } | { userId, role } |
| **Response Data** | No role | Includes role + isVerified |

---

## 🧪 Testing Scenarios for Person 8

**Scenario 1: Register Volunteer**
- Send: `{ role: "volunteer" }`
- Check: MongoDB shows `role: "volunteer"`
- Check: Token includes `role: "volunteer"`

**Scenario 2: Register Organizer**
- Send: `{ role: "organizer" }`
- Check: MongoDB shows `role: "organizer"`
- Check: Token includes `role: "organizer"`

**Scenario 3: Register Admin**
- Send: `{ role: "admin" }`
- Check: MongoDB shows `role: "admin", isVerified: true`
- Check: Token includes `role: "admin"`

**Scenario 4: Send Invalid Role**
- Send: `{ role: "superuser" }`
- Check: MongoDB shows `role: "volunteer"` (defaults)
- Proves: Invalid roles are rejected

---

## 💡 Future Backend Work for Person 8

After basic role system, add:

1. **Role Middleware:**
```javascript
function requireAdmin(req, res, next) {
  if (req.user.role !== "admin") {
    return res.status(403).json({ message: "Admin access required" });
  }
  next();
}
```

2. **Protected Routes:**
```javascript
app.delete("/api/users/:id", requireAdmin, deleteUserHandler);
app.post("/api/organizers", requireAdmin, createOrganizerHandler);
```

3. **Role Validation:**
```javascript
const roleHierarchy = {
  volunteer: 0,
  organizer: 1,
  admin: 2
};

function requireMinRole(minRole) {
  return (req, res, next) => {
    if (roleHierarchy[req.user.role] < roleHierarchy[minRole]) {
      return res.status(403).json({ message: "Insufficient permissions" });
    }
    next();
  };
}
```

---

**Person 8 now explains BOTH database schema AND role-based authentication! 🎯**
