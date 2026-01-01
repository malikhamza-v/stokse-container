# Stokse - Complete POS & Business Management System

A comprehensive Point of Sale (POS) and business management solution built with modern technologies. Stokse helps growing businesses manage sales, inventory, customers, employees, and appointments across multiple stores.

## Repository Overview

This monorepo contains three interconnected repositories:

| Repository | Description | Tech Stack |
|------------|-------------|------------|
| [stokse-container](#stokse-container) | Docker orchestration & deployment | Docker, Docker Compose |
| [stokse-server](#stokse-server) | REST API backend | Django 5, DRF, PostgreSQL |
| [stokse-client](#stokse-client) | Desktop & web client | Electron, React, TypeScript |

---

## Stokse Container

The container repository orchestrates the entire Stokse system using Docker and Docker Compose. It integrates the server and client as Git submodules for easy deployment and updates.

### Architecture

```
stokse-container/
├── docker-compose.yml      # Orchestrates all services
├── db_variables.env        # Database configuration
├── stokse-server/          # Git submodule (Backend API)
└── stokse-client/          # Git submodule (Desktop/Web Client)
```

### Services

| Service | Image | Ports | Description |
|---------|-------|-------|-------------|
| `database` | postgres:latest | 5432 | PostgreSQL database |
| `django` | django:stokse_server_api | 8000 | Django REST API |
| `react` | react:stokse_client_build | - | Web build output |

### Quick Start

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/malikhamza-v/stokse-container.git
cd stokse-container

# Configure environment
cp db_variables.env.example db_variables.env
# Edit db_variables.env with your secure password

cd stokse-server
cp .env.example .env
# Edit .env with your configuration

# Build and start
cd ..
docker-compose build
docker-compose up -d

# Check logs
docker-compose logs -f

# Create superuser (optional)
docker-compose exec django python manage.py createsuperuser
```

### Environment Variables

**db_variables.env:**
```env
POSTGRES_DB=stokse
POSTGRES_USER=stokse_user
POSTGRES_PASSWORD=your_secure_password
```

**stokse-server/.env:**
```env
DJANGO_SECRET_KEY=generate-a-secret-key
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1,your-domain.com

DB_NAME=stokse
DB_USER=stokse_user
DB_PASSWORD=your_secure_password
DB_HOST=database
DB_PORT=5432

# Email (optional)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=465
EMAIL_USE_SSL=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
DEFAULT_FROM_EMAIL=noreply@yourdomain.com
```

### Volume Mounts

| Host | Container | Purpose |
|------|-----------|---------|
| `./stokse-server` | `/django` | Live code mounting |
| `/var/www/django_static` | `/django/static` | Static files |
| `/var/www/django_media` | `/django/media` | Media uploads |
| `/var/www/stokse_client` | `/react/dist/web` | Built web app |

### Database Management

```bash
# Backup
docker-compose exec database pg_dump -U stokse_user stokse > db-bak/backup-$(date +%Y%m%d).sql

# Restore
docker-compose exec -T database psql -U stokse_user stokse < db-bak/backup-20240101.sql
```

### Updating Submodules

```bash
git submodule update --remote
docker-compose build
docker-compose up -d
```

---

## Stokse Server

Production-ready Django REST API backend providing comprehensive business management capabilities.

### Tech Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Framework | Django | 5.0.2 |
| API Toolkit | Django REST Framework | 3.14.0 |
| Database | PostgreSQL | 12+ |
| Server | Gunicorn | 23.0.0 |
| Authentication | JWT (PyJWT) | 2.8.0 |
| PDF Generation | pdfkit | 1.0.0 |
| Excel Handling | pandas, openpyxl | 2.2.1, 3.1.2 |

### Project Structure

```
stokse-server/
├── manage.py
├── requirements.txt
├── .env.example
├── Dockerfile
├── entrypoint.sh
├── stokse/                 # Django settings
│   ├── settings.py
│   ├── urls.py
│   └── utils.py           # JWT utilities
├── users/                 # Authentication
├── businesses/            # Business & Store management
├── products/              # Product catalog
├── orders/                # Order processing
├── employees/             # Employee management
├── services/              # Service catalog
├── appointments/          # Appointment booking
├── customer/              # Customer management
├── setting/               # Categories, brands, taxes, payment methods
├── activity/              # Activity logging
└── universal/             # Shared data (currencies)
```

### API Endpoints

#### Authentication (`/api/auth/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/register/` | User registration with signup code |
| POST | `/login/` | User login (returns JWT) |
| POST | `/logout/` | User logout |
| GET | `/user/` | Get current user |
| GET | `/managers-list/` | List managers (Admin) |
| POST | `/create-manager/` | Create manager (Admin) |
| PUT | `/update-manager/` | Update manager (Admin) |
| DELETE | `/remove-manager/` | Remove manager (Admin) |
| GET | `/get-all-stores/` | Get user's stores |

#### Business & Store (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/business/` | Create business |
| GET | `/store-list/` | List all stores |
| POST | `/store/` | Create store |
| PUT | `/store/<id>/` | Update store |
| GET | `/store/analytics/sales/` | Sales analytics |
| GET | `/store/analytics/customers/` | Customer analytics |
| GET | `/store/analytics/items-sold/` | Items sold analytics |
| GET | `/store/analytics/top-transactions/` | Top transactions |
| GET | `/store/analytics/top-customers/` | Top customers |
| GET | `/store/analytics/top-products/` | Top products |
| GET | `/store/analytics/overall/` | Overall statistics |

#### Products (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/products/` | List products (by store) |
| POST | `/products/` | Create product |
| GET | `/products/<id>/` | Get single product |
| PUT | `/products/<id>/` | Update product |
| DELETE | `/products/<id>/` | Delete product |
| POST | `/upload-products/` | Bulk upload (CSV/Excel) |

#### Orders (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/orders/` | List orders (paginated) |
| POST | `/orders/` | Create order |
| GET | `/order/<id>/` | Get order details |
| POST | `/order/<id>/send-invoice/` | Email invoice |

#### Services (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/services/` | List services |
| POST | `/service/` | Create service |
| PUT | `/service/<id>/` | Update service |
| DELETE | `/service/<id>/` | Delete service |

#### Appointments (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/appointments/` | List appointments |
| POST | `/appointments/` | Create appointment |
| PUT | `/appointments/<id>/` | Update appointment |
| PATCH | `/appointments/<id>/status/` | Update status |
| GET | `/appointments/check-availability/` | Check availability |

#### Employees (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/employee/` | List employees |
| POST | `/employee/` | Create employee |
| PUT | `/employee/<id>/` | Update employee |
| DELETE | `/employee/<id>/` | Delete employee |

#### Customers (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/customers/` | List customers |
| POST | `/customers/` | Create customer |
| PUT | `/customer/<id>/` | Update customer |
| DELETE | `/customer/<id>/` | Delete customer |

#### Settings (`/api/`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET/POST | `/category/` | List/create categories |
| GET/POST | `/brand/` | List/create brands |
| GET/POST | `/payment-method/` | List/create payment methods |
| GET/POST | `/tax/` | List/create taxes |

### Database Schema

#### Core Models

**User:**
- id, name, email (unique), password, phone_number (unique)
- is_superadmin, role (Admin/Manager)
- created_at

**Business:**
- id, admin (FK User), name, created_at

**Store:**
- id, business (FK), managers (M2M User), name, description
- city, country, address, phone, email, logo
- opening_time, closing_time, from_day, to_day
- currency (FK), is_active, created_at

**Product:**
- id, product_id (auto-generated), name, category (FK), brand (FK)
- description, cost_price, sale_price, stock_quantity
- enable_low_stock_notification, low_stock_level, reorder_quantity
- taxes (JSON), store (FK), created_at, updated_at

**Order:**
- id, items (JSON), sub_total, order_discount, tip, total
- payment_methods (JSON), tax (JSON), created_by (FK User)
- payment_status, status, store (FK), customer (FK)
- created_at, updated_at

**Employee:**
- id, store (FK), role (Admin/Manager/Staff), name, email, phone
- can_perform_services, address, date_of_birth, date_joined, is_active
- created_at

**Service:**
- id, service_id (auto-generated), name, category (FK)
- description, price, price_type, duration, team (M2M Employee)
- store (FK), created_at, updated_at

**Appointment:**
- id, customer (FK), store (FK), employee (FK), created_by (FK)
- date, start_time, frequency_value, frequency_unit, frequency_end_value
- services (JSON), total_price, total_duration
- payment_status, appointment_status, notes
- created_at, updated_at

**Customer:**
- id, store (FK), name, email, phone, created_at

### Authentication

- **JWT-based** with 30-day expiration
- **Device types:** Desktop (Bearer token), Mobile (HttpOnly cookies)
- **Roles:** Admin (full access), Manager (assigned stores), Staff

### Standalone Setup

```bash
# Clone
git clone https://github.com/malikhamza-v/stokse-server.git
cd stokse-server

# Virtual environment
python -m venv venv
source venv/bin/activate

# Install
pip install -r requirements.txt

# Configure
cp .env.example .env
# Edit .env

# Migrate (run in order!)
python manage.py migrate users
python manage.py migrate businesses
python manage.py migrate employees
python manage.py migrate setting
python manage.py migrate products
python manage.py migrate orders
python manage.py migrate customer
python manage.py migrate services
python manage.py migrate appointments
python manage.py migrate activity
python manage.py migrate universal

# Superuser
python manage.py createsuperuser

# Run
python manage.py runserver
```

### Management Commands

```bash
# Seed default store with dummy data
python manage.py default_store
```

Creates:
- User: `admin@milano.com` / `Password@123`
- Business: Milano Enterprises
- Store: Milano I
- 30 employees, 50 customers, 50 orders

---

## Stokse Client

Cross-platform desktop application and web client built with Electron and React.

### Tech Stack

| Category | Technology | Version |
|----------|------------|---------|
| Desktop Framework | Electron | 26.2.1 |
| UI Framework | React | 18.2.0 |
| Language | TypeScript | 5.2.2 |
| State Management | Redux Toolkit | 2.2.1 |
| Server State | React Query | 5.51.11 |
| Routing | React Router DOM | 6.16.0 |
| Styling | Tailwind CSS + DaisyUI | 3.4.1, 4.11.1 |
| Build Tools | Webpack, Vite | 5.99.4, 5.3.3 |
| HTTP Client | Axios | 1.6.7 |
| Charts | ApexCharts | 1.4.1 |
| Calendar | React Big Calendar | 1.13.1 |

### Project Structure

```
stokse-client/
├── .erb/                  # Electron React Boilerplate configs
├── assets/                # Icons and static resources
├── src/
│   ├── main/              # Electron main process
│   │   ├── main.ts        # Entry point, window creation
│   │   ├── menu.ts        # Application menu
│   │   └── preload.ts     # IPC preload
│   ├── renderer/          # React frontend
│   │   ├── App.tsx        # Main routing
│   │   ├── components/
│   │   │   ├── commonComponents/  # Reusable UI
│   │   │   ├── layout/            # MainLayout
│   │   │   └── viewComponents/    # Feature components
│   │   ├── views/         # Pages
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Signin.tsx / Signup.tsx
│   │   │   ├── products/  # Product CRUD
│   │   │   ├── services/  # Service CRUD
│   │   │   ├── inventory/ # Inventory wrapper
│   │   │   ├── sale/      # POS
│   │   │   ├── orders/    # Order management
│   │   │   ├── customer/  # Customer CRUD
│   │   │   ├── employee/  # Employee CRUD
│   │   │   ├── appointments/  # Calendar
│   │   │   └── setting/   # Settings
│   │   └── utils/
│   │       ├── axios.ts   # API client
│   │       ├── hooks/     # Custom hooks
│   │       └── methods.ts # Utilities
│   └── store/             # Redux store
│       ├── slices/        # appSlice, cartSlice, signupSlice
│       └── index.js       # Store config
├── release/               # Build output
└── vite.config.mts        # Web build config
```

### Key Features

**Authentication & Onboarding:**
- Multi-step signup: Admin → Verify code → Business → Store
- Sign-in with email/password
- Multi-store selection
- JWT token stored in localStorage

**Dashboard Analytics:**
- Real-time metrics (Sales, Customers, Volume)
- Time filters (Today, Week, Month)
- Percentage change indicators
- Top transactions, customers, products
- Interactive charts

**POS/Sales System:**
- Product catalog with search/filter
- Shopping cart with real-time updates
- Customer selection
- Multiple payment methods
- Tax calculations
- Order completion with receipt

**Inventory Management:**
- Products (ID, name, category, brand, prices, stock)
- Services (duration-based pricing)
- Stock tracking with alerts

**Order Management:**
- Order history with details
- Edit orders
- Payment status tracking

**Customer Management (CRM):**
- Customer profiles
- Purchase history tracking
- Customer analytics

**Employee Management:**
- Employee profiles
- Role-based access (Admin, Manager, Staff)

**Appointment/Calendar:**
- Day/week/month views
- Create/view/edit appointments
- Employee scheduling
- Service booking with time slots
- Status workflow (booked → completed)

**Settings:**
- Categories (product/service types)
- Brands management
- Taxes configuration
- Payment methods
- Multi-store management
- Activity logs

### API Configuration

```typescript
// src/renderer/utils/axios.ts
const api = axios.create({
  baseURL: 'https://api.stokse.com/api',
});

// Adds headers:
// - device-type: 'desktop'
// - Authorization: 'Bearer <token>'
```

### Standalone Setup

```bash
# Clone
git clone https://github.com/malikhamza-v/stokse-client.git
cd stokse-client

# Install
npm install

# Development
npm start          # Electron app
npm run start:web  # Web version

# Build
npm run build      # Production build
npm run dist       # Create distributables
```

### Build Targets

- **Windows:** NSIS installer
- **Linux:** DEB package
- **Web:** Static build via Vite

### Chrome Sandbox Fix (Linux)

```bash
sudo chown root node_modules/electron/dist/chrome-sandbox
sudo chmod 4755 node_modules/electron/dist/chrome-sandbox
```

### Windows Build via Docker

```bash
docker run --rm -ti \
  --env-file <(env | grep -iE 'DEBUG|NODE_|ELECTRON_|YARN_|NPM_|CI|CIRCLE|TRAVIS_TAG|TRAVIS|TRAVIS_REPO_|TRAVIS_BUILD_|TRAVIS_BRANCH|TRAVIS_PULL_REQUEST_|APPVEYOR_|CSC_|GH_|GITHUB_|BT_|AWS_|STRIP|BUILD_') \
  --env ELECTRON_CACHE="/root/.cache/electron" \
  --env ELECTRON_BUILDER_CACHE="/root/.cache/electron-builder" \
  -v ${PWD}:/project \
  -v ${PWD##*/}-node-modules:/project/node_modules \
  -v ~/.cache/electron:/root/.cache/electron \
  -v ~/.cache/electron-builder:/root/.cache/electron-builder \
  electronuserland/builder:wine

npm run dist
```

---

## Development Workflow

### Local Development (Without Docker)

1. **Start PostgreSQL database**
2. **Configure server `.env`** with local DB credentials
3. **Run server:** `cd stokse-server && python manage.py runserver`
4. **Configure client API URL** in `src/renderer/utils/axios.ts`:
   ```typescript
   baseURL: 'http://localhost:8000/api',
   ```
5. **Run client:** `cd stokse-client && npm start`

### Production Deployment

1. **Configure all environment variables**
2. **Build containers:** `docker-compose build`
3. **Start services:** `docker-compose up -d`
4. **Configure reverse proxy** (nginx) for SSL termination
5. **Set up static file serving**

---

## Links

| Repository | URL |
|------------|-----|
| Container | https://github.com/malikhamza-v/stokse-container |
| Server | https://github.com/malikhamza-v/stokse-server |
| Client | https://github.com/malikhamza-v/stokse-client |
| Releases | https://github.com/malikhamza-v/stokse-builds |

---

## License

Copyright (c) Stokse. All rights reserved.
