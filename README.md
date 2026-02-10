{
  "name": "organic-basket-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "mongoose": "^7.0.3",
    "nodemailer": "^6.9.1"
  }
}
const mongoose = require("mongoose");

const connectDB = async () => {
  await mongoose.connect("mongodb://127.0.0.1:27017/organicBasket");
  console.log("MongoDB Connected");
};

module.exports = connectDB;
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  isVerified: { type: Boolean, default: false }
});

module.exports = mongoose.model("User", userSchema);
const mongoose = require("mongoose");

const productSchema = new mongoose.Schema({
  name: String,
  description: String,
  price: Number,
  category: String,
  stock: Number,
  image: String,
  rating: Number
});

module.exports = mongoose.model("Product", productSchema);
const express = require("express");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const User = require("../models/User");

const router = express.Router();

router.post("/register", async (req, res) => {
  const { name, email, password } = req.body;
  const hashed = await bcrypt.hash(password, 10);

  const user = await User.create({ name, email, password: hashed });
  res.json({ message: "Registered Successfully" });
});

router.post("/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user) return res.status(400).json({ message: "User not found" });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(400).json({ message: "Wrong password" });

  const token = jwt.sign({ id: user._id }, "secretkey");
  res.json({ token });
});

module.exports = router;
const express = require("express");
const Product = require("../models/Product");
const router = express.Router();

router.post("/add", async (req, res) => {
  const product = await Product.create(req.body);
  res.json(product);
});

router.get("/", async (req, res) => {
  const products = await Product.find();
  res.json(products);
});

module.exports = router;
const express = require("express");
const cors = require("cors");
const connectDB = require("./config/db");

const authRoutes = require("./routes/authRoutes");
const productRoutes = require("./routes/productRoutes");

const app = express();
connectDB();

app.use(cors());
app.use(express.json());

app.use("/api/auth", authRoutes);
app.use("/api/products", productRoutes);

app.listen(5000, () => console.log("Server running on port 5000"));
import React from "react";
import Login from "./pages/Login";
import Register from "./pages/Register";
import Products from "./pages/Products";

function App() {
  return (
    <>
      <Register />
      <Login />
      <Products />
    </>
  );
}

export default App;
import axios from "axios";

export default function Register() {
  const register = async () => {
    await axios.post("http://localhost:5000/api/auth/register", {
      name: "User",
      email: "user@gmail.com",
      password: "123456"
    });
    alert("Registered");
  };

  return <button onClick={register}>Register</button>;
}
import axios from "axios";

export default function Login() {
  const login = async () => {
    const res = await axios.post("http://localhost:5000/api/auth/login", {
      email: "user@gmail.com",
      password: "123456"
    });
    localStorage.setItem("token", res.data.token);
    alert("Logged In");
  };

  return <button onClick={login}>Login</button>;
}
import axios from "axios";
import { useEffect, useState } from "react";

export default function Products() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get("http://localhost:5000/api/products")
      .then(res => setProducts(res.data));
  }, []);

  return (
    <div>
      {products.map(p => (
        <h3 key={p._id}>{p.name} - ₹{p.price}</h3>
      ))}
    </div>
  );
}
