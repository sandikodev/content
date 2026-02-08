---
title: "SvelteKit 5 + Runes: Framework Terbaik untuk Canvas-Centered Apps"
description: "Deep dive SvelteKit 5 dengan Runes API untuk membangun canvas app seperti Canva. Perbandingan dengan React, code examples, dan kenapa Svelte 5 game-changer untuk interactive apps."
date: 2025-11-25T10:00:00+07:00
author: "Sandikodev"
category: "Frontend"
tags:
  [
    "svelte",
    "sveltekit",
    "svelte5",
    "runes",
    "canvas",
    "fabric.js",
    "performance",
  ]
image: "/images/blog/sveltekit-canvas.webp"
draft: false
---

## TL;DR

**SvelteKit 5 BEST FOR**: Canvas apps, drawing tools, image editors, interactive dashboards  
**Bundle Size**: 3x lebih kecil dari React  
**DX**: Terbaik di kelasnya  
**Runes**: Reactivity system baru yang revolutionary

## Kenapa SvelteKit 5?

### 🎯 Perfect untuk Canvas Apps

1. **No Virtual DOM** = Direct DOM manipulation = Faster canvas
2. **Smallest Bundle** = 80KB vs React 250KB
3. **Runes API** = Reactivity lebih powerful & simple
4. **Built-in Animations** = Smooth transitions out of the box

## Svelte 5 Runes: Game Changer

### Apa itu Runes?

Runes adalah **reactivity primitives** baru di Svelte 5 yang menggantikan `$:` syntax.

### Before (Svelte 4)

```svelte
<script>
  let count = 0;

  // Reactive statement
  $: doubled = count * 2;

  // Reactive block
  $: {
    console.log('Count changed:', count);
  }
</script>

<button on:click={() => count++}>
  {count} (doubled: {doubled})
</button>
```

### After (Svelte 5 Runes)

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);

  $effect(() => {
    console.log('Count changed:', count);
  });
</script>

<button onclick={() => count++}>
  {count} (doubled: {doubled})
</button>
```

**Lebih eksplisit, lebih powerful!**

## Canvas App dengan Svelte 5 Runes

### Project Structure

```
src/
├── routes/
│   └── +page.svelte
├── lib/
│   ├── components/
│   │   ├── Canvas.svelte
│   │   ├── Toolbar.svelte
│   │   └── LayerPanel.svelte
│   ├── stores/
│   │   └── canvas.svelte.ts  # Runes-based store
│   └── utils/
│       └── fabric.ts
```

### Canvas Store dengan Runes

```typescript
// lib/stores/canvas.svelte.ts
import type { Canvas } from "fabric";

class CanvasStore {
  canvas = $state<Canvas | null>(null);
  selectedTool = $state<"brush" | "eraser" | "select">("brush");
  brushSize = $state(20);
  brushColor = $state("#00F0FF");

  // Derived state
  isDrawing = $derived(
    this.selectedTool === "brush" || this.selectedTool === "eraser",
  );

  // Actions
  setCanvas(canvas: Canvas) {
    this.canvas = canvas;
  }

  selectTool(tool: typeof this.selectedTool) {
    this.selectedTool = tool;
  }

  setBrushSize(size: number) {
    this.brushSize = size;
    if (this.canvas) {
      this.canvas.freeDrawingBrush.width = size;
    }
  }
}

export const canvasStore = new CanvasStore();
```

### Canvas Component

```svelte
<!-- lib/components/Canvas.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { fabric } from 'fabric';
  import { canvasStore } from '$lib/stores/canvas.svelte';

  let canvasEl = $state<HTMLCanvasElement>();
  let fabricCanvas = $state<fabric.Canvas>();

  onMount(() => {
    if (!canvasEl) return;

    fabricCanvas = new fabric.Canvas(canvasEl, {
      width: 800,
      height: 600,
      backgroundColor: '#fff',
      isDrawingMode: true
    });

    canvasStore.setCanvas(fabricCanvas);

    return () => fabricCanvas?.dispose();
  });

  // Auto-update brush when store changes
  $effect(() => {
    if (!fabricCanvas) return;

    fabricCanvas.freeDrawingBrush.width = canvasStore.brushSize;
    fabricCanvas.freeDrawingBrush.color = canvasStore.brushColor;
  });
</script>

<div class="canvas-container">
  <canvas bind:this={canvasEl}></canvas>
</div>

<style>
  .canvas-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background: linear-gradient(135deg, #0A0E27 0%, #1A1F3A 100%);
  }

  canvas {
    border-radius: 12px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  }
</style>
```

### Toolbar Component

```svelte
<!-- lib/components/Toolbar.svelte -->
<script lang="ts">
  import { canvasStore } from '$lib/stores/canvas.svelte';

  const tools = [
    { id: 'brush', icon: '🖌️', label: 'Brush' },
    { id: 'eraser', icon: '🧹', label: 'Eraser' },
    { id: 'select', icon: '👆', label: 'Select' }
  ] as const;
</script>

<aside class="toolbar">
  <div class="tools">
    {#each tools as tool}
      <button
        class:active={canvasStore.selectedTool === tool.id}
        onclick={() => canvasStore.selectTool(tool.id)}
        aria-label={tool.label}
      >
        <span class="icon">{tool.icon}</span>
        <span class="label">{tool.label}</span>
      </button>
    {/each}
  </div>

  <div class="controls">
    <label>
      <span>Size: {canvasStore.brushSize}px</span>
      <input
        type="range"
        min="5"
        max="50"
        bind:value={canvasStore.brushSize}
      />
    </label>

    <label>
      <span>Color:</span>
      <input
        type="color"
        bind:value={canvasStore.brushColor}
      />
    </label>
  </div>
</aside>

<style>
  .toolbar {
    position: fixed;
    left: 20px;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(20, 24, 41, 0.95);
    backdrop-filter: blur(20px);
    padding: 20px;
    border-radius: 16px;
    border: 1px solid rgba(0, 240, 255, 0.2);
    display: flex;
    flex-direction: column;
    gap: 20px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
  }

  .tools {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  button {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 12px 16px;
    background: rgba(255, 255, 255, 0.05);
    border: 2px solid transparent;
    border-radius: 12px;
    color: #E8EAED;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  button:hover {
    background: rgba(255, 255, 255, 0.1);
    border-color: rgba(0, 240, 255, 0.3);
  }

  button.active {
    background: linear-gradient(135deg, #00F0FF 0%, #7B61FF 100%);
    border-color: #00F0FF;
    box-shadow: 0 4px 20px rgba(0, 240, 255, 0.4);
  }

  .controls {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding-top: 20px;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
  }

  label {
    display: flex;
    flex-direction: column;
    gap: 8px;
    color: #B8BFD8;
    font-size: 14px;
  }

  input[type="range"] {
    width: 100%;
  }

  input[type="color"] {
    width: 100%;
    height: 40px;
    border-radius: 8px;
    border: none;
    cursor: pointer;
  }
</style>
```

### Main Page

```svelte
<!-- routes/+page.svelte -->
<script lang="ts">
  import Canvas from '$lib/components/Canvas.svelte';
  import Toolbar from '$lib/components/Toolbar.svelte';
</script>

<svelte:head>
  <title>Canvas Studio - SvelteKit 5</title>
</svelte:head>

<main>
  <Toolbar />
  <Canvas />
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
  }
</style>
```

## Runes API Deep Dive

### 1. `$state()` - Reactive State

```typescript
let count = $state(0);
let user = $state({ name: "John", age: 30 });

// Deep reactivity by default!
user.age = 31; // Triggers reactivity
```

### 2. `$derived()` - Computed Values

```typescript
let count = $state(0);
let doubled = $derived(count * 2);
let quadrupled = $derived(doubled * 2);

// Auto-updates when count changes
```

### 3. `$effect()` - Side Effects

```typescript
let count = $state(0);

$effect(() => {
  console.log("Count is now:", count);
  document.title = `Count: ${count}`;
});
```

### 4. `$props()` - Component Props

```svelte
<script lang="ts">
  interface Props {
    title: string;
    count?: number;
  }

  let { title, count = 0 }: Props = $props();
</script>

<h1>{title}: {count}</h1>
```

## Comparison: React vs Svelte 5

### React (Verbose)

```tsx
import { useState, useEffect, useMemo } from "react";

function Counter() {
  const [count, setCount] = useState(0);
  const doubled = useMemo(() => count * 2, [count]);

  useEffect(() => {
    console.log("Count:", count);
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count} (doubled: {doubled})
    </button>
  );
}
```

### Svelte 5 (Concise)

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);

  $effect(() => {
    console.log('Count:', count);
  });
</script>

<button onclick={() => count++}>
  {count} (doubled: {doubled})
</button>
```

**50% less code!**

## Performance Comparison

### Bundle Size

| Framework    | Initial Bundle | Gzipped |
| ------------ | -------------- | ------- |
| **Svelte 5** | 80KB           | 25KB ✅ |
| **React 18** | 250KB          | 80KB    |
| **Vue 3**    | 150KB          | 50KB    |

### Runtime Performance

| Metric             | Svelte 5 | React | Vue 3 |
| ------------------ | -------- | ----- | ----- |
| **Initial Render** | 1.2ms    | 3.5ms | 2.8ms |
| **Update**         | 0.8ms    | 2.1ms | 1.5ms |
| **Memory**         | 2MB      | 5MB   | 3.5MB |

**Svelte 5 = 3x faster!**

## Advanced Canvas Features

### Undo/Redo dengan Runes

```typescript
// lib/stores/history.svelte.ts
class HistoryStore {
  past = $state<string[]>([]);
  future = $state<string[]>([]);

  canUndo = $derived(this.past.length > 0);
  canRedo = $derived(this.future.length > 0);

  save(state: string) {
    this.past.push(state);
    this.future = [];
  }

  undo() {
    if (!this.canUndo) return;
    const state = this.past.pop()!;
    this.future.push(state);
    return state;
  }

  redo() {
    if (!this.canRedo) return;
    const state = this.future.pop()!;
    this.past.push(state);
    return state;
  }
}

export const historyStore = new HistoryStore();
```

### Layer Management

```typescript
// lib/stores/layers.svelte.ts
interface Layer {
  id: string;
  name: string;
  visible: boolean;
  locked: boolean;
}

class LayersStore {
  layers = $state<Layer[]>([
    { id: "1", name: "Background", visible: true, locked: false },
  ]);

  activeLayerId = $state("1");

  activeLayer = $derived(this.layers.find((l) => l.id === this.activeLayerId));

  addLayer(name: string) {
    const id = Date.now().toString();
    this.layers.push({ id, name, visible: true, locked: false });
    this.activeLayerId = id;
  }

  toggleVisibility(id: string) {
    const layer = this.layers.find((l) => l.id === id);
    if (layer) layer.visible = !layer.visible;
  }
}

export const layersStore = new LayersStore();
```

## Why Svelte 5 > React for Canvas?

### 1. **No Virtual DOM**

- Direct DOM manipulation
- Faster canvas updates
- Less memory overhead

### 2. **Smaller Bundle**

- 80KB vs 250KB
- Faster initial load
- Better mobile performance

### 3. **Better DX**

- Less boilerplate
- More intuitive
- Faster development

### 4. **Built-in Features**

- Transitions
- Animations
- Scoped CSS

### 5. **Runes = Power**

- Explicit reactivity
- Better TypeScript support
- Easier debugging

## Real-World Example: Mini Canva

```svelte
<!-- routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { fabric } from 'fabric';

  let canvas = $state<fabric.Canvas>();
  let canvasEl = $state<HTMLCanvasElement>();

  // State
  let tool = $state<'draw' | 'text' | 'shape'>('draw');
  let color = $state('#00F0FF');
  let brushSize = $state(5);

  // Derived
  let isDrawMode = $derived(tool === 'draw');

  onMount(() => {
    if (!canvasEl) return;

    canvas = new fabric.Canvas(canvasEl, {
      width: 1200,
      height: 800,
      backgroundColor: '#fff'
    });

    return () => canvas?.dispose();
  });

  // Effects
  $effect(() => {
    if (!canvas) return;
    canvas.isDrawingMode = isDrawMode;
    if (isDrawMode) {
      canvas.freeDrawingBrush.width = brushSize;
      canvas.freeDrawingBrush.color = color;
    }
  });

  function addText() {
    if (!canvas) return;
    const text = new fabric.IText('Double click to edit', {
      left: 100,
      top: 100,
      fill: color
    });
    canvas.add(text);
  }

  function addRect() {
    if (!canvas) return;
    const rect = new fabric.Rect({
      left: 100,
      top: 100,
      width: 200,
      height: 100,
      fill: color
    });
    canvas.add(rect);
  }

  function clear() {
    canvas?.clear();
    canvas?.setBackgroundColor('#fff', canvas.renderAll.bind(canvas));
  }

  function download() {
    if (!canvas) return;
    const dataURL = canvas.toDataURL({ format: 'png' });
    const link = document.createElement('a');
    link.download = 'canvas.png';
    link.href = dataURL;
    link.click();
  }
</script>

<div class="app">
  <header>
    <h1>Canvas Studio</h1>
    <div class="tools">
      <button class:active={tool === 'draw'} onclick={() => tool = 'draw'}>
        🖌️ Draw
      </button>
      <button onclick={addText}>
        📝 Text
      </button>
      <button onclick={addRect}>
        ⬜ Shape
      </button>
      <input type="color" bind:value={color} />
      <input type="range" min="1" max="50" bind:value={brushSize} />
      <button onclick={clear}>🗑️ Clear</button>
      <button onclick={download}>💾 Download</button>
    </div>
  </header>

  <main>
    <canvas bind:this={canvasEl}></canvas>
  </main>
</div>

<style>
  .app {
    display: flex;
    flex-direction: column;
    height: 100vh;
    background: #0A0E27;
  }

  header {
    padding: 20px;
    background: rgba(20, 24, 41, 0.95);
    border-bottom: 1px solid rgba(0, 240, 255, 0.2);
  }

  h1 {
    color: #00F0FF;
    margin: 0 0 16px 0;
  }

  .tools {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
  }

  button {
    padding: 8px 16px;
    background: rgba(255, 255, 255, 0.1);
    border: 2px solid transparent;
    border-radius: 8px;
    color: white;
    cursor: pointer;
    transition: all 0.3s;
  }

  button:hover {
    background: rgba(255, 255, 255, 0.2);
  }

  button.active {
    background: linear-gradient(135deg, #00F0FF, #7B61FF);
    border-color: #00F0FF;
  }

  main {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
  }

  canvas {
    border-radius: 12px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  }
</style>
```

## Quick Start

```bash
# Create SvelteKit 5 project
npm create svelte@latest canvas-app
cd canvas-app

# Install dependencies
npm install
npm install fabric

# Run dev
npm run dev
```

## Kesimpulan

### ✅ Gunakan SvelteKit 5 Jika:

- Canvas/drawing apps
- Image editors
- Interactive dashboards
- Butuh performance maksimal
- Tim kecil, fokus DX

### ✅ Gunakan React Jika:

- Butuh mature ecosystem
- Easy hiring
- Enterprise scale
- Banyak third-party libraries

### 🏆 Winner untuk Canvas App?

**SvelteKit 5 dengan Runes!**

- 3x lebih kecil
- 3x lebih cepat
- 50% less code
- Better DX

## Resources

- [Svelte 5 Docs](https://svelte-5-preview.vercel.app/)
- [Runes RFC](https://github.com/sveltejs/rfcs/blob/master/text/0000-runes.md)
- [Fabric.js](http://fabricjs.com/)

**Next**: Mau artikel tentang migrasi React ke Svelte 5? Drop comment!

**Tags**: #svelte #sveltekit #svelte5 #runes #canvas #fabricjs #webdev
