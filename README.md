foster-care-platform/
├── backend/
│   ├── src/
│   │   ├── app.js
│   │   ├── server.js
│   │   ├── db.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── applicants.js
│   │   │   └── match.js
│   │   ├── middleware/
│   │   │   └── auth.js
│   ├── package.json
│   ├── .env.example
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   └── api.js
│   │   ├── pages/
│   │   │   ├── Login.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   └── ApplicantProfile.jsx
│   │   ├── components/
│   │   │   └── Navbar.jsx
│   │   ├── App.jsx
│   │   ├── main.jsx
│   ├── index.html
│   └── package.json
│
├── docker-compose.yml
├── README.md
└── .gitignore
{
  "name": "foster-backend",
  "version": "1.0.0",
  "main": "src/server.js",
  "scripts": {
    "dev": "node src/server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2"
  }
}
require("dotenv").config();
const express = require("express");
const cors = require("cors");

const authRoutes = require("./routes/auth");
const applicantRoutes = require("./routes/applicants");
const matchRoutes = require("./routes/match");

const app = express();

app.use(cors());
app.use(express.json());

app.get("/", (_, res) => res.json({ status: "API running" }));

app.use("/api/auth", authRoutes);
app.use("/api/applicants", applicantRoutes);
app.use("/api/match", matchRoutes);

module.exports = app;
const app = require("./app");

const PORT = process.env.PORT || 4000;
app.listen(PORT, () =>
  console.log(`Backend running on http://localhost:${PORT}`)
);
const express = require("express");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

const router = express.Router();

// Temporary in-memory users (replace with DB later)
const users = [];

router.post("/register", async (req, res) => {
  const { email, password, role } = req.body;
  const hash = await bcrypt.hash(password, 10);

  users.push({ email, password: hash, role });

  res.status(201).json({ message: "User registered" });
});

router.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = users.find((u) => u.email === email);

  if (!user) return res.status(401).json({ error: "Invalid credentials" });

  const ok = await bcrypt.compare(password, user.password);
  if (!ok) return res.status(401).json({ error: "Invalid credentials" });

  const token = jwt.sign(
    { email: user.email, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: "7d" }
  );

  res.json({ token });
});

module.exports = router;
const express = require("express");
const router = express.Router();

let applicants = [];

router.post("/", (req, res) => {
  applicants.push(req.body);
  res.status(201).json({ message: "Applicant saved" });
});

router.get("/", (_, res) => {
  res.json(applicants);
});

module.exports = router;
const express = require("express");
const router = express.Router();

router.post("/", (req, res) => {
  const { traits, criteria } = req.body;

  let score = 0;
  let count = 0;

  for (const key in criteria) {
    if (traits[key]) {
      score += Math.min(traits[key], criteria[key]) / criteria[key];
      count++;
    }
  }

  res.json({
    matchScore: Math.round((score / count) * 100 || 0)
  });
});

module.exports = router;
const express = require("express");
const router = express.Router();

router.post("/", (req, res) => {
  const { traits, criteria } = req.body;

  let score = 0;
  let count = 0;

  for (const key in criteria) {
    if (traits[key]) {
      score += Math.min(traits[key], criteria[key]) / criteria[key];
      count++;
    }
  }

  res.json({
    matchScore: Math.round((score / count) * 100 || 0)
  });
});

module.exports = router;

Front end (React + Vite)p
{
  "name": "foster-frontend",
  "private": true,
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "axios": "^1.6.8",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^6.25.1"
  }
}

import axios from "axios";

export const api = axios.create({
  baseURL: "http://localhost:4000/api"
});

import { BrowserRouter, Routes, Route } from "react-router-dom";
import Login from "./pages/Login";
import Dashboard from "./pages/Dashboard";
import ApplicantProfile from "./pages/ApplicantProfile";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<ApplicantProfile />} />
      </Routes>
    </BrowserRouter>
  );
}

import { useState } from "react";
import { api } from "../api/api";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const login = async () => {
    const res = await api.post("/auth/login", { email, password });
    localStorage.setItem("token", res.data.token);
    window.location.href = "/dashboard";
  };

  return (
    <div>
      <h2>Login</h2>
      <input placeholder="Email" onChange={e => setEmail(e.target.value)} />
      <input type="password" placeholder="Password" onChange={e => setPassword(e.target.value)} />
      <button onClick={login}>Login</button>
    </div>
  );
}

export default function Dashboard() {
  return (
    <div>
      <h1>Agency Dashboard</h1>
      <p>Welcome to the foster care platform</p>
    </div>
  );
}

import { useState } from "react";
import { api } from "../api/api";

export default function ApplicantProfile() {
  const [profile, setProfile] = useState({});

  const save = async () => {
    await api.post("/applicants", profile);
    alert("Profile saved");
  };

  return (
    <div>
      <h2>Applicant Profile</h2>
      <input placeholder="Full Name" onChange={e => setProfile({ ...profile, name: e.target.value })} />
      <button onClick={save}>Save</button>
    </div>
  );
}


