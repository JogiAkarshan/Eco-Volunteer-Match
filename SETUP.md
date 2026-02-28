# 🚀 Quick Setup Guide

## Prerequisites Check
✅ Node.js v18+ installed  
✅ MongoDB running locally  
✅ 2 terminal windows  

---

## Step 1: Start Backend (Terminal 1)

```bash
cd demo-app/server
npm install
npm start
```

**Expected Output:**
```
[DB] Connected
[API] running on http://localhost:4000
```

✅ **Backend is ready!** Keep this terminal running.

---

## Step 2: Start Frontend (Terminal 2)

```bash
cd demo-app
npm install
npm run dev
```

**Expected Output:**
```
➜  Local:   http://localhost:5173/
```

✅ **Frontend is ready!** Browser will auto-open.

---

## Step 3: Test Registration

1. **Wait 2.2s** → App auto-navigates to login
2. **Click "Register" tab**
3. **Fill in form:**
   - Name: `Test User`
   - Email: `test@example.com`
   - Password: `password123`
4. **Click "Create Account"**
5. **✅ Success!** User saved in MongoDB

---

## Step 4: Test Login

1. **Click "Login" tab**
2. **Enter:**
   - Email: `test@example.com`
   - Password: `password123`
3. **Click "Login"**
4. **✅ Success!** Authentication works

---

## ✨ What's Happening Behind the Scenes

| Action | What Happens |
|--------|--------------|
| Register | Frontend sends to `POST /api/auth/register` → Backend creates user with bcrypt password → MongoDB stores user |
| Login | Frontend sends to `POST /api/auth/login` → Backend verifies password → Returns JWT token |
| Token Save | Frontend stores JWT in localStorage for future requests |

---

## 🐛 If Something Doesn't Work

### Backend won't start?
```bash
# Is MongoDB running?
mongod

# Check if port 4000 is free
netstat -ano | findstr :4000
```

### Frontend won't connect to backend?
- Check backend terminal: Should say `[API] running on http://localhost:4000`
- Check browser F12 → Network tab for errors
- Check CORS is enabled in `server/.env`

### Registration fails?
- Look at backend terminal for error messages
- Check MongoDB: `mongosh` → `show databases`
- Restart both servers

### Email already exists error?
- That email is already registered in MongoDB
- Use a different email address

---

## 📁 File Structure

```
demo-app/
├── src/                    ← React frontend
├── server/                 ← Node.js backend
│   ├── src/
│   │   ├── index.js       ← Express server
│   │   ├── models/User.js ← MongoDB schema
│   │   └── routes/auth.js ← Register/Login endpoints
│   └── .env               ← MongoDB config
├── package.json
├── README.md
└── SETUP.md              ← You are here
```

---

## ✅ Checklist Before Demo

- [ ] MongoDB is running
- [ ] Backend started (`npm start` in server folder)
- [ ] Frontend started (`npm run dev` in demo-app folder)
- [ ] App loads on http://localhost:5173
- [ ] Splash screen shows (2.2s animation)
- [ ] Can register new user
- [ ] Can login with registered user
- [ ] No errors in browser F12 console

---

## 🎯 Demo Flow (3 minutes)

1. **Show splash screen** (2.2s)
2. **Show auth page** (Login/Register tabs)
3. **Register new user** (fill form, click button)
4. **Show success message** (user saved to MongoDB)
5. **Login with same credentials** (prove authentication works)
6. **Explain architecture** (Frontend→Backend→MongoDB)

---

## 📞 Quick Reference

| Command | What it does |
|---------|-------------|
| `cd server && npm start` | Start backend API |
| `npm run dev` | Start frontend dev server |
| `mongosh` | Open MongoDB shell to view data |
| `db.users.find()` | See all registered users |

---

**That's it! You're ready to demo.** 🎉
