# FaucetHaven

FaucetHaven is a cryptocurrency-based application using **Laravel 11** for the backend, **Next.js** for the frontend, and **Clerk** for authentication. This project integrates a **faucet system**, **staking**, and **mining power upgrades**.

## 🚀 Tech Stack

### Backend (Laravel 11)
- PHP 8+
- Laravel 11
- Sanctum (API authentication)
- MySQL (Database)
- CloudPanel (Deployment)

### Frontend (Next.js)
- Next.js (React + TypeScript)
- Clerk (Authentication)
- Axios (API requests)
- Tailwind CSS (Styling)
- Vercel (Deployment)

---

## 📌 Setup Guide

### 1️⃣ Backend Setup (Laravel 11)

#### Install Laravel 11
```bash
composer create-project laravel/laravel faucethaven-api
cd faucethaven-api
```

#### Configure Database
Edit `.env`:
```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=faucethaven
DB_USERNAME=root
DB_PASSWORD=yourpassword
```
Then migrate:
```bash
php artisan migrate
```

#### Install Sanctum for API Authentication
```bash
composer require laravel/sanctum
php artisan migrate
```

Enable Sanctum in **`bootstrap/app.php`**:
```php
$app->withMiddleware([
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
]);
```

#### Middleware for Clerk Authentication
Create **ClerkAuthMiddleware.php**:
```php
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class ClerkAuthMiddleware {
    public function handle(Request $request, Closure $next) {
        $token = $request->bearerToken();
        if (!$token) return response()->json(['error' => 'Unauthorized'], 401);

        $response = Http::withHeaders(['Authorization' => "Bearer $token"])
            ->get('https://api.clerk.dev/v1/me');

        if ($response->failed()) return response()->json(['error' => 'Invalid token'], 401);

        return $next($request);
    }
}
```
Register Middleware in **`bootstrap/app.php`**:
```php
$app->withMiddleware([
    \App\Http\Middleware\ClerkAuthMiddleware::class,
]);
```

Apply Middleware to API Routes (`routes/api.php`):
```php
Route::middleware(['clerk'])->group(function () {
    Route::get('/user-data', [UserController::class, 'getUserData']);
});
```

#### CORS Configuration
Edit **`bootstrap/app.php`**:
```php
$app->withMiddleware([
    \Illuminate\Http\Middleware\HandleCors::class,
]);

$app->withCors([
    'paths' => ['api/*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['https://faucethaven.io'],
    'allowed_headers' => ['*'],
    'supports_credentials' => true,
]);
```

Clear cache:
```bash
php artisan config:clear
php artisan cache:clear
```

---

### 2️⃣ Frontend Setup (Next.js)

#### Install Next.js & Dependencies
```bash
npx create-next-app@latest faucethaven-frontend -ts --tailwind
cd faucethaven-frontend
npm install @clerk/nextjs axios react-query
```

#### Configure Clerk
Create `.env.local`:
```ini
NEXT_PUBLIC_CLERK_FRONTEND_API=your_clerk_frontend_api
```

Wrap `_app.tsx` with Clerk Provider:
```tsx
import { ClerkProvider } from '@clerk/nextjs';

function MyApp({ Component, pageProps }) {
  return (
    <ClerkProvider>
      <Component {...pageProps} />
    </ClerkProvider>
  );
}

export default MyApp;
```

#### Fetch API Data
Modify `pages/dashboard.tsx`:
```tsx
import { useUser, useAuth } from '@clerk/nextjs';
import { useEffect, useState } from 'react';

export default function Dashboard() {
  const { user } = useUser();
  const { getToken } = useAuth();
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const token = await getToken();
      const res = await fetch("https://api.faucethaven.io/user-data", {
        headers: { Authorization: `Bearer ${token}` },
      });
      setData(await res.json());
    };

    if (user) fetchData();
  }, [user]);

  return (
    <div>
      <h1>Welcome, {user?.fullName}</h1>
      <p>Balance: {data?.balance} TOKEN</p>
      <p>Power: {data?.power}</p>
    </div>
  );
}
```

---

## 🔄 Deployment

### Backend (Laravel 11) on CloudPanel
1. Upload Laravel project to CloudPanel.
2. Set environment variables in `.env`.
3. Run migrations:
   ```bash
   php artisan migrate --seed
   ```
4. Start Laravel:
   ```bash
   php artisan serve --host=0.0.0.0 --port=8000
   ```

### Frontend (Next.js) on Vercel
1. Push Next.js to GitHub.
2. Deploy on Vercel.
3. Set environment variables in Vercel dashboard.

---

## 📌 Features
✅ **User Authentication** (Clerk)
✅ **Real-time Balance Updates**
✅ **Faucet & Mining Power System**
✅ **Secure API Communication**
✅ **Scalable Infrastructure**

---

## ⚡ What's Next?
- **Staking & Rewards System**
- **WebSockets for Live Updates**
- **Multi-Currency Support**

---

### 💡 Need Help?
Feel free to contribute or reach out for support! 🚀

