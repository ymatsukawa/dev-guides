# Hooks & Side Effects
This document describes the essence of Hooks & Side Effects in React and the recommendations and deprecations that support it.

## Essence of Hooks & Side Effects

In one sentence, Hooks & Side Effects is about managing stateful logic and external interactions through proper lifecycle control and stable references. When using this, follow these philosophies:

- [ ] Custom hooks should be optimized to prevent unnecessary re-renders
- [ ] Debounce/throttle functions must be memoized to work correctly
- [ ] Refs should only be used for DOM access and non-render data

## Recommendations

**Use Refs only for DOM access and internal data storage**:

Refs are specialized for accessing actual DOM elements and storing values that don't trigger re-renders (like timer IDs or previous state values).

Use Refs for declaratively impossible operations like focus, scroll, or size measurements. Store data that doesn't directly affect UI rendering. Access Refs only in useEffect or event handlers, not during render.

```javascript
// DOM access example
const Form = () => {
  const [name, setName] = useState('');
  const inputRef = useRef(null);
  
  const onSubmitClick = () => {
    if (!name && inputRef.current) {
      inputRef.current.focus(); // Direct DOM manipulation
    } else {
      // Submit form data
    }
  };
  
  return (
    <>
      <input 
        ref={inputRef}
        value={name}
        onChange={(e) => setName(e.target.value)} 
      />
      <button onClick={onSubmitClick}>Submit</button>
    </>
  );
};

// Storing non-render data
const Timer = () => {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null); // Store timer ID
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);
  
  const pause = () => {
    clearInterval(intervalRef.current);
  };
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={pause}>Pause</button>
    </div>
  );
};
```

**Leverage Context for performance optimization**:

Context avoids prop drilling and enables direct data passing to deeply nested components, preventing unnecessary re-renders of intermediate components.

Always memoize Context Provider value objects with useMemo and useCallback. Split Context into smaller, granular providers to minimize consumer re-render frequency.

```javascript
// Example: Navigation context with memoized value
const NavigationContext = React.createContext({
  isNavExpanded: true,
  toggle: () => {},
});

const NavigationController = ({ children }) => {
  const [isNavExpanded, setIsNavExpanded] = useState(true);
  
  const toggle = useCallback(() => {
    setIsNavExpanded(prev => !prev);
  }, []); // Stable function reference
  
  const value = useMemo(
    () => ({ isNavExpanded, toggle }), 
    [isNavExpanded, toggle]
  ); // Memoized context value
  
  return (
    <NavigationContext.Provider value={value}>
      {children}
    </NavigationContext.Provider>
  );
};

// Consumer component
const ExpandButton = () => {
  const { isNavExpanded, toggle } = useContext(NavigationContext);
  return (
    <button onClick={toggle}>
      {isNavExpanded ? 'Collapse' : 'Expand'}
    </button>
  );
};
```

## Deprecations

**Avoid keeping custom hooks that trigger re-renders unoptimized**:

Custom hooks abstract stateful logic but internal state triggers re-renders in components using them, regardless of whether the data is used.

When custom hooks contain state, ensure components using them don't re-render unnecessarily by separating state into lower components.

```javascript
// Bad: useModalDialog state causes entire App to re-render
const useModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);
  return { 
    isOpen, 
    open: () => setIsOpen(true), 
    close: () => setIsOpen(false) 
  };
};

const App = () => {
  const { isOpen, open, close } = useModalDialog();
  return (
    <div>
      <Button onClick={open}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={close} /> : null}
      <VerySlowComponent /> {/* Re-renders when modal opens/closes */}
    </div>
  );
};

// Good: Encapsulate hook usage in dedicated component
const ButtonWithModalDialog = () => {
  const { isOpen, open, close } = useModalDialog();
  return (
    <>
      <Button onClick={open}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={close} /> : null}
    </>
  );
};

const ImprovedApp = () => {
  return (
    <div>
      <ButtonWithModalDialog /> {/* Only this component re-renders */}
      <VerySlowComponent /> {/* Unaffected */}
    </div>
  );
};
```

**Avoid using Refs to control UI rendering**:

Ref updates don't trigger re-renders, so data stored in Refs won't update UI even when displayed or passed as props.

Manage UI-affecting data with useState or useReducer. Use Refs only for values that shouldn't trigger re-renders.

```javascript
// Bad: UI doesn't update when ref changes
const BadForm = () => {
  const inputRef = useRef('');
  const numberOfLetters = inputRef.current.length; // Always shows 0
  
  const handleChange = (e) => {
    inputRef.current = e.target.value; // Doesn't trigger re-render
  };
  
  return (
    <>
      <input type="text" onChange={handleChange} />
      <p>Character count: {numberOfLetters}</p> {/* Never updates */}
    </>
  );
};

// Good: State triggers UI updates
const GoodForm = () => {
  const [input, setInput] = useState('');
  const numberOfLetters = input.length;
  
  return (
    <>
      <input 
        type="text" 
        value={input}
        onChange={(e) => setInput(e.target.value)} 
      />
      <p>Character count: {numberOfLetters}</p> {/* Updates correctly */}
    </>
  );
};
```

**Avoid calling debounce/throttle directly in render functions**:

Calling debounce/throttle in render functions recreates them on every re-render, resetting internal timers and caches. They act as simple delay functions instead.

Memoize debounced/throttled functions with useMemo or useCallback. Use the mutable Ref pattern for callbacks needing access to latest state while maintaining stable references.

```javascript
// Bad: Debounce recreated every render
const BadInput = () => {
  const [value, setValue] = useState('');
  
  // This creates a new debounced function every render
  const debouncedLog = debounce(() => {
    console.log('Value:', value);
  }, 500);
  
  const onChange = (e) => {
    setValue(e.target.value);
    debouncedLog(); // Doesn't work as expected
  };
  
  return <input onChange={onChange} value={value} />;
};

// Good: Custom hook with proper memoization
const useDebounce = (callback, delay) => {
  const latestCallback = useRef();
  
  useEffect(() => {
    latestCallback.current = callback;
  }, [callback]);
  
  const debouncedCallback = useMemo(() => {
    const func = (...args) => {
      latestCallback.current?.(...args);
    };
    return debounce(func, delay);
  }, [delay]);
  
  return debouncedCallback;
};

const GoodInput = () => {
  const [value, setValue] = useState('');
  
  const debouncedLog = useDebounce(() => {
    console.log('Debounced value:', value);
  }, 500);
  
  const onChange = (e) => {
    setValue(e.target.value);
    debouncedLog();
  };
  
  return <input onChange={onChange} value={value} />;
};
```