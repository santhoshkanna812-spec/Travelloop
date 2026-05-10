# Travelloop
// ==============================
// server.js
// ==============================

const express = require('express');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(express.static(path.join(__dirname, 'public')));

// MongoDB Connection
mongoose
  .connect('mongodb://127.0.0.1:27017/traveloop')
  .then(() => console.log('MongoDB Connected'))
  .catch((err) => console.log(err));

// ==============================
// Schema
// ==============================

const itinerarySchema = new mongoose.Schema({
  city: String,
  travelDate: Date,
  activities: [String],
  budget: Number,
  costBreakdown: Object,
});

const userSchema = new mongoose.Schema({
  username: String,
  itineraries: [itinerarySchema],
});

const User = mongoose.model('User', userSchema);

// ==============================
// Routes
// ==============================

// Create User
app.post('/api/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();

    res.status(201).json(user);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// Get User Itineraries
app.get('/api/users/:id/itineraries', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    res.json(user.itineraries);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// Add Itinerary
app.post('/api/users/:id/itineraries', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    user.itineraries.push(req.body);

    await user.save();

    res.status(201).json(user);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// Frontend Route
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// Start Server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

// ==============================
// public/index.html
// ==============================

const html = `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Traveloop</title>

  <style>

    body{
      font-family: Arial, sans-serif;
      background:#f4f4f4;
      margin:0;
      padding:20px;
    }

    h1{
      text-align:center;
      color:#333;
    }

    #app{
      max-width:600px;
      margin:auto;
      background:white;
      padding:20px;
      border-radius:10px;
      box-shadow:0 0 10px rgba(0,0,0,0.1);
    }

    form{
      display:flex;
      flex-direction:column;
      gap:10px;
    }

    input{
      padding:10px;
      border:1px solid #ccc;
      border-radius:5px;
    }

    button{
      padding:10px;
      background:#28a745;
      color:white;
      border:none;
      border-radius:5px;
      cursor:pointer;
    }

    button:hover{
      background:#218838;
    }

    .card{
      background:#fafafa;
      padding:15px;
      margin-top:15px;
      border-radius:5px;
      border-left:5px solid #28a745;
    }

  </style>
</head>

<body>

  <h1>Traveloop</h1>

  <div id="app">

    <h2>Create Your Itinerary</h2>

    <form id="itinerary-form">

      <input 
        type="text" 
        id="city" 
        placeholder="Enter City"
        required
      >

      <input 
        type="date" 
        id="travelDate"
        required
      >

      <input 
        type="text" 
        id="activities"
        placeholder="Activities (comma separated)"
        required
      >

      <input 
        type="number" 
        id="budget"
        placeholder="Enter Budget"
        required
      >

      <button type="submit">
        Save Itinerary
      </button>

    </form>

    <div id="itineraries"></div>

  </div>

<script>

const USER_ID = "YOUR_USER_ID";

document
  .getElementById("itinerary-form")
  .addEventListener("submit", async (e) => {

    e.preventDefault();

    const city = document.getElementById("city").value;

    const travelDate =
      document.getElementById("travelDate").value;

    const activities =
      document
        .getElementById("activities")
        .value
        .split(",");

    const budget =
      document.getElementById("budget").value;

    const response = await fetch(
      \`/api/users/\${USER_ID}/itineraries\`,
      {
        method:"POST",

        headers:{
          "Content-Type":"application/json"
        },

        body:JSON.stringify({
          city,
          travelDate,
          activities,
          budget
        })
      }
    );

    const data = await response.json();

    console.log(data);

    loadItineraries();

  });

async function loadItineraries(){

  const response = await fetch(
    \`/api/users/\${USER_ID}/itineraries\`
  );

  const itineraries = await response.json();

  const itinerariesDiv =
    document.getElementById("itineraries");

  itinerariesDiv.innerHTML =
    itineraries.map((itinerary) => {

      return \`
        <div class="card">

          <h3>\${itinerary.city}</h3>

          <p>
            <strong>Date:</strong>
            \${new Date(itinerary.travelDate)
              .toDateString()}
          </p>

          <p>
            <strong>Activities:</strong>
            \${itinerary.activities.join(", ")}
          </p>

          <p>
            <strong>Budget:</strong>
            ₹\${itinerary.budget}
          </p>

        </div>
      \`;

    }).join("");

}

loadItineraries();

</script>

</body>
</html>
`;

console.log(html);

// ==============================
// package.json
// ==============================

const packageJson = `
{
  "name": "traveloop",
  "version": "1.0.0",
  "description": "Travel Planning App",
  "main": "server.js",

  "scripts": {
    "start": "node server.js"
  },

  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.0.0"
  }
}
`;

console.log(packageJson);

// ==============================
// HOW TO RUN
// ==============================

/*

1. Create Folder

traveloop/

2. Inside Folder Create:

- server.js
- public/

3. Inside public folder create:

- index.html

4. Install Packages

npm install express mongoose

5. Start MongoDB

mongod

6. Run Server

node server.js

7. Open Browser

http://localhost:3000

*/
