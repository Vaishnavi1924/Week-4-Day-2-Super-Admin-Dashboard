# Week-4-Day-2-Super-Admin-Dashboard
Super Admin │ ├── Total Users ├── Total Vendors ├── Total Customers ├── Total Stores ├── Total Products ├── Total Orders ├── Total Sales └── Store/Vendor Overview


Step 1 — Create Super Admin Analytics Controller

Create:

backend/controllers/adminController.js

Add:

const User = require("../models/User");
const Store = require("../models/Store");
const Product = require("../models/Product");
const Order = require("../models/Order");

const getAdminAnalytics = async (req, res) => {
  try {
    if (req.user.role !== "superadmin") {
      return res.status(403).json({
        message: "Only super admin can access this dashboard"
      });
    }

    const totalUsers =
      await User.countDocuments();

    const totalVendors =
      await User.countDocuments({
        role: "vendor"
      });

    const totalCustomers =
      await User.countDocuments({
        role: "customer"
      });

    const totalStores =
      await Store.countDocuments();

    const activeStores =
      await Store.countDocuments({
        isActive: true
      });

    const totalProducts =
      await Product.countDocuments();

    const orders =
      await Order.find();

    const totalOrders =
      orders.length;

    const totalSales =
      orders
        .filter(
          (order) =>
            order.paymentStatus === "paid"
        )
        .reduce(
          (total, order) =>
            total + order.totalAmount,
          0
        );

    const pendingOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "placed"
      ).length;

    const processingOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "processing"
      ).length;

    const shippedOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "shipped"
      ).length;

    const deliveredOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "delivered"
      ).length;

    const cancelledOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "cancelled"
      ).length;

    res.status(200).json({
      totalUsers,
      totalVendors,
      totalCustomers,
      totalStores,
      activeStores,
      totalProducts,
      totalOrders,
      totalSales,
      pendingOrders,
      processingOrders,
      shippedOrders,
      deliveredOrders,
      cancelledOrders
    });
  } catch (error) {
    res.status(500).json({
      message: error.message
    });
  }
};

module.exports = {
  getAdminAnalytics
};
Step 2 — Create Admin Routes

Create:

backend/routes/adminRoutes.js

Add:

const express = require("express");

const {
  getAdminAnalytics
} = require("../controllers/adminController");

const {
  protect,
  authorizeRoles
} = require("../middleware/authMiddleware");

const router = express.Router();

router.get(
  "/analytics",
  protect,
  authorizeRoles("superadmin"),
  getAdminAnalytics
);

module.exports = router;

The new API is:

GET /api/admin/analytics

Only:

role = superadmin

can access it.

Step 3 — Add Admin Route to server.js

At the top:

const adminRoutes =
  require("./routes/adminRoutes");

Then add:

app.use(
  "/api/admin",
  adminRoutes
);

So your route section becomes:

app.use("/api/auth", authRoutes);
app.use("/api/users", userRoutes);
app.use("/api/stores", storeRoutes);
app.use("/api/products", productRoutes);
app.use("/api/upload", uploadRoutes);
app.use("/api/cart", cartRoutes);
app.use("/api/orders", orderRoutes);
app.use("/api/payments", paymentRoutes);
app.use("/api/analytics", analyticsRoutes);
app.use("/api/admin", adminRoutes);
Step 4 — Create Super Admin Dashboard

Create:

frontend/src/pages/AdminDashboard.jsx

Add:

import { useEffect, useState } from "react";
import API from "../services/api";

function AdminDashboard() {
  const [analytics, setAnalytics] =
    useState(null);

  const [loading, setLoading] =
    useState(true);

  const [message, setMessage] =
    useState("");

  const fetchAnalytics = async () => {
    try {
      const response =
        await API.get(
          "/admin/analytics"
        );

      setAnalytics(response.data);
    } catch (error) {
      setMessage(
        error.response?.data?.message ||
        "Unable to load admin dashboard"
      );
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchAnalytics();
  }, []);

  if (loading) {
    return (
      <div className="analytics-page">
        <h1>Super Admin Dashboard</h1>
        <p>Loading dashboard...</p>
      </div>
    );
  }

  if (message) {
    return (
      <div className="analytics-page">
        <h1>Super Admin Dashboard</h1>
        <p>{message}</p>
      </div>
    );
  }

  return (
    <div className="analytics-page">
      <h1>Super Admin Dashboard</h1>

      <div className="analytics-grid">

        <div className="analytics-card">
          <h3>Total Users</h3>
          <p>{analytics.totalUsers}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Vendors</h3>
          <p>{analytics.totalVendors}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Customers</h3>
          <p>{analytics.totalCustomers}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Stores</h3>
          <p>{analytics.totalStores}</p>
        </div>

        <div className="analytics-card">
          <h3>Active Stores</h3>
          <p>{analytics.activeStores}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Products</h3>
          <p>{analytics.totalProducts}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Orders</h3>
          <p>{analytics.totalOrders}</p>
        </div>

        <div className="analytics-card">
          <h3>Total Sales</h3>
          <p>₹{analytics.totalSales}</p>
        </div>

      </div>

      <h2>Order Status</h2>

      <div className="analytics-grid">

        <div className="analytics-card">
          <h3>Placed</h3>
          <p>{analytics.pendingOrders}</p>
        </div>

        <div className="analytics-card">
          <h3>Processing</h3>
          <p>{analytics.processingOrders}</p>
        </div>

        <div className="analytics-card">
          <h3>Shipped</h3>
          <p>{analytics.shippedOrders}</p>
        </div>

        <div className="analytics-card">
          <h3>Delivered</h3>
          <p>{analytics.deliveredOrders}</p>
        </div>

        <div className="analytics-card">
          <h3>Cancelled</h3>
          <p>{analytics.cancelledOrders}</p>
        </div>

      </div>
    </div>
  );
}

export default AdminDashboard;
Step 5 — Add Route in App.jsx

You already have an Admin Dashboard route in your project, but make sure it looks like this.

Import:

import AdminDashboard from "./pages/AdminDashboard";

Then:

<Route
  path="/admin-dashboard"
  element={
    <ProtectedRoute
      allowedRoles={["superadmin"]}
    >
      <AdminDashboard />
    </ProtectedRoute>
  }
/>

This gives you two levels of protection:

Frontend
   ↓
ProtectedRoute
   ↓
role === superadmin
   ↓
Backend
   ↓
JWT verification
   ↓
authorizeRoles("superadmin")
   ↓
Admin API
Step 6 — Add Admin Link to Navbar

In Navbar.jsx, add:

{user.role === "superadmin" && (
  <Link to="/admin-dashboard">
    Admin Dashboard
  </Link>
)}

So the navigation changes according to the logged-in user's role:

CUSTOMER
Shop
Cart
My Orders

VENDOR
Dashboard
Products
Inventory
Store
Orders
Analytics

SUPER ADMIN
Admin Dashboard
Step 7 — Important: Creating a Super Admin

Your current registration deliberately prevents public users from choosing:

superadmin

That is good security.

Your registration code currently does:

const userRole =
  role === "vendor"
    ? "vendor"
    : "customer";

So even if someone sends:

{
  "role": "superadmin"
}

they will not become a super admin.

For development/testing, you can temporarily create a super-admin directly in MongoDB or create a protected server-side seed script.

For example, in MongoDB Compass you can have a user document like:

{
  "name": "Platform Admin",
  "email": "admin@example.com",
  "password": "HASHED_PASSWORD",
  "role": "superadmin",
  "storeId": null
}

Do not store a real password as plain text. Your application should store the bcrypt hash.

Step 8 — Test the Dashboard

Start backend:

cd backend
npm run dev

Start frontend:

cd frontend
npm run dev

Login as:

Super Admin

Then open:

/admin-dashboard

You should see:

       SUPER ADMIN DASHBOARD

┌──────────────┐ ┌──────────────┐
│ Total Users  │ │ Total Vendors│
│     50       │ │      10      │
└──────────────┘ └──────────────┘

┌──────────────┐ ┌──────────────┐
│  Customers   │ │ Total Stores │
│     40       │ │      10      │
└──────────────┘ └──────────────┘

┌──────────────┐ ┌──────────────┐
│   Products   │ │ Total Orders │
│     120      │ │      75      │
└──────────────┘ └──────────────┘

┌───────────────────────────────┐
│       Total Sales             │
│       ₹1,25,000               │
└───────────────────────────────┘

The actual values will come from your MongoDB database.

🔐 Multi-Tenant Structure Now

Your application now has a clear hierarchy:

                    PLATFORM
                       │
                 SUPER ADMIN
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Store A         Store B        Store C
        │              │              │
     Vendor A       Vendor B       Vendor C
        │              │              │
     Products       Products       Products
     Orders         Orders         Orders
Super Admin

Can see:

All stores
All vendors
All customers
All products
All orders
All sales
Vendor

Can see:

Only own store
Only own products
Only own orders
Only own sales
Customer

Can see:

Products
Own cart
Own orders
