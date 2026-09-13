# 🔄 The Ultimate Deep Dive: React Lifecycle from First Render to State/Effect Updates

This comprehensive guide tracks exactly what React does behind the scenes, step-by-step, during the
**Initial Render**, a **State Update**, and an **Effect Trigger**.

---

## 🚀 Phase 1: The Initial Render (Mounting)

This is what happens when your application boots up and mounts a component to the screen for the
first time.

### Step 1.1: Triggering the Root Render

- **The Action:** Your entry file calls `root.render(<App />)`.
- **Behind the Scenes:** React initializes the internal container infrastructure. It builds the very
  first root node of the **Fiber Tree** (`HostRoot`), which serves as the permanent anchor point for
  your entire application.

### Step 1.2: Component Execution & The Element Tree

- **The Action:** React executes your functional components from top to bottom, beginning with `<App
/>`.
- **Behind the Scenes:**
  - As code runs, any `useState` hooks encountered are allocated their initial values. These values
        are stored inside an internal linked list of hook objects attached to the component's Fiber.
  - The components return JSX. Your build tools compile this JSX into nested `React.createElement()`
      calls, instantly producing the **React Element Tree**—the lightweight JSON snapshot describing
      your target UI.

### Step 1.3: Constructing the Fiber Tree

- **The Action:** React uses the brand-new Element Tree to construct the persistent **Fiber Tree**.
- **Behind the Scenes:** For every JSON object in the Element Tree, React spawns a mutable `Fiber`
  node. This node keeps track of the component type, state, hooks, pending updates, and its
  corresponding browser DOM element. React weaves these nodes together using pointers (`child`,
  `sibling`, and `return`).

### Step 1.4: Target Mapping (The Render Phase)

- **The Action:** React completes the initial reconciliation pass.
- **Behind the Scenes:** Because there is no previous tree to compare against, React creates a new
  browser DOM node in memory for every primitive element (like `div` or `button`). It attaches these
  elements directly to the `stateNode` property of their respective Fibers and tags every node with
  a `Placement` flag.

### Step 1.5: Flushing to the Screen (The Commit Phase)

- **The Action:** React injects the fully constructed DOM tree into the actual browser container
  (e.g., `<div id="root"></div>`).
- **Behind the Scenes:** React works through the Fiber tree synchronously, executing all `Placement`
  mutations. The real browser DOM is updated all at once, forcing the browser to calculate layout
  and paint the UI onto the physical screen.

---

## 🔄 Phase 2: The State Update Lifecycle (Re-rendering)

This cycle triggers when a user interacts with the UI, firing a state setter (e.g., `setCount(count + 1)`).

### Step 2.1: Enqueueing the Update

- **The Action:** A state setter function is executed inside a component.
- **Behind the Scenes:**
  - React doesn't immediately stop everything to re-render. Instead, it creates an **Update Object**
    containing the new value or action.
  - It pushes this update object into a queue on the component's specific Fiber node.
  - React schedules a micro-task in the browser event loop, marking this component and its ancestors
    as having "pending work".

### Step 2.2: Double-Buffering Setup (`workInProgress`)

- **The Action:** The scheduled micro-task runs, kicking off the Render Phase.
- **Behind the Scenes:** React uses a performance optimization technique called
  **Double-Buffering**. It keeps the current, visible Fiber tree completely intact. It then clones
  the root node to start building a parallel tree called the `workInProgress` tree. This ensures a
  broken render won't cause a flickering, half-broken UI on screen.

### Step 2.3: Re-executing Components (Top-Down Diffing)

- **The Action:** React begins walking down the `workInProgress` tree, starting at the root.
- **Behind the Scenes:**
  - If a component's state or props haven't changed, React safely bypasses it.
  - Once it reaches the component that triggered the state update, it executes that component
    function again.
  - During execution, the component encounters the `useState` hook. React reads the pending update
    queue on the Fiber node, calculates the new state value, and returns it.
  - The component returns a brand-new, temporary **React Element Tree** snapshot reflecting the new state.

### Step 2.4: Reconciliation (The Diffing Engine)

- **The Action:** React compares the brand-new Element Tree snapshot against the old, stable Fiber
  tree.
- **Behind the Scenes:** React applies its core diffing heuristics node-by-node:
  - **Same Element Type:** If a node is still a `<div>`, it reuses the existing Fiber and browser
    DOM node, simply mapping over the modified props (like changing text content or a `className`).
    It tags the node with an `Update` flag.
  - **Different Element Type:** If a `<div>` was replaced by a `<section>`, React immediately tags
    the old Fiber and its entire sub-tree for `Deletion`, throwing away their states, and marks a
    brand-new Fiber for `Placement`.
  - **Keys Check:** For lists, React matches elements using their `key` props to determine if items
    merely shifted positions, avoiding tearing down elements unnecessarily.

### Step 2.5: Committing mutations to the Real DOM

- **The Action:** The `workInProgress` tree is fully calculated and ready to go live.
- **Behind the Scenes:** React enters the Commit Phase. In a single, synchronous pass, it applies
  the accumulated flags (`Update`, `Placement`, `Deletion`) directly to the **Real Browser DOM**.
  - If only a single text character changed inside an `<h1>`, _only_ that precise text property is
    mutated in the browser DOM. The rest of the surrounding elements are left completely untouched.
  - Once the mutations finish, the `workInProgress` tree instantly swap roles to become the current,
    stable Fiber tree.

---

## ⚡ Phase 3: The Effect Lifecycle (`useEffect`)

Effects run completely outside the main rendering pipeline to prevent slow operations (like fetching
data or hitting APIs) from blocking visual painting.

### Step 3.1: Registering Dependencies

- **The Action:** During the Render Phase, React encounters a `useEffect(callback, [deps])` block.
- **Behind the Scenes:**
  - React checks the hook's index inside the component's Fiber hooks list.
  - It extracts the current dependency array values and compares them shallowly (`Object.is`)
    against the dependency values saved from the _previous_ render.
  - If the dependencies have changed (or if no dependency array was provided), React tags this
    effect hook with a specialized side-effect flag (`HasEffect`).

### Step 3.2: Passing Visual Control to the Browser

- **The Action:** The Commit Phase completes, changing the visual DOM layout.
- **Behind the Scenes:** React actively relinquishes control back to the browser. The browser
  handles layout reflow, parses the styling rules, and paints the new pixels onto the monitor. This
  ensures the user sees the updated UI immediately, without any lag.

### Step 3.3: Executing Cleanups (Asynchronous)

- **The Action:** Immediately _after_ the browser paints the new screen, React initiates its
  asynchronous side-effect queue.
- **Behind the Scenes:** Before running the new effect logic, React looks at the previous render's
  records. If the effect ran previously and provided a returned cleanup function, React executes
  that cleanup function right now. This safely closes old database streams, clears active timers, or
  removes stale event listeners.

### Step 3.4: Running the New Effect Action

- **The Action:** The cleanup phase finishes.
- **Behind the Scenes:** React fires the actual callback function inside your `useEffect`. If your
  effect happens to trigger _another_ state update inside its callback, the entire cycle loops right
  back up to **Phase 2, Step 2.1**, queueing a brand-new re-render.
