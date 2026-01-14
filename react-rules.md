# React Rules: Suspense, use, and useTransition for Data Fetching

> Source: [React 19 Async Example - jherr/react-19-2-async](https://github.com/jherr/react-19-2-async/blob/main/src/routes/index.tsx)

## Overview

React 19 introduces powerful features for handling asynchronous data fetching and transitions: `Suspense`, the `use()` hook, and `useTransition()`. These work together to create smooth, declarative async data loading experiences.

---

## Core Concepts

### The Pattern: Promise → Suspense → use()

1. **Create a Promise** - Fetch data and return a Promise
2. **Store Promise in State** - Keep the Promise in component state
3. **Wrap with Suspense** - Use Suspense boundary to handle loading
4. **Unwrap with use()** - Use the `use()` hook to extract data from Promise

---

## Suspense

### Purpose

`Suspense` allows you to declaratively handle loading states for async operations. When a component suspends (throws a promise), Suspense shows the fallback UI.

### Basic Usage

```typescript
import { Suspense } from "react";

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <AsyncComponent />
    </Suspense>
  );
}
```

### Rules

- **Always provide a fallback** - Suspense requires a fallback prop
- **Fallback should match structure** - Keep similar layout to prevent layout shift
- **Nested Suspense is allowed** - You can have multiple Suspense boundaries
- **Suspense catches promises** - Components using `use()` will suspend when promise is pending

### Example: Loading State Pattern

```typescript
function App() {
  const [imageDataPromise, setImageDataPromise] = useState<Promise<ImageData>>(
    () => fetchImage(1)
  );

  return (
    <Suspense fallback={<ImageSkeleton imageId={imageId} />}>
      <Image imageDataPromise={imageDataPromise} imageId={imageId} />
    </Suspense>
  );
}
```

**Key Point:** The fallback should match the structure of the content being loaded to prevent layout shift.

---

## use() Hook

### Purpose

The `use()` hook unwraps Promises and Context values. For data fetching, it extracts data from a Promise and automatically suspends the component while the Promise is pending.

### Basic Usage

```typescript
import { use } from "react";

function Component({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise);
  return <div>{data.title}</div>;
}
```

### Rules

- **Must be called unconditionally** - `use()` cannot be inside conditionals or loops
- **Automatically suspends** - Component suspends while promise is pending
- **Throws on rejection** - Rejected promises need error boundaries
- **Can be called multiple times** - Use multiple `use()` calls for different promises
- **Works with Context too** - Can also unwrap React Context values

### Example: Unwrapping Promise Data

```typescript
function Image({
  imageDataPromise,
  imageId,
}: {
  imageDataPromise: Promise<ImageData>;
  imageId: number;
}) {
  // use() unwraps the promise and suspends if not ready
  const image = use(imageDataPromise);

  return (
    <div>
      <h2>{image.title}</h2>
      <img src={image.url} alt={image.title} />
    </div>
  );
}
```

### Error Handling

```typescript
import { use } from "react";
import { ErrorBoundary } from "react-error-boundary";

function Component({ dataPromise }: { dataPromise: Promise<Data> }) {
  try {
    const data = use(dataPromise);
    return <div>{data.title}</div>;
  } catch (error) {
    // Handle error or let ErrorBoundary catch it
    throw error;
  }
}

// Wrap with ErrorBoundary
<ErrorBoundary fallback={<ErrorUI />}>
  <Suspense fallback={<Loading />}>
    <Component dataPromise={promise} />
  </Suspense>
</ErrorBoundary>;
```

---

## useTransition

### Purpose

`useTransition()` marks state updates as transitions, allowing React to keep the current UI responsive while preparing the new state. It's perfect for non-urgent updates like data fetching.

### Basic Usage

```typescript
import { useTransition } from "react";

function Component() {
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(() => {
      // State updates here are marked as transitions
      setState(newValue);
    });
  };

  return (
    <button disabled={isPending} onClick={handleClick}>
      {isPending ? "Loading..." : "Click me"}
    </button>
  );
}
```

### Rules

- **Returns array of [isPending, startTransition]** - Destructure to get both values
- **isPending indicates transition state** - Use to show loading indicators
- **startTransition wraps updates** - Mark non-urgent state updates
- **Keeps UI responsive** - Current UI stays interactive during transition
- **Can wrap async functions** - Works with async/await

### Example: Button with Transition

```typescript
function Button({
  action,
  children,
}: {
  action: () => void;
  children: React.ReactNode;
}) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await action();
        });
      }}
      className={isPending ? "disabled" : ""}
    >
      {children}
    </button>
  );
}
```

### Combining with Data Fetching

```typescript
function App() {
  const [imageId, setImageId] = useState(1);
  const [imageDataPromise, setImageDataPromise] = useState<Promise<ImageData>>(
    () => fetchImage(imageId)
  );
  const [isPending, startTransition] = useTransition();

  const handleNext = () => {
    startTransition(() => {
      const nextId = imageId + 1;
      setImageId(nextId);
      setImageDataPromise(fetchImage(nextId));
    });
  };

  return (
    <div>
      <button disabled={isPending} onClick={handleNext}>
        Next Image
      </button>
      <Suspense fallback={<ImageSkeleton imageId={imageId} />}>
        <Image imageDataPromise={imageDataPromise} imageId={imageId} />
      </Suspense>
    </div>
  );
}
```

---

## Complete Data Fetching Pattern

### Step-by-Step Implementation

```typescript
import { Suspense, use, useTransition, useState } from "react";

// 1. Create async data fetching function
async function fetchImage(id: number): Promise<ImageData> {
  await new Promise((r) => setTimeout(r, 800)); // Simulate network delay
  return { id, url: `/images/${id}.jpg`, title: `Image ${id}` };
}

// 2. Main component manages promise state
function App() {
  const [imageId, setImageId] = useState(1);
  const [imageDataPromise, setImageDataPromise] = useState<Promise<ImageData>>(
    () => fetchImage(imageId) // Initialize with promise
  );
  const [isPending, startTransition] = useTransition();

  const handleNext = () => {
    startTransition(() => {
      const nextId = imageId + 1;
      setImageId(nextId);
      // Create new promise for new data
      setImageDataPromise(fetchImage(nextId));
    });
  };

  return (
    <div>
      <Button action={handleNext} isPending={isPending}>
        Next Image
      </Button>
      {/* 3. Wrap async component with Suspense */}
      <Suspense fallback={<ImageSkeleton imageId={imageId} />}>
        <Image imageDataPromise={imageDataPromise} imageId={imageId} />
      </Suspense>
    </div>
  );
}

// 4. Component uses use() to unwrap promise
function Image({
  imageDataPromise,
  imageId,
}: {
  imageDataPromise: Promise<ImageData>;
  imageId: number;
}) {
  const image = use(imageDataPromise); // Suspends if promise pending

  return (
    <div>
      <h2>{image.title}</h2>
      <img src={image.url} alt={image.title} />
    </div>
  );
}

// 5. Loading fallback component
function ImageSkeleton({ imageId }: { imageId: number }) {
  return (
    <div>
      <h2>Loading...</h2>
      <div className="skeleton" />
    </div>
  );
}
```

---

## Best Practices

### 1. Store Promises in State, Not Data

```typescript
// ✅ GOOD: Store the promise
const [dataPromise, setDataPromise] = useState(() => fetchData(id));

// ❌ BAD: Don't store the data directly
const [data, setData] = useState(null);
useEffect(() => {
  fetchData(id).then(setData);
}, [id]);
```

### 2. Create New Promises for Each Fetch

```typescript
// ✅ GOOD: Create new promise when ID changes
const handleNext = () => {
  startTransition(() => {
    const nextId = imageId + 1;
    setImageDataPromise(fetchImage(nextId)); // New promise
  });
};

// ❌ BAD: Reusing same promise
const promise = fetchImage(1);
setImageDataPromise(promise); // Won't trigger re-render
```

### 3. Match Fallback Structure

```typescript
// ✅ GOOD: Fallback matches content structure
<Suspense fallback={<ImageSkeleton imageId={imageId} />}>
  <Image imageDataPromise={promise} imageId={imageId} />
</Suspense>

// ❌ BAD: Different structure causes layout shift
<Suspense fallback={<div>Loading...</div>}>
  <Image imageDataPromise={promise} imageId={imageId} />
</Suspense>
```

### 4. Use useTransition for User Actions

```typescript
// ✅ GOOD: Wrap user-initiated updates
startTransition(() => {
  setImageId(nextId);
  setImageDataPromise(fetchImage(nextId));
});

// ❌ BAD: Don't use for urgent updates (like user input)
// Urgent updates should happen immediately without transition
```

### 5. Handle Errors with Error Boundaries

```typescript
import { ErrorBoundary } from "react-error-boundary";

<ErrorBoundary fallback={<ErrorUI />}>
  <Suspense fallback={<Loading />}>
    <Component dataPromise={promise} />
  </Suspense>
</ErrorBoundary>;
```

---

## Common Patterns

### Pattern 1: Sequential Data Loading

```typescript
function SequentialData({ userId }: { userId: number }) {
  const userPromise = useMemo(() => fetchUser(userId), [userId]);
  const postsPromise = useMemo(() => fetchPosts(userId), [userId]);

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
      <Suspense fallback={<PostsLoading />}>
        <PostsList postsPromise={postsPromise} />
      </Suspense>
    </Suspense>
  );
}
```

### Pattern 2: Dependent Data Loading

```typescript
function DependentData({ userId }: { userId: number }) {
  const userPromise = useMemo(() => fetchUser(userId), [userId]);

  return (
    <Suspense fallback={<Loading />}>
      <UserData userPromise={userPromise} />
    </Suspense>
  );
}

function UserData({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  const postsPromise = useMemo(() => fetchPosts(user.id), [user.id]);

  return (
    <div>
      <h1>{user.name}</h1>
      <Suspense fallback={<PostsLoading />}>
        <PostsList postsPromise={postsPromise} />
      </Suspense>
    </div>
  );
}
```

### Pattern 3: Optimistic Updates with Transitions

```typescript
function OptimisticUpdate() {
  const [isPending, startTransition] = useTransition();
  const [dataPromise, setDataPromise] = useState(() => fetchData());

  const handleUpdate = async (newData: Data) => {
    // Optimistically update UI
    setDataPromise(Promise.resolve(newData));

    startTransition(async () => {
      // Then sync with server
      const serverData = await saveData(newData);
      setDataPromise(Promise.resolve(serverData));
    });
  };

  return (
    <Suspense fallback={<Loading />}>
      <DataComponent dataPromise={dataPromise} />
    </Suspense>
  );
}
```

### Pattern 4: Search with Debouncing

```typescript
function SearchComponent() {
  const [query, setQuery] = useState("");
  const [isPending, startTransition] = useTransition();
  const [resultsPromise, setResultsPromise] = useState(() =>
    Promise.resolve([])
  );

  const handleSearch = (newQuery: string) => {
    setQuery(newQuery); // Update input immediately

    startTransition(() => {
      // Debounce search
      const timeoutId = setTimeout(() => {
        setResultsPromise(search(newQuery));
      }, 300);

      return () => clearTimeout(timeoutId);
    });
  };

  return (
    <div>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      <Suspense fallback={<ResultsLoading />}>
        <Results resultsPromise={resultsPromise} />
      </Suspense>
    </div>
  );
}
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Calling use() Conditionally

```typescript
// ❌ WRONG
function Component({ promise }: { promise?: Promise<Data> }) {
  if (!promise) return null;
  const data = use(promise); // Error: use() called conditionally
  return <div>{data.title}</div>;
}

// ✅ CORRECT
function Component({ promise }: { promise?: Promise<Data> }) {
  if (!promise) return null;
  // Always call use() unconditionally
  const data = use(promise!);
  return <div>{data.title}</div>;
}
```

### ❌ Mistake 2: Not Wrapping with Suspense

```typescript
// ❌ WRONG
function Component({ promise }: { promise: Promise<Data> }) {
  const data = use(promise); // Will throw without Suspense
  return <div>{data.title}</div>;
}

// ✅ CORRECT
<Suspense fallback={<Loading />}>
  <Component promise={promise} />
</Suspense>;
```

### ❌ Mistake 3: Reusing Same Promise

```typescript
// ❌ WRONG
const promise = fetchData(1);
setDataPromise(promise);
setDataPromise(promise); // Same promise, won't trigger update

// ✅ CORRECT
setDataPromise(fetchData(1));
setDataPromise(fetchData(2)); // New promise triggers update
```

### ❌ Mistake 4: Not Using useTransition for Non-Urgent Updates

```typescript
// ❌ WRONG: Blocks UI during data fetch
const handleNext = () => {
  setImageId(nextId);
  setImageDataPromise(fetchImage(nextId)); // Blocks UI
};

// ✅ CORRECT: Keeps UI responsive
const handleNext = () => {
  startTransition(() => {
    setImageId(nextId);
    setImageDataPromise(fetchImage(nextId)); // Non-blocking
  });
};
```

---

## TypeScript Types

### Promise Type Definitions

```typescript
type ImageData = {
  id: number;
  url: string;
  title: string;
};

// Component props with Promise
interface ImageProps {
  imageDataPromise: Promise<ImageData>;
  imageId: number;
}

// Generic Promise Component
interface AsyncComponentProps<T> {
  dataPromise: Promise<T>;
  fallback?: React.ReactNode;
}
```

### Generic Async Component Pattern

```typescript
function AsyncComponent<T>({
  dataPromise,
  children,
}: {
  dataPromise: Promise<T>;
  children: (data: T) => React.ReactNode;
}) {
  const data = use(dataPromise);
  return <>{children(data)}</>;
}

// Usage
<AsyncComponent dataPromise={fetchUser(1)}>
  {(user) => <div>{user.name}</div>}
</AsyncComponent>;
```

---

## Testing

### Setup

```typescript
import { render, screen, waitFor, act } from "@testing-library/react";
import { Suspense } from "react";
import { describe, it, expect, jest, beforeEach } from "@jest/globals";
```

### Testing Components with use()

#### Basic Test: Resolved Promise

```typescript
describe("Image Component", () => {
  it("renders image data when promise resolves", async () => {
    const mockImageData: ImageData = {
      id: 1,
      url: "/images/1.jpg",
      title: "Test Image",
    };

    const promise = Promise.resolve(mockImageData);

    render(
      <Suspense fallback={<div>Loading...</div>}>
        <Image imageDataPromise={promise} imageId={1} />
      </Suspense>
    );

    // Wait for promise to resolve and component to render
    await waitFor(() => {
      expect(screen.getByText("Test Image")).toBeInTheDocument();
    });

    expect(screen.getByAltText("Test Image")).toHaveAttribute(
      "src",
      "/images/1.jpg"
    );
  });
});
```

#### Test: Loading State (Suspense Fallback)

```typescript
describe("Image Component with Suspense", () => {
  it("shows loading fallback while promise is pending", async () => {
    // Create a promise that doesn't resolve immediately
    let resolvePromise: (value: ImageData) => void;
    const promise = new Promise<ImageData>((resolve) => {
      resolvePromise = resolve;
    });

    render(
      <Suspense fallback={<div data-testid="loading">Loading...</div>}>
        <Image imageDataPromise={promise} imageId={1} />
      </Suspense>
    );

    // Should show loading state initially
    expect(screen.getByTestId("loading")).toBeInTheDocument();

    // Resolve the promise
    await act(async () => {
      resolvePromise!({
        id: 1,
        url: "/images/1.jpg",
        title: "Test Image",
      });
    });

    // Wait for component to render with data
    await waitFor(() => {
      expect(screen.queryByTestId("loading")).not.toBeInTheDocument();
      expect(screen.getByText("Test Image")).toBeInTheDocument();
    });
  });
});
```

#### Test: Error Handling

```typescript
describe("Image Component Error Handling", () => {
  it("handles promise rejection with error boundary", async () => {
    const promise = Promise.reject(new Error("Failed to load image"));

    const ErrorFallback = ({ error }: { error: Error }) => (
      <div data-testid="error">{error.message}</div>
    );

    // In a real app, you'd use react-error-boundary
    render(
      <ErrorBoundary fallback={<ErrorFallback error={new Error("Failed")} />}>
        <Suspense fallback={<div>Loading...</div>}>
          <Image imageDataPromise={promise} imageId={1} />
        </Suspense>
      </ErrorBoundary>
    );

    await waitFor(() => {
      expect(screen.getByTestId("error")).toBeInTheDocument();
    });
  });
});
```

### Testing useTransition()

#### Test: Transition State

```typescript
describe("Button with useTransition", () => {
  it("shows pending state during transition", async () => {
    const mockAction = jest.fn(() => Promise.resolve());

    render(<Button action={mockAction}>Next Image</Button>);

    const button = screen.getByRole("button", { name: /next image/i });

    // Click button
    await act(async () => {
      button.click();
    });

    // Button should be disabled during transition
    expect(button).toBeDisabled();

    // Wait for transition to complete
    await waitFor(() => {
      expect(button).not.toBeDisabled();
    });

    expect(mockAction).toHaveBeenCalled();
  });
});
```

#### Test: Multiple Rapid Clicks

```typescript
describe("Button with useTransition", () => {
  it("handles rapid clicks gracefully", async () => {
    let resolveCount = 0;
    const mockAction = jest.fn(
      () =>
        new Promise<void>((resolve) => {
          setTimeout(() => {
            resolveCount++;
            resolve();
          }, 100);
        })
    );

    render(<Button action={mockAction}>Next</Button>);

    const button = screen.getByRole("button");

    // Click multiple times rapidly
    await act(async () => {
      button.click();
      button.click();
      button.click();
    });

    // Should handle all clicks
    await waitFor(() => {
      expect(resolveCount).toBeGreaterThan(0);
    });
  });
});
```

### Testing Complete Data Fetching Pattern

#### Test: Full App Flow

```typescript
describe("App with Suspense and useTransition", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it("loads and displays image data", async () => {
    const mockFetchImage = jest.fn((id: number) =>
      Promise.resolve({
        id,
        url: `/images/${id}.jpg`,
        title: `Image ${id}`,
      })
    );

    // Mock the fetch function
    jest.mock("./api", () => ({
      fetchImage: mockFetchImage,
    }));

    render(<App />);

    // Initial load should show loading
    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    // Wait for initial image to load
    await waitFor(() => {
      expect(screen.getByText("Image 1")).toBeInTheDocument();
    });

    // Click next button
    const nextButton = screen.getByRole("button", { name: /next image/i });
    await act(async () => {
      nextButton.click();
    });

    // Should show loading again
    await waitFor(() => {
      expect(screen.getByText(/loading/i)).toBeInTheDocument();
    });

    // Wait for next image to load
    await waitFor(() => {
      expect(screen.getByText("Image 2")).toBeInTheDocument();
    });

    expect(mockFetchImage).toHaveBeenCalledWith(2);
  });
});
```

### Mocking Promises

#### Helper: Create Controllable Promise

```typescript
function createControllablePromise<T>() {
  let resolve: (value: T) => void;
  let reject: (error: Error) => void;

  const promise = new Promise<T>((res, rej) => {
    resolve = res;
    reject = rej;
  });

  return {
    promise,
    resolve: resolve!,
    reject: reject!,
  };
}

// Usage in tests
describe("Controllable Promise", () => {
  it("allows manual control of promise resolution", async () => {
    const { promise, resolve } = createControllablePromise<ImageData>();

    render(
      <Suspense fallback={<div>Loading...</div>}>
        <Image imageDataPromise={promise} imageId={1} />
      </Suspense>
    );

    expect(screen.getByText("Loading...")).toBeInTheDocument();

    await act(async () => {
      resolve({
        id: 1,
        url: "/images/1.jpg",
        title: "Test Image",
      });
    });

    await waitFor(() => {
      expect(screen.getByText("Test Image")).toBeInTheDocument();
    });
  });
});
```

### Testing Suspense Boundaries

#### Test: Nested Suspense

```typescript
describe("Nested Suspense", () => {
  it("shows appropriate fallback for nested boundaries", async () => {
    const userPromise = Promise.resolve({ id: 1, name: "John" });
    const postsPromise = new Promise(() => {}); // Never resolves

    render(
      <Suspense fallback={<div>Loading user...</div>}>
        <UserProfile userPromise={userPromise} />
        <Suspense fallback={<div>Loading posts...</div>}>
          <PostsList postsPromise={postsPromise} />
        </Suspense>
      </Suspense>
    );

    // User should load
    await waitFor(() => {
      expect(screen.getByText("John")).toBeInTheDocument();
    });

    // Posts should show loading
    expect(screen.getByText("Loading posts...")).toBeInTheDocument();
    expect(screen.queryByText("Loading user...")).not.toBeInTheDocument();
  });
});
```

### Testing with React Testing Library Utilities

#### Using findBy Queries

```typescript
describe("Async Component Testing", () => {
  it("uses findBy queries for async content", async () => {
    const promise = Promise.resolve({
      id: 1,
      title: "Async Title",
    });

    render(
      <Suspense fallback={<div>Loading...</div>}>
        <Component dataPromise={promise} />
      </Suspense>
    );

    // findBy queries automatically wait
    const title = await screen.findByText("Async Title");
    expect(title).toBeInTheDocument();
  });
});
```

#### Using waitFor with Custom Conditions

```typescript
describe("Custom Wait Conditions", () => {
  it("waits for custom conditions", async () => {
    const promise = Promise.resolve({ count: 5 });

    render(
      <Suspense fallback={<div>Loading...</div>}>
        <Counter dataPromise={promise} />
      </Suspense>
    );

    await waitFor(
      () => {
        const counter = screen.getByTestId("counter");
        expect(counter).toHaveTextContent("5");
      },
      { timeout: 3000 }
    );
  });
});
```

### Testing Best Practices

#### 1. Always Wrap with Suspense

```typescript
// ✅ GOOD: Always include Suspense in tests
render(
  <Suspense fallback={<div>Loading...</div>}>
    <Component dataPromise={promise} />
  </Suspense>
);

// ❌ BAD: Missing Suspense will cause errors
render(<Component dataPromise={promise} />);
```

#### 2. Use act() for State Updates

```typescript
// ✅ GOOD: Wrap state updates in act()
await act(async () => {
  resolvePromise(data);
});

// ❌ BAD: State updates outside act() may cause warnings
resolvePromise(data);
```

#### 3. Clean Up Promises

```typescript
describe("Component", () => {
  let resolvePromise: (value: Data) => void;

  beforeEach(() => {
    const promise = new Promise<Data>((resolve) => {
      resolvePromise = resolve;
    });
    // Use promise in tests
  });

  afterEach(() => {
    // Clean up any pending promises
    jest.clearAllTimers();
  });
});
```

#### 4. Mock External Dependencies

```typescript
// Mock fetch function
jest.mock("./api", () => ({
  fetchImage: jest.fn((id: number) =>
    Promise.resolve({
      id,
      url: `/images/${id}.jpg`,
      title: `Image ${id}`,
    })
  ),
}));
```

### Complete Test Example

```typescript
import { render, screen, waitFor, act } from "@testing-library/react";
import { Suspense, use, useState, useTransition } from "react";
import { describe, it, expect, jest, beforeEach } from "@jest/globals";

type ImageData = {
  id: number;
  url: string;
  title: string;
};

async function fetchImage(id: number): Promise<ImageData> {
  await new Promise((r) => setTimeout(r, 100));
  return { id, url: `/images/${id}.jpg`, title: `Image ${id}` };
}

function Image({
  imageDataPromise,
}: {
  imageDataPromise: Promise<ImageData>;
}) {
  const image = use(imageDataPromise);
  return (
    <div>
      <h2>{image.title}</h2>
      <img src={image.url} alt={image.title} />
    </div>
  );
}

function App() {
  const [imageId, setImageId] = useState(1);
  const [imageDataPromise, setImageDataPromise] = useState<Promise<ImageData>>(
    () => fetchImage(imageId)
  );
  const [isPending, startTransition] = useTransition();

  const handleNext = () => {
    startTransition(() => {
      const nextId = imageId + 1;
      setImageId(nextId);
      setImageDataPromise(fetchImage(nextId));
    });
  };

  return (
    <div>
      <button onClick={handleNext} disabled={isPending}>
        Next Image
      </button>
      <Suspense fallback={<div>Loading...</div>}>
        <Image imageDataPromise={imageDataPromise} />
      </Suspense>
    </div>
  );
}

describe("App Integration Test", () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.runOnlyPendingTimers();
    jest.useRealTimers();
  });

  it("loads initial image and transitions to next", async () => {
    render(<App />);

    // Initial loading state
    expect(screen.getByText("Loading...")).toBeInTheDocument();

    // Fast-forward timers
    await act(async () => {
      jest.advanceTimersByTime(100);
    });

    // Wait for initial image
    await waitFor(() => {
      expect(screen.getByText("Image 1")).toBeInTheDocument();
    });

    // Click next button
    const button = screen.getByRole("button");
    await act(async () => {
      button.click();
      jest.advanceTimersByTime(100);
    });

    // Should show loading again
    expect(screen.getByText("Loading...")).toBeInTheDocument();

    // Wait for next image
    await waitFor(() => {
      expect(screen.getByText("Image 2")).toBeInTheDocument();
    });
  });
});
```

---

## Summary

### Key Takeaways

1. **Suspense** - Declarative loading states, always provide fallback
2. **use()** - Unwraps promises, automatically suspends, must be unconditional
3. **useTransition()** - Marks non-urgent updates, keeps UI responsive
4. **Store promises in state** - Not the resolved data
5. **Create new promises** - For each fetch operation
6. **Match fallback structure** - Prevent layout shift
7. **Use Error Boundaries** - Handle promise rejections

### The Complete Flow

```
User Action → useTransition → Update State with New Promise
                                    ↓
                            Component Suspends
                                    ↓
                            Suspense Shows Fallback
                                    ↓
                            Promise Resolves
                                    ↓
                            use() Returns Data
                                    ↓
                            Component Renders
```

This pattern provides a smooth, declarative way to handle async data fetching in React 19!
