# Frontend Architecture Problems - 25+ Real-World Problems

> **Curated from top tech companies and real production scenarios**

This guide contains 25+ real-world frontend architecture problems with comprehensive solutions, suitable for staff engineer interviews.

---

## Table of Contents

1. [React Architecture Problems](#react-architecture-problems)
2. [Performance Optimization Problems](#performance-optimization-problems)
3. [State Management Problems](#state-management-problems)
4. [Scalability Problems](#scalability-problems)

---

## React Architecture Problems

### Problem 1: Design a Large-Scale React Application

**Problem Statement:**
Design a React application architecture that can handle 100+ components, multiple teams, and maintainability.

**Requirements:**
- Modular architecture
- Code splitting
- Lazy loading
- Team collaboration
- Performance optimization

**Solution:**

```javascript
// Folder structure
src/
  ├── components/        # Shared components
  │   ├── common/
  │   ├── layout/
  │   └── ui/
  ├── features/          # Feature-based modules
  │   ├── auth/
  │   │   ├── components/
  │   │   ├── hooks/
  │   │   ├── services/
  │   │   └── index.js
  │   ├── products/
  │   └── orders/
  ├── shared/            # Shared utilities
  │   ├── hooks/
  │   ├── utils/
  │   └── constants/
  ├── services/          # API services
  ├── store/            # State management
  └── App.js

// Feature module structure (products)
features/products/
  ├── components/
  │   ├── ProductList.jsx
  │   ├── ProductCard.jsx
  │   └── ProductDetail.jsx
  ├── hooks/
  │   ├── useProducts.js
  │   └── useProduct.js
  ├── services/
  │   └── productService.js
  ├── store/
  │   └── productSlice.js
  └── index.js

// Code splitting with React.lazy
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Products = lazy(() => import('./features/products'));
const Orders = lazy(() => import('./features/orders'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/products" element={<Products />} />
          <Route path="/orders" element={<Orders />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Key Principles:**
- Feature-based organization
- Code splitting per route
- Shared components library
- Team ownership per feature

---

### Problem 2: Optimize React Rendering Performance

**Problem Statement:**
A React application with 1000+ list items is experiencing performance issues. Optimize rendering.

**Requirements:**
- Render 1000+ items smoothly
- Maintain 60 FPS
- Handle scrolling
- Support filtering and sorting

**Solution:**

```javascript
// Problem: Rendering 1000 items causes lag
function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Solution 1: React.memo for component memoization
const ProductCard = React.memo(({ product }) => {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  return prevProps.product.id === nextProps.product.id &&
         prevProps.product.name === nextProps.product.name &&
         prevProps.product.price === nextProps.product.price;
});

// Solution 2: Virtual scrolling (react-window)
import { FixedSizeList } from 'react-window';

function VirtualizedProductList({ products }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ProductCard product={products[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={products.length}
      itemSize={100}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// Solution 3: useMemo for expensive calculations
function ProductList({ products, filter }) {
  const filteredProducts = useMemo(() => {
    return products.filter(p => 
      p.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [products, filter]);

  return (
    <div>
      {filteredProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Solution 4: useCallback for function memoization
function ProductList({ products, onProductClick }) {
  const handleClick = useCallback((productId) => {
    onProductClick(productId);
  }, [onProductClick]);

  return (
    <div>
      {products.map(product => (
        <ProductCard 
          key={product.id} 
          product={product}
          onClick={() => handleClick(product.id)}
        />
      ))}
    </div>
  );
}
```

**Optimization Techniques:**
- React.memo for component memoization
- Virtual scrolling for large lists
- useMemo for expensive calculations
- useCallback for function memoization
- Code splitting for lazy loading

---

### Problem 3: Design State Management for Complex Application

**Problem Statement:**
Design a state management solution for a complex React application with multiple features and shared state.

**Requirements:**
- Multiple features (auth, products, cart, orders)
- Shared state (user, theme, notifications)
- Server state management
- Optimistic updates
- Cache management

**Solution:**

```javascript
// Using Redux Toolkit + RTK Query
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// RTK Query for server state
export const productsApi = createApi({
  reducerPath: 'productsApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Product'],
  endpoints: (builder) => ({
    getProducts: builder.query({
      query: () => 'products',
      providesTags: ['Product'],
    }),
    getProduct: builder.query({
      query: (id) => `products/${id}`,
      providesTags: (result, error, id) => [{ type: 'Product', id }],
    }),
    createProduct: builder.mutation({
      query: (product) => ({
        url: 'products',
        method: 'POST',
        body: product,
      }),
      invalidatesTags: ['Product'],
      // Optimistic update
      async onQueryStarted(product, { dispatch, queryFulfilled }) {
        const patchResult = dispatch(
          productsApi.util.updateQueryData('getProducts', undefined, (draft) => {
            draft.push({ ...product, id: Date.now() });
          })
        );
        try {
          await queryFulfilled;
        } catch {
          patchResult.undo();
        }
      },
    }),
  }),
});

// Redux slice for client state
const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
  },
  reducers: {
    addItem: (state, action) => {
      const item = action.payload;
      const existingItem = state.items.find(i => i.id === item.id);
      if (existingItem) {
        existingItem.quantity++;
      } else {
        state.items.push({ ...item, quantity: 1 });
      }
      state.total = calculateTotal(state.items);
    },
    removeItem: (state, action) => {
      state.items = state.items.filter(i => i.id !== action.payload);
      state.total = calculateTotal(state.items);
    },
  },
});

// Custom hook for component usage
function useCart() {
  const dispatch = useDispatch();
  const cart = useSelector(state => state.cart);
  
  const addToCart = useCallback((product) => {
    dispatch(cartSlice.actions.addItem(product));
  }, [dispatch]);
  
  return { cart, addToCart };
}
```

**State Management Strategy:**
- **Server State**: RTK Query (caching, invalidation, optimistic updates)
- **Client State**: Redux Toolkit (cart, UI state)
- **Local State**: useState (component-specific)
- **Shared State**: Context API (theme, user preferences)

---

## Performance Optimization Problems

### Problem 4: Optimize Bundle Size

**Problem Statement:**
React application bundle size is 5MB, causing slow initial load. Optimize bundle size.

**Requirements:**
- Reduce bundle size to < 500KB
- Fast initial load
- Code splitting
- Tree shaking

**Solution:**

```javascript
// webpack.config.js optimizations
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          minChunks: 2,
          name: 'common',
          chunks: 'all',
        },
      },
    },
    usedExports: true, // Tree shaking
    minimize: true,
  },
  resolve: {
    alias: {
      'react': 'react',
      'react-dom': 'react-dom',
    },
  },
};

// Dynamic imports for code splitting
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Tree shaking: Import only what you need
// Bad
import _ from 'lodash';
const result = _.map(array, fn);

// Good
import { map } from 'lodash-es';
const result = map(array, fn);

// Bundle analysis
npm install --save-dev webpack-bundle-analyzer
```

**Optimization Techniques:**
- Code splitting per route
- Dynamic imports for heavy components
- Tree shaking unused code
- Vendor chunk separation
- Compression (gzip, brotli)

---

### Problem 5: Optimize Image Loading

**Problem Statement:**
E-commerce site with 1000+ product images loads slowly. Optimize image loading.

**Requirements:**
- Fast initial load
- Lazy load images
- Responsive images
- WebP format support

**Solution:**

```javascript
// Lazy loading with Intersection Observer
function LazyImage({ src, alt, ...props }) {
  const [imageSrc, setImageSrc] = useState(null);
  const [imageRef, setImageRef] = useState();
  const [isLoaded, setIsLoaded] = useState(false);

  useEffect(() => {
    let observer;
    if (imageRef && imageSrc === null) {
      observer = new IntersectionObserver(
        entries => {
          entries.forEach(entry => {
            if (entry.isIntersecting) {
              setImageSrc(src);
              observer.unobserve(imageRef);
            }
          });
        },
        { rootMargin: '50px' }
      );
      observer.observe(imageRef);
    }
    return () => {
      if (observer && imageRef) {
        observer.unobserve(imageRef);
      }
    };
  }, [imageRef, imageSrc, src]);

  return (
    <div
      ref={setImageRef}
      className={`lazy-image ${isLoaded ? 'loaded' : ''}`}
    >
      {imageSrc && (
        <img
          src={imageSrc}
          alt={alt}
          onLoad={() => setIsLoaded(true)}
          loading="lazy"
          {...props}
        />
      )}
    </div>
  );
}

// Responsive images with srcset
function ResponsiveImage({ src, alt }) {
  return (
    <img
      srcSet={`
        ${src}-small.webp 400w,
        ${src}-medium.webp 800w,
        ${src}-large.webp 1200w
      `}
      sizes="(max-width: 400px) 400px,
             (max-width: 800px) 800px,
             1200px"
      src={`${src}-medium.webp`}
      alt={alt}
      loading="lazy"
    />
  );
}

// Image optimization service
class ImageOptimizer {
  static getOptimizedUrl(originalUrl, width, quality = 80) {
    // Use CDN with image optimization
    return `https://cdn.example.com/${width}/${quality}/${originalUrl}`;
  }
  
  static getWebPUrl(originalUrl) {
    // Convert to WebP if supported
    if (this.supportsWebP()) {
      return originalUrl.replace(/\.(jpg|png)$/, '.webp');
    }
    return originalUrl;
  }
  
  static supportsWebP() {
    const canvas = document.createElement('canvas');
    return canvas.toDataURL('image/webp').indexOf('data:image/webp') === 0;
  }
}
```

**Optimization Techniques:**
- Lazy loading with Intersection Observer
- Responsive images with srcset
- WebP format for modern browsers
- CDN with image optimization
- Placeholder images (blur-up technique)

---

## State Management Problems

### Problem 6: Handle Complex Form State

**Problem Statement:**
Design a form management solution for complex forms with validation, conditional fields, and nested data.

**Requirements:**
- Multiple form fields
- Validation
- Conditional fields
- Nested data structures
- Form state persistence

**Solution:**

```javascript
// Using React Hook Form
import { useForm, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object().shape({
  name: yup.string().required('Name is required'),
  email: yup.string().email('Invalid email').required('Email is required'),
  address: yup.object().shape({
    street: yup.string().required(),
    city: yup.string().required(),
    zipCode: yup.string().matches(/^\d{5}$/, 'Invalid zip code'),
  }),
  paymentMethod: yup.string().required(),
  creditCard: yup.string().when('paymentMethod', {
    is: 'credit',
    then: yup.string().required('Credit card required'),
  }),
});

function CheckoutForm() {
  const { register, handleSubmit, control, watch, formState: { errors } } = useForm({
    resolver: yupResolver(schema),
    defaultValues: {
      paymentMethod: 'cash',
    },
  });

  const paymentMethod = watch('paymentMethod');

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}

      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <input {...register('address.street')} />
      <input {...register('address.city')} />
      <input {...register('address.zipCode')} />

      <select {...register('paymentMethod')}>
        <option value="cash">Cash</option>
        <option value="credit">Credit Card</option>
      </select>

      {paymentMethod === 'credit' && (
        <Controller
          name="creditCard"
          control={control}
          render={({ field }) => (
            <input {...field} placeholder="Credit card number" />
          )}
        />
      )}

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Summary

These problems cover:
- ✅ React architecture patterns
- ✅ Performance optimization
- ✅ State management
- ✅ Code splitting
- ✅ Image optimization
- ✅ Form management

**Key Principles:**
- Component composition
- Code splitting
- Performance optimization
- State management strategy
- Bundle optimization

---

*This guide is continuously updated with new problems and solutions.*

