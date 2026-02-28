# Quick Test: User Roles System

## 🧪 Test All Three Roles (5 minutes)

### Test 1: Register as VOLUNTEER

**Step 1:** Open http://localhost:5173  
**Step 2:** Click "Register" tab  
**Step 3:** Fill form:
```
Name: John Volunteer
Email: john@test.com
Password: password123
Role: Volunteer (selected)
```
**Step 4:** Click "Create Account"  
**Step 5:** You see: "Welcome John Volunteer! Role: volunteer"  

**Verify in MongoDB:**
```bash
mongosh
use eco-volunteer-match
db.users.findOne({ email: "john@test.com" })
```
Should show:
```json
{
  name: "John Volunteer",
  email: "john@test.com",
  role: "volunteer",
  isVerified: false
}
```

---

### Test 2: Register as ORGANIZER

**Step 1:** Clear form (switch to Login then back to Register)  
**Step 2:** Fill form:
```
Name: Sarah Organizer
Email: sarah@test.com
Password: password123
Role: Event Organizer (selected)
```
**Step 3:** Click "Create Account"  
**Step 4:** You see: "Welcome Sarah Organizer! Role: organizer"  

**Verify in MongoDB:**
```bash
db.users.findOne({ email: "sarah@test.com" })
```
Should show:
```json
{
  name: "Sarah Organizer",
  email: "sarah@test.com",
  role: "organizer",
  isVerified: false
}
```

---

### Test 3: Register as ADMIN

**Step 1:** Clear form again  
**Step 2:** Fill form:
```
Name: Admin User
Email: admin@test.com
Password: password123
Role: Administrator (selected)
```
**Step 3:** Click "Create Account"  
**Step 4:** You see: "Welcome Admin User! Role: admin"  

**Verify in MongoDB:**
```bash
db.users.findOne({ email: "admin@test.com" })
```
Should show:
```json
{
  name: "Admin User",
  email: "admin@test.com",
  role: "admin",
  isVerified: true  ← Auto-verified! Only admins are auto-verified
}
```

---

### Test 4: Login Test (Role in Response)

**Step 1:** Click "Login" tab  
**Step 2:** Enter:
```
Email: sarah@test.com
Password: password123
```
**Step 3:** Click "Sign In"  
**Step 4:** You see: "Login successful! Welcome Sarah Organizer! Role: organizer"  

**Check localStorage:**
- Press F12 (Developer Tools)
- Go to: Application → Local Storage → http://localhost:5173
- Find key: `user`
- Value should show: `{"name":"Sarah Organizer",...,"role":"organizer"}`

---

### Test 5: View All Users with Roles

**Step 1:** Open terminal/MongoDB Compass  
**Step 2:** Run:
```bash
mongosh
use eco-volunteer-match
db.users.find().pretty()
```

**Expected output:**
```json
[
  {
    "_id": ObjectId("..."),
    "name": "John Volunteer",
    "email": "john@test.com",
    "role": "volunteer",
    "isVerified": false,
    "city": "Hyderabad",
    "points": 0,
    "badges": ["b1"],
    "createdAt": ISODate("..."),
    "updatedAt": ISODate("...")
  },
  {
    "_id": ObjectId("..."),
    "name": "Sarah Organizer",
    "email": "sarah@test.com",
    "role": "organizer",
    "isVerified": false,
    "city": "Hyderabad",
    "points": 0,
    "badges": ["b1"],
    "createdAt": ISODate("..."),
    "updatedAt": ISODate("...")
  },
  {
    "_id": ObjectId("..."),
    "name": "Admin User",
    "email": "admin@test.com",
    "role": "admin",
    "isVerified": true,     ← Auto-verified!
    "city": "Hyderabad",
    "points": 0,
    "badges": ["b1"],
    "createdAt": ISODate("..."),
    "updatedAt": ISODate("...")
  }
]
```

✅ **All three roles present!**
✅ **Admin is auto-verified!**
✅ **Role field saved in database!**

---

### Test 6: Test Invalid Role (Should Default to Volunteer)

**Using curl/API client:**
```bash
curl -X POST http://localhost:4000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@test.com",
    "password": "password123",
    "role": "invalid_role"
  }'
```

**Response:**
```json
{
  "user": {
    "role": "volunteer"  ← Defaults to volunteer, not invalid_role!
  }
}
```

✅ **Invalid roles are automatically defaulted to volunteer!**

---

## 📊 Test Verification Checklist

After all tests, verify:

```
✓ Volunteer registered successfully
✓ Organizer registered successfully
✓ Admin registered successfully
✓ All three roles visible in MongoDB
✓ Admin is auto-verified
✓ Login returns role in response
✓ localStorage stores user role
✓ Dropdown shows all three options
✓ Cannot select invalid role (defaults to volunteer)
✓ Success messages show role name
```

---

## 🐛 If Something Doesn't Work

### Issue: Dropdown not showing
**Fix:** Clear browser cache (Ctrl+Shift+Delete), refresh

### Issue: Role not saving
**Fix:** Restart backend: `npm start` in server folder

### Issue: Database shows old data
**Fix:** Drop collection: `db.users.drop()` then register again

### Issue: Role shows as undefined
**Fix:** Make sure backend is running and frontend is calling http://localhost:4000

---

## 📋 Demo Script (For Teacher)

```
"As you can see, our system now supports three types of users:

1. VOLUNTEER (default)
   - Registers for environmental events
   - Earns points and badges
   [Show registered volunteer in MongoDB]

2. ORGANIZER (creates events)
   - Can create and manage events
   - Accepts volunteer registrations
   [Show registered organizer in MongoDB]

3. ADMIN (system manager)
   - Manages all users
   - Approves organizers
   - Auto-verified when registered
   [Show registered admin with isVerified: true]

All roles are stored in MongoDB with their role type.
When users login, we return their role so the frontend can
show different dashboards based on their role.

This enables role-based access control (RBAC) for the entire application."
```

---

**Testing complete! Your role system is working! 🎉**
