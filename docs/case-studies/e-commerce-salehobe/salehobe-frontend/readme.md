# Salehobe-frontend
<a name="readme-top"></a>

<div>

# 📗 Table of Contents

- [📖 About the Project](#about-project)
  - [🛠 Built With](#built-with)
    - [Key Features](#key-features)
     - [🚀 Live Demo](#live-demo)
    - [Performance Targets](#performance-targets)
    - [Architecture Overview](#architecture-overview)
- [💻 Getting Started](#getting-started)
  - [Setup](#setup)
  - [Branching Strategy](#branching-strategy)
  - [Development Workflow](#development-workflow)
- [🏗️ Project Structure](#project-structure)
- [⚡ Performance Optimizations](#performance-optimizations)
- [🔌 Integration with Backend](#integration-backend)
- [☁️ Deployment & Hosting](#deployment-hosting)
- [🧪 Testing & Quality](#testing-quality)
- [🔭 Future Features](#future-features)
- [👥 Authors](#authors)
- [📝 License](#license)
- [🙏 Acknowledgements](#acknowledgements)

## 📖 About the Project <a name="about-project"></a>

SALEHOBE-FRONTEND is a high-performance e-commerce frontend built with Next.js 15 (App Router), designed to achieve sub-second homepage rendering times. The project follows Technical Agile principles with an emphasis on performance, modular architecture, and seamless integration with the Salehobe Rails API.

## 🛠 Built With <a name="built-with"></a>

- **Next.js 15** — App Router with React 19
- **TypeScript** — Full type safety
- **Tailwind CSS** — Utility-first styling
- **Cloudflare** — Domain registration & hosting
- **React Query (@tanstack/react-query)** — Server state management
- **OpenNext.js Cloudflare Adapter** — Edge-compatible deployment

## Key Features <a name="key-features"></a>

- **Sub-second TTFB** — Homepage target: < 1 second render time
- **App Router Architecture** — File-based routing with layouts
- **Performance-First** — Turbo mode, optimized builds, edge runtime
- **Modular Components** — Organized by domain (admin, products, shared)
- **Type Safety** — Full TypeScript integration
- **Edge Deployment** — Cloudflare Workers for global low latency
  <!-- LIVE DEMO -->
## 🚀 Live Demo <a name="live-demo"></a>

-  [custome doamin](https://salehobe.com/) or [cloudflare](https://my-app.utechdynamics.workers.dev/)
<!--  [Video description] -->


## Performance Targets <a name="performance-targets"></a>

- **Homepage Load Time**: < 1 second
- **First Contentful Paint (FCP)**: < 800ms
- **Lighthouse Performance Score**: > 90
- **Core Web Vitals**: All "Good" thresholds

## Architecture Overview <a name="architecture-overview"></a>

- **Routing**: Next.js App Router with nested layouts
- **State Management**: React Context for UI, React Query for server state
- **Styling**: Tailwind CSS with PostCSS optimization
- **Build**: OpenNext.js Cloudflare adapter for edge compatibility
- **Deployment**: Cloudflare Pages with Workers integration

## 💻 Getting Started <a name="getting-started"></a>

### Setup <a name="setup"></a>

```bash
# Clone the repository
git clone https://github.com/HossainAraf/saleshoc-frontend.git

# Install dependencies
npm install

# Initialize Tailwind CSS
npm run tailwind:init

# Start development server with Turbo mode
npm run dev
```

### Environment Configuration

Create `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
NEXT_PUBLIC_SITE_URL=http://localhost:3001
```

### Branching Strategy <a name="branching-strategy"></a>

- `main` — Production-ready code
- `dev` — Development branch
- `feat/*` — Feature branches
- `perf/*` — Performance optimizations
- `fix/*` — Bug fixes

### Development Workflow <a name="development-workflow"></a>

```bash
# Development (with Turbo mode)
npm run dev

# Build for production
npm run build

# Preview Cloudflare deployment
npm run preview

# Deploy to Cloudflare
npm run deploy

# Generate Cloudflare types
npm run cf-typegen

# Linting
npm run lint
npm run lint:css
```

## 🏗️ Project Structure <a name="project-structure"></a>

```
app/
├── (auth)/                    # Authentication routes
├── admin/                     # Admin dashboard
│   ├── add-product/
│   │   └── page.tsx
│   ├── all-products/
│   └── orders/
├── cart/                      # Shopping cart
├── categories/                # Category pages
├── checkout/                  # Checkout flow
├── orders/                    # Order history
├── globals.css               # Global styles
├── layout.tsx                # Root layout
└── page.tsx                  # Homepage

components/
├── admin/                    # Admin-specific components
├── order/                    # Order management
├── products/                 # Product display
├── shared/                   # Reusable components
│   ├── AnimatedText.jsx
│   ├── CartContext.tsx
│   ├── Footer.jsx
│   ├── Nav.tsx
│   ├── ProductCard.tsx
│   └── ProductForm.tsx
└── providers.tsx            # React providers
```

## ⚡ Performance Optimizations <a name="performance-optimizations"></a>

### Why Next.js for < 1s Render Time?

1. **Server-Side Rendering (SSR)** — Critical content rendered on server
2. **Static Generation** — Pre-rendered pages at build time
3. **Incremental Static Regeneration** — Fresh content without rebuilds
4. **Turbo Mode** — Fast refresh and optimized bundling
5. **Image Optimization** — Automatic WebP conversion and lazy loading
6. **Code Splitting** — Automatic chunk splitting by route

### Cloudflare Edge Benefits

- **Global CDN** — 200+ edge locations worldwide
- **Edge Computing** — Serverless functions at edge
- **Zero Cold Starts** — Instant response times
- **Automatic Optimization** — Image, CSS, JS optimization

## 🔌 Integration with Backend <a name="integration-backend"></a>

### API Communication Pattern

```typescript
// Example: React Query hook for products
import { useQuery } from '@tanstack/react-query';

const fetchProducts = async () => {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/products`);
  return res.json();
};

export const useProducts = () => {
  return useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};
```

### API Endpoints Integration

| Frontend Route       | Backend Endpoint                       | Method |
|----------------------|----------------------------------------|--------|
| `/admin/add-product` | `/api/v1/products`                     | POST   |
| `/products`          | `/api/v1/products`                     | GET    |
| `/categories/[slug]` | `/api/v1/products/filter_by_category`  | GET    |
| `/cart/checkout`     | `/api/v1/orders`                       | POST   |
| `/orders`            | `/api/v1/orders`                       | GET    |

## ☁️ Deployment & Hosting <a name="deployment-hosting"></a>

### Why Cloudflare?

1. **Performance** — Edge network for global low latency
2. **Cost-Effective** — Generous free tier
3. **Developer Experience** — Seamless Next.js integration
4. **Security** — Built-in DDoS protection and WAF
5. **Domain Management** — Unified DNS and hosting

### Deployment Pipeline

```bash
# Local build and preview
npm run build
npm run preview

# Production deployment
npm run deploy

# Manual deployment via wrangler
npx wrangler pages deploy .vercel/output/static
```

### Cloudflare Configuration

```toml
# wrangler.toml
name = "salehobe-frontend"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".vercel/output/static"
```

## 🧪 Testing & Quality <a name="testing-quality"></a>

### Quality Assurance Tools

- **ESLint** — Code quality and consistency
- **Stylelint** — CSS/SCSS linting
- **TypeScript** — Compile-time type checking
- **Next.js Built-in Checks** — Link validation, image optimization

### Testing Strategy

```bash
# Run all linting
npm run lint          # JavaScript/TypeScript
npm run lint:css      # CSS/SCSS

# Type checking
npx tsc --noEmit

# Build verification
npm run build
```

## 🔭 Future Features <a name="future-features"></a>

- **PWA Support** — Offline capabilities and installability
- **Real-time Updates** — WebSocket for stock/price updates
- **Advanced Search** — Algolia or Meilisearch integration
- **A/B Testing** — Feature flags for experimentation
- **Analytics Dashboard** — Real-time performance monitoring
- **Mobile App** — React Native with shared business logic
- **Internationalization** — Multi-language support (i18n)
- **Dark Mode** — Theme switching with system detection

## 👥 Authors <a name="authors"></a>

**Md Arafat Hossain**

- GitHub: <a href="https://github.com/HossainAraf">HossainAraf</a>
- LinkedIn: <a href="https://www.linkedin.com/in/hossain-arafat-engineer/">Md. Arafat Hossain</a>

**Anum Munir**
- GitHub: <a href="https://github.com/AnumMunir1907">AnumMunir1907</a>

## 📄 License <a name="license"></a>

MIT License — See [LICENSE](LICENSE)

## 🙏 Acknowledgements <a name="acknowledgements"></a>

- **Family Support** — For continuous encouragement
- **Microverse** — Structure, standards, and discipline
- **Next.js Team** — Excellent documentation and tooling
- **Cloudflare** — Developer-friendly edge platform
- **Open Source Community** — For invaluable tools and libraries

---

<p align="right">(<a href="#readme-top">back to top</a>)</p>
</div>
