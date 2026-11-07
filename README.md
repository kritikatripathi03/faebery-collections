# FÄBERY COLLECTIONS 🛍️ ![CI](https://github.com/kritikatripathi03/fabery-collections/actions/workflows/ci.yml/badge.svg)

A full stack MERN e-commerce application for a premium fashion brand. Built with React, Tailwind CSS, Node.js, Express and MongoDB.

![FÄBERY Collections](https://images.unsplash.com/photo-1441986300917-64674bd600d8?w=1200&q=80)

## 🌐 Live Demo

👉 [faeberycollections.vercel.app](https://faeberycollections.vercel.app)

> **Demo account:** email: `demo@faeberycollections.com` / password: `demo123`

---

## ✨ Features

- 🔐 **Authentication** — Register and login with JWT based auth, persistent sessions
- 🛍️ **Product Catalog** — Browse products with search, filters by size/category/price, and sorting
- 📦 **Product Detail** — View product images, sizes, stock availability, add to cart or wishlist
- 🛒 **Cart** — Add, update quantity, remove items, view order total
- ❤️ **Wishlist** — Save products for later
- 📋 **Orders** — Place orders with shipping address, view order history with status
- 💳 **Checkout** — Full checkout flow with Razorpay payment gateway
- 👤 **Profile** — View account details and complete order history with status badges
- 🛠️ **Admin Dashboard** — Manage products (with Cloudinary image upload), orders and users
- 📧 **Email Notifications** — Order confirmation emails via Resend
- 📱 **Responsive** — Works on mobile, tablet and desktop

---

## 🛠️ Tech Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| React 18 | UI library |
| Tailwind CSS | Styling |
| React Router DOM | Client side routing |
| Axios | API calls with JWT interceptor |
| Context API | Global auth state management |

### Backend
| Technology | Purpose |
|------------|---------|
| Node.js | Runtime |
| Express.js | Server framework |
| MongoDB Atlas | Cloud database |
| Mongoose | ODM with schema validation |
| JWT (HS256) | Stateless authentication |
| bcryptjs | Password hashing (10 salt rounds) |
| Razorpay | Payment gateway |
| Cloudinary | Image storage and delivery |
| Resend | Transactional email API |
| Multer | File upload handling |

### DevOps
| Service | Purpose |
|---------|---------|
| Docker + Docker Compose | Containerization |
| Vercel | Frontend hosting |
| Render | Backend hosting |
| MongoDB Atlas | Cloud database |
| GitHub | Version control |
| GitHub Actions | CI pipeline: backend dependency install, frontend lint and build on every push and pull request |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        Client                            │
│              React + Tailwind (Vercel)                   │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  Pages   │  │  Context │  │  Axios   │  │ Router │  │
│  │ Products │  │   Auth   │  │Interceptor│  │Protected│ │
│  │   Cart   │  │  (JWT)   │  │  (Token) │  │ Routes │  │
│  │ Wishlist │  └──────────┘  └──────────┘  └────────┘  │
│  │  Admin   │                                            │
│  └──────────┘                                            │
└────────────────────────┬────────────────────────────────┘
                         │ HTTPS (Axios + JWT Bearer)
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Express API (Render)                   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │               Middleware Chain                    │   │
│  │  CORS → express.json → authMiddleware → routes   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  /auth   │  │ /products│  │  /cart   │  │/orders │  │
│  │ /wishlist│  │  /admin  │  │ /payment │  │        │  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              External Services                    │   │
│  │   Razorpay (payments) │ Cloudinary (images)      │   │
│  │   Resend (email)                                  │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────┘
                         │ Mongoose ODM
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  MongoDB Atlas (Cloud)                    │
│                                                          │
│  ┌────────┐ ┌─────────┐ ┌──────┐ ┌──────────┐ ┌─────┐ │
│  │ users  │ │products │ │carts │ │wishlists │ │orders│ │
│  └────────┘ └─────────┘ └──────┘ └──────────┘ └─────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## 🧠 Design Decisions

These are the engineering decisions made during development and the reasoning behind them:

### 1. JWT over Session-based Auth
Chose stateless JWT (HS256) over server-side sessions because:
- Sessions require shared storage across servers — doesn't scale horizontally
- JWT is self-contained — the server verifies the signature without a DB lookup on every request
- Tradeoff: tokens cannot be invalidated before expiry (mitigated by short expiry + logout clearing localStorage)

### 2. Snapshot Pricing at Order Time
When an order is placed, the product price is copied into the order document rather than referenced:
```js
price: item.product.price  // stored at purchase time
```
This means if a product price changes later, historical orders remain accurate. A reference to the product would show the wrong price in order history.

### 3. Populate vs Embedding for Cart Items
Cart items store only the product `ObjectId`, not the full product details:
```js
product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' }
```
Then `.populate('items.product')` fetches details at read time. This avoids data duplication — if a product's name or image changes, the cart always shows the latest version. Embedding would require updating every cart that contains the product.

### 4. RBAC with Double Protection
Admin routes are protected on both frontend and backend:
- **Frontend:** `AdminRoute` component redirects non-admins before the page loads
- **Backend:** `adminMiddleware` checks `req.user.isAdmin` on every admin API call

Frontend protection is UX — backend protection is security. Never trust the frontend alone.

### 5. Cloudinary over Local Storage for Images
Product images are uploaded to Cloudinary instead of stored on the server because:
- Server disk space is limited and costly
- Render (our host) has an ephemeral filesystem — local files are wiped on redeploy
- Cloudinary provides automatic optimization, CDN delivery and transformations

### 6. Resend over Nodemailer + Gmail SMTP
Render free tier blocks outgoing SMTP ports (465/587). Resend uses HTTP API instead — works everywhere, better deliverability, no port issues. Also a more production-appropriate choice than raw Gmail SMTP.

### 7. FormData for Product Upload (not JSON)
Product creation/edit uses `multipart/form-data` instead of JSON because files cannot be sent as JSON. Multer middleware handles the file, passes it to Cloudinary, and the returned URL is stored in MongoDB.

### 8. Docker Multi-stage Build for Frontend
The frontend Dockerfile uses two stages:
- **Stage 1 (builder):** Node Alpine — installs dependencies, runs `vite build`
- **Stage 2 (nginx):** Copies only the built `dist/` folder into Nginx

This keeps the final image small — no Node.js runtime in production, just static files served by Nginx.

### 9. CI Pipeline with GitHub Actions
Every push and pull request to `main` runs two parallel jobs on Ubuntu:
- **Backend:** `npm ci` for a reproducible install from the lockfile
- **Frontend:** `npm ci`, ESLint, then `vite build`

Running on Linux also catches case-sensitivity bugs that pass on Windows and macOS, such as the `Utils` vs `utils` import that broke the first Render deploy. The pipeline does not yet run API tests, so runtime regressions in route handlers are not caught. Adding Jest and Supertest is the next step.

---

## 📁 Project Structure

```
fabery-collections/
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   └── axios.js          # Axios instance with JWT interceptor
│   │   ├── components/
│   │   │   ├── admin/            # Admin dashboard pages
│   │   │   ├── Navbar.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Products.jsx
│   │   │   ├── ProductCard.jsx
│   │   │   ├── ProductProfile.jsx
│   │   │   ├── Cart.jsx
│   │   │   ├── Wishlist.jsx
│   │   │   ├── Checkout.jsx
│   │   │   ├── Profile.jsx
│   │   │   ├── About.jsx
│   │   │   ├── ProtectedRoute.jsx
│   │   │   └── AdminRoute.jsx
│   │   ├── context/
│   │   │   └── AuthContext.jsx   # Global auth state
│   │   └── App.jsx
│   ├── Dockerfile                # Multi-stage build
│   ├── nginx.conf                # Nginx config with /api proxy
│   └── .dockerignore
│
├── backend/
│   ├── config/
│   │   ├── db.js                 # MongoDB connection
│   │   └── cloudinary.js        # Cloudinary + Multer config
│   ├── middleware/
│   │   ├── authMiddleware.js     # JWT protect middleware
│   │   └── adminMiddleware.js    # RBAC admin check
│   ├── models/
│   │   ├── User.js
│   │   ├── Product.js
│   │   ├── Cart.js
│   │   ├── Wishlist.js
│   │   └── Order.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── products.js
│   │   ├── cart.js
│   │   ├── wishlist.js
│   │   ├── orders.js
│   │   ├── payments.js
│   │   └── admin.js
│   ├── utils/
│   │   └── sendEmail.js          # Resend email helper
│   ├── seeder.js                 # Seed dummy products
│   ├── Dockerfile
│   └── server.js
│
├── docker-compose.yml
└── README.md
```

---

## 🔌 API Routes

### Auth
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| POST | `/api/auth/register` | Public | Register new user |
| POST | `/api/auth/login` | Public | Login, returns JWT |
| GET | `/api/auth/profile` | Private | Get logged in user |

### Products
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| GET | `/api/products` | Public | Get all products |
| GET | `/api/products/:id` | Public | Get single product |
| GET | `/api/products/category/:category` | Public | Get by category |

### Cart
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| GET | `/api/cart` | Private | Get user's cart |
| POST | `/api/cart/add` | Private | Add item |
| PUT | `/api/cart/update/:itemId` | Private | Update quantity |
| DELETE | `/api/cart/remove/:itemId` | Private | Remove item |
| DELETE | `/api/cart/clear` | Private | Clear cart |

### Wishlist
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| GET | `/api/wishlist` | Private | Get wishlist |
| POST | `/api/wishlist/add` | Private | Add item |
| DELETE | `/api/wishlist/remove/:itemId` | Private | Remove item |
| DELETE | `/api/wishlist/clear` | Private | Clear wishlist |

### Orders
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| POST | `/api/orders` | Private | Place order (COD) |
| GET | `/api/orders/myorders` | Private | Get my orders |
| GET | `/api/orders/:id` | Private | Get single order |

### Payments
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| POST | `/api/payment/create-order` | Private | Create Razorpay order |
| POST | `/api/payment/verify` | Private | Verify and place order |

### Admin
| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| GET | `/api/admin/products` | Admin | All products |
| POST | `/api/admin/products` | Admin | Add product + image |
| PUT | `/api/admin/products/:id` | Admin | Edit product |
| DELETE | `/api/admin/products/:id` | Admin | Delete product |
| GET | `/api/admin/orders` | Admin | All orders |
| PUT | `/api/admin/orders/:id` | Admin | Update order status |
| GET | `/api/admin/users` | Admin | All users |
| DELETE | `/api/admin/users/:id` | Admin | Delete user |

---

## 🚀 Running Locally

### Prerequisites
- Node.js 18+
- Docker (optional)
- MongoDB Atlas account or local MongoDB

### Without Docker

**Backend:**
```bash
cd backend
npm install
```

Create `backend/.env`:
```
PORT=5000
MONGO_URI=your_mongodb_uri
JWT_SECRET=your_jwt_secret
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
RESEND_API_KEY=your_resend_key
RAZORPAY_KEY_ID=your_razorpay_key
RAZORPAY_KEY_SECRET=your_razorpay_secret
```

```bash
npm run seed   # seed dummy products
npm run dev    # start server on port 5000
```

**Frontend:**
```bash
cd frontend
npm install
```

Create `frontend/.env`:
```
VITE_API_URL=http://localhost:5000/api
```

```bash
npm run dev    # start on port 5173
```

### With Docker
```bash
# Create root .env with all backend variables (see above)
docker-compose up --build
```

- Frontend: `http://localhost:80`
- Backend: `http://localhost:5000`

---

## 🔮 Upcoming / In Progress

- [ ] Input validation with Zod
- [ ] Rate limiting and Helmet security headers
- [ ] httpOnly cookie based JWT (replacing localStorage)
- [ ] Razorpay webhook for missed payment reconciliation
- [x] GitHub Actions CI pipeline (lint and build)
- [ ] Extend CI with API tests (Jest + Supertest)- [ ] GitHub Actions CI/CD pipeline
- [ ] Product reviews and ratings
- [ ] Pagination

---

## 👩‍💻 Author

Built by [Kritika Tripathi](https://github.com/kritikatripathi03)

---

## 📄 License

MIT
