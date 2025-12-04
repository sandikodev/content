---
title: "Svelte 5 Runes: The Revolutionary Evolution from Svelte 4"
description: "Complete guide to Svelte 5's Runes system - exploring the syntax evolution, performance improvements, and why Runes make Svelte even more intuitive and efficient."
date: 2025-12-04T10:00:00+07:00
author: "Sandikodev"
category: "Frontend"
tags: ["svelte", "svelte5", "runes", "reactivity", "performance", "migration"]
image: "/images/blog/svelte5-runes.webp"
draft: false
---

## What Are Runes?

Runes are **reactive primitives** that provide explicit, fine-grained reactivity. They replace Svelte 4's implicit reactivity with a more predictable and powerful system.

```javascript
// Svelte 5 Runes - Explicit and powerful
import { $state, $derived, $effect } from 'svelte';

let count = $state(0);
let doubled = $derived(() => count * 2);

$effect(() => {
  console.log(`Count is now ${count}`);
});
```

## Svelte 4 vs Svelte 5: Side-by-Side Comparison

### 1. State Management

#### Svelte 4 - Implicit Reactivity
```svelte
<!-- Svelte 4 -->
<script>
  let count = 0;
  let name = 'World';
  
  // Reactive declarations
  $: doubled = count * 2;
  $: greeting = `Hello, ${name}!`;
  
  // Reactive statements
  $: if (count > 10) {
    console.log('Count is getting high!');
  }
</script>

<p>{greeting}</p>
<p>Count: {count}, Doubled: {doubled}</p>
<button on:click={() => count++}>Increment</button>
```

#### Svelte 5 - Explicit Runes
```svelte
<!-- Svelte 5 -->
<script>
  import { $state, $derived, $effect } from 'svelte';
  
  let count = $state(0);
  let name = $state('World');
  
  // Derived state
  let doubled = $derived(() => count * 2);
  let greeting = $derived(() => `Hello, ${name}!`);
  
  // Effects
  $effect(() => {
    if (count > 10) {
      console.log('Count is getting high!');
    }
  });
</script>

<p>{greeting}</p>
<p>Count: {count}, Doubled: {doubled}</p>
<button onclick={() => count++}>Increment</button>
```

**Key Improvements:**
- ✅ **Explicit state** - `$state()` makes reactive variables clear
- ✅ **Better performance** - Fine-grained reactivity tracking
- ✅ **Type safety** - Better TypeScript inference
- ✅ **Predictable behavior** - No magic, clear dependencies

### 2. Complex State Objects

#### Svelte 4 - Assignment-based Reactivity
```svelte
<!-- Svelte 4 -->
<script>
  let user = {
    name: 'John',
    age: 30,
    preferences: {
      theme: 'dark',
      language: 'en'
    }
  };
  
  // Need to reassign for reactivity
  function updateTheme(newTheme) {
    user = {
      ...user,
      preferences: {
        ...user.preferences,
        theme: newTheme
      }
    };
  }
  
  function incrementAge() {
    user = { ...user, age: user.age + 1 };
  }
</script>

<p>{user.name} is {user.age} years old</p>
<p>Theme: {user.preferences.theme}</p>
<button on:click={() => updateTheme('light')}>Light Theme</button>
<button on:click={incrementAge}>Birthday</button>
```

#### Svelte 5 - Deep Reactivity with Runes
```svelte
<!-- Svelte 5 -->
<script>
  import { $state } from 'svelte';
  
  let user = $state({
    name: 'John',
    age: 30,
    preferences: {
      theme: 'dark',
      language: 'en'
    }
  });
  
  // Direct mutation works!
  function updateTheme(newTheme) {
    user.preferences.theme = newTheme;
  }
  
  function incrementAge() {
    user.age++;
  }
</script>

<p>{user.name} is {user.age} years old</p>
<p>Theme: {user.preferences.theme}</p>
<button onclick={() => updateTheme('light')}>Light Theme</button>
<button onclick={incrementAge}>Birthday</button>
```

**Revolutionary Changes:**
- 🚀 **Deep reactivity** - Nested object mutations are tracked
- 🚀 **Natural mutations** - No more spread operator gymnastics
- 🚀 **Better performance** - Only affected parts re-render
- 🚀 **Simpler code** - Write what you mean

### 3. Derived State and Computations

#### Svelte 4 - Reactive Declarations
```svelte
<!-- Svelte 4 -->
<script>
  let items = [
    { id: 1, name: 'Apple', price: 1.2, category: 'fruit' },
    { id: 2, name: 'Bread', price: 2.5, category: 'bakery' },
    { id: 3, name: 'Milk', price: 3.0, category: 'dairy' }
  ];
  
  let filter = '';
  let sortBy = 'name';
  
  // Complex reactive declarations
  $: filteredItems = items.filter(item => 
    item.name.toLowerCase().includes(filter.toLowerCase())
  );
  
  $: sortedItems = filteredItems.sort((a, b) => {
    if (sortBy === 'price') return a.price - b.price;
    return a.name.localeCompare(b.name);
  });
  
  $: totalPrice = sortedItems.reduce((sum, item) => sum + item.price, 0);
  $: averagePrice = sortedItems.length > 0 ? totalPrice / sortedItems.length : 0;
</script>
```

#### Svelte 5 - Derived Runes
```svelte
<!-- Svelte 5 -->
<script>
  import { $state, $derived } from 'svelte';
  
  let items = $state([
    { id: 1, name: 'Apple', price: 1.2, category: 'fruit' },
    { id: 2, name: 'Bread', price: 2.5, category: 'bakery' },
    { id: 3, name: 'Milk', price: 3.0, category: 'dairy' }
  ]);
  
  let filter = $state('');
  let sortBy = $state('name');
  
  // Derived state with clear dependencies
  let filteredItems = $derived(() => 
    items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    )
  );
  
  let sortedItems = $derived(() => 
    filteredItems.sort((a, b) => {
      if (sortBy === 'price') return a.price - b.price;
      return a.name.localeCompare(b.name);
    })
  );
  
  let totalPrice = $derived(() => 
    sortedItems.reduce((sum, item) => sum + item.price, 0)
  );
  
  let averagePrice = $derived(() => 
    sortedItems.length > 0 ? totalPrice / sortedItems.length : 0
  );
</script>
```

**Derived State Benefits:**
- 🎯 **Explicit dependencies** - Clear what each computation depends on
- 🎯 **Better caching** - More efficient memoization
- 🎯 **Easier debugging** - Clear computation chains
- 🎯 **Type inference** - Better TypeScript support

### 4. Side Effects and Lifecycle

#### Svelte 4 - Reactive Statements and Lifecycle
```svelte
<!-- Svelte 4 -->
<script>
  import { onMount, onDestroy } from 'svelte';
  
  let count = 0;
  let data = null;
  let interval;
  
  // Reactive side effects
  $: if (count > 0) {
    console.log(`Count changed to ${count}`);
  }
  
  $: {
    // Complex reactive block
    if (count % 5 === 0 && count > 0) {
      fetchData();
    }
  }
  
  onMount(() => {
    interval = setInterval(() => {
      count++;
    }, 1000);
  });
  
  onDestroy(() => {
    if (interval) {
      clearInterval(interval);
    }
  });
  
  async function fetchData() {
    const response = await fetch(`/api/data/${count}`);
    data = await response.json();
  }
</script>
```

#### Svelte 5 - Effect Runes
```svelte
<!-- Svelte 5 -->
<script>
  import { $state, $effect } from 'svelte';
  
  let count = $state(0);
  let data = $state(null);
  
  // Effect with automatic cleanup
  $effect(() => {
    const interval = setInterval(() => {
      count++;
    }, 1000);
    
    // Cleanup function
    return () => clearInterval(interval);
  });
  
  // Reactive side effect
  $effect(() => {
    if (count > 0) {
      console.log(`Count changed to ${count}`);
    }
  });
  
  // Conditional effect
  $effect(() => {
    if (count % 5 === 0 && count > 0) {
      fetchData();
    }
  });
  
  async function fetchData() {
    const response = await fetch(`/api/data/${count}`);
    data = await response.json();
  }
</script>
```

**Effect Improvements:**
- 🔄 **Automatic cleanup** - Return cleanup function from effects
- 🔄 **Fine-grained tracking** - Only re-run when dependencies change
- 🔄 **No lifecycle confusion** - Effects handle both mount and updates
- 🔄 **Better error handling** - Clearer error boundaries

## Advanced Runes Features

### 1. $state.frozen() for Immutable State

```javascript
import { $state } from 'svelte';

// Mutable state (default)
let mutableUser = $state({ name: 'John', age: 30 });
mutableUser.age++; // ✅ Works

// Immutable state
let immutableUser = $state.frozen({ name: 'Jane', age: 25 });
// immutableUser.age++; // ❌ Error in development

// Update immutable state
immutableUser = { ...immutableUser, age: 26 }; // ✅ Works
```

### 2. $derived.by() for Complex Computations

```javascript
import { $state, $derived } from 'svelte';

let users = $state([
  { name: 'John', posts: 5, followers: 100 },
  { name: 'Jane', posts: 12, followers: 250 }
]);

// Complex derived computation
let topInfluencer = $derived.by(() => {
  let best = users[0];
  for (const user of users) {
    const score = user.posts * 2 + user.followers;
    const bestScore = best.posts * 2 + best.followers;
    if (score > bestScore) {
      best = user;
    }
  }
  return best;
});
```

### 3. $effect.pre() for Pre-DOM Updates

```javascript
import { $state, $effect } from 'svelte';

let scrollY = $state(0);
let element;

// Effect that runs before DOM updates
$effect.pre(() => {
  if (element) {
    element.style.transform = `translateY(${scrollY * 0.5}px)`;
  }
});
```

## Performance Improvements

### Bundle Size Comparison

```bash
# Svelte 4 Todo App
Bundle size: ~12KB (gzipped)
Runtime overhead: ~3KB

# Svelte 5 Todo App (same functionality)
Bundle size: ~8KB (gzipped)  # 33% smaller!
Runtime overhead: ~2KB       # More efficient runtime
```

### Reactivity Performance

```javascript
// Svelte 4 - Coarse-grained reactivity
let items = [/* 1000 items */];
$: filteredItems = items.filter(item => item.active);
// Re-runs entire filter when ANY item changes

// Svelte 5 - Fine-grained reactivity
let items = $state([/* 1000 items */]);
let filteredItems = $derived(() => items.filter(item => item.active));
// Only re-runs when items array or relevant item.active changes
```

## Migration Guide: Svelte 4 → Svelte 5

### 1. Basic State Migration

```javascript
// Before (Svelte 4)
let count = 0;
let name = 'World';

// After (Svelte 5)
import { $state } from 'svelte';
let count = $state(0);
let name = $state('World');
```

### 2. Reactive Declarations → Derived

```javascript
// Before (Svelte 4)
$: doubled = count * 2;
$: greeting = `Hello, ${name}!`;

// After (Svelte 5)
import { $derived } from 'svelte';
let doubled = $derived(() => count * 2);
let greeting = $derived(() => `Hello, ${name}!`);
```

### 3. Reactive Statements → Effects

```javascript
// Before (Svelte 4)
$: if (count > 10) {
  console.log('High count!');
}

// After (Svelte 5)
import { $effect } from 'svelte';
$effect(() => {
  if (count > 10) {
    console.log('High count!');
  }
});
```

### 4. Event Handlers Update

```svelte
<!-- Before (Svelte 4) -->
<button on:click={() => count++}>Increment</button>

<!-- After (Svelte 5) -->
<button onclick={() => count++}>Increment</button>
```

## Real-World Example: Shopping Cart

### Svelte 4 Implementation
```svelte
<script>
  let items = [];
  let discount = 0;
  
  $: subtotal = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  $: discountAmount = subtotal * (discount / 100);
  $: total = subtotal - discountAmount;
  $: itemCount = items.reduce((sum, item) => sum + item.quantity, 0);
  
  function addItem(product) {
    const existing = items.find(item => item.id === product.id);
    if (existing) {
      items = items.map(item => 
        item.id === product.id 
          ? { ...item, quantity: item.quantity + 1 }
          : item
      );
    } else {
      items = [...items, { ...product, quantity: 1 }];
    }
  }
  
  function removeItem(id) {
    items = items.filter(item => item.id !== id);
  }
</script>
```

### Svelte 5 Implementation
```svelte
<script>
  import { $state, $derived } from 'svelte';
  
  let items = $state([]);
  let discount = $state(0);
  
  let subtotal = $derived(() => 
    items.reduce((sum, item) => sum + (item.price * item.quantity), 0)
  );
  
  let discountAmount = $derived(() => subtotal * (discount / 100));
  let total = $derived(() => subtotal - discountAmount);
  let itemCount = $derived(() => 
    items.reduce((sum, item) => sum + item.quantity, 0)
  );
  
  function addItem(product) {
    const existing = items.find(item => item.id === product.id);
    if (existing) {
      existing.quantity++; // Direct mutation works!
    } else {
      items.push({ ...product, quantity: 1 }); // Direct push works!
    }
  }
  
  function removeItem(id) {
    const index = items.findIndex(item => item.id === id);
    if (index > -1) {
      items.splice(index, 1); // Direct splice works!
    }
  }
</script>
```

**Svelte 5 Advantages:**
- 🛒 **Natural mutations** - Direct array/object manipulation
- 🛒 **Better performance** - Fine-grained reactivity
- 🛒 **Cleaner code** - No spread operator needed
- 🛒 **Easier debugging** - Clear state mutations

## TypeScript Integration

### Enhanced Type Safety

```typescript
// Svelte 5 with TypeScript
import { $state, $derived, $effect } from 'svelte';

interface User {
  id: number;
  name: string;
  email: string;
}

interface CartItem {
  product: User;
  quantity: number;
}

// Fully typed reactive state
let users = $state<User[]>([]);
let cart = $state<CartItem[]>([]);

// Derived state with perfect type inference
let totalItems = $derived(() => 
  cart.reduce((sum, item) => sum + item.quantity, 0)
);

// Type-safe effects
$effect(() => {
  // TypeScript knows cart is CartItem[]
  cart.forEach(item => {
    console.log(`${item.product.name}: ${item.quantity}`);
  });
});
```

## When to Migrate to Svelte 5

### ✅ Migrate If:
- You want better performance
- You need fine-grained reactivity
- You're starting a new project
- You want better TypeScript support
- You're tired of spread operator patterns

### ⏳ Wait If:
- You have a large Svelte 4 codebase
- Your team needs time to learn Runes
- You're using many Svelte 4-specific libraries
- You're close to a major release

## Conclusion: The Future is Runes

Svelte 5 Runes represent a **quantum leap** in frontend reactivity:

**Key Benefits:**
- 🚀 **30% smaller bundles** - More efficient compilation
- 🚀 **Fine-grained reactivity** - Better performance
- 🚀 **Natural mutations** - Write intuitive code
- 🚀 **Better TypeScript** - Enhanced type safety
- 🚀 **Explicit dependencies** - Predictable behavior

**The Evolution:**
- **Svelte 3/4**: Magical but sometimes unpredictable
- **Svelte 5**: Magical AND predictable

Runes make Svelte even more **intuitive, efficient, and powerful**. They represent the natural evolution of reactive programming - explicit where it matters, magical where it helps.

**Ready to upgrade?** Start with new components using Runes, then gradually migrate existing ones. The future of Svelte is here, and it's more beautiful than ever! ✨


*Want to see Runes in action? Check out my [RENDER project](https://github.com/workspace-framework) where I'm using Svelte 5 to build revolutionary desktop development tools!*

*This article complements my [Modern Frontend DX Wars series](/blog/modern-frontend-dx-part1-svelte-excellence) - exploring why Svelte continues to lead in developer experience.*
