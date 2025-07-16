# Component Architecture
This document describes the essence of Component Architecture in React and the recommendations and deprecations that support it.

## Essence of Component Architecture

In one sentence, Component Architecture is about creating reusable, predictable components through proper structure, unique keys, and flexible composition patterns. When using this, follow these philosophies:

- [ ] Define components at module level for proper lifecycle management
- [ ] Use unique, stable keys for list items and state control
- [ ] Leverage composition patterns for flexible component reuse

## Recommendations

**Use render props for advanced configuration and state sharing**:

Render props provide a flexible way to share stateful logic between components, allowing parent components to pass state to dynamically rendered children.

Use render props when icon props need dynamic adjustment based on button state, or when sharing DOM-dependent logic like scroll detection.

```javascript
// Example: Button with dynamic icon based on hover state
const Button = ({ appearance, size, renderIcon }) => {
  const [isHovered, setIsHovered] = useState(false);
  const iconParams = {
    size: size === 'large' ? 'large' : 'medium',
    color: appearance === 'primary' ? 'white' : 'black',
    isHovered, // Pass state to icon
  };
  return (
    <button 
      onMouseOver={() => setIsHovered(true)} 
      onMouseOut={() => setIsHovered(false)}
    >
      Submit {renderIcon(iconParams)}
    </button>
  );
};

// Usage
<Button
  appearance="primary"
  renderIcon={(props) => <HomeIcon {...props} />}
/>

// Scroll detection example
const ScrollDetector = ({ children }) => {
  const [scroll, setScroll] = useState(0);
  return (
    <div onScroll={(e) => setScroll(e.currentTarget.scrollTop)}>
      {children(scroll)} {/* Pass scroll value to children */}
    </div>
  );
};

const Layout = () => {
  return (
    <ScrollDetector>
      {(scroll) => {
        return <>{scroll > 30 ? <SomeBlock /> : null}</>;
      }}
    </ScrollDetector>
  );
};
```

**Use key attribute properly for reuse control and state reset**:

The key attribute helps React identify elements in lists and decide whether to reuse or remount instances between renders, preventing unexpected state retention.

Assign unique, stable keys to dynamic list items. Use the "state reset" technique by changing key values to force remounting when intentionally resetting component state.

```javascript
// State reset example
const Form = () => {
  const [isCompany, setIsCompany] = useState(false);
  return (
    <>
      <Checkbox onChange={() => setIsCompany(!isCompany)} />
      {isCompany ? (
        <Input 
          key="company-tax-id" // Key forces remount on change
          placeholder="Enter company Tax ID" 
        />
      ) : (
        <Input 
          key="person-tax-id" // Different key = new instance
          placeholder="Enter personal Tax ID" 
        />
      )}
    </>
  );
};

// Dynamic list with stable keys
const TodoList = ({ todos }) => {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} // Stable, unique identifier
          todo={todo}
        />
      ))}
    </ul>
  );
};
```

## Deprecations

**Avoid defining components inside other components**:

Defining components inside others causes child components to be recreated on every parent re-render. React treats them as new components, unmounting previous instances.

Define all components at the top level or module level to ensure React can properly track and reuse component instances.

```javascript
// Bad: Input loses state on every Component re-render
const Component = () => {
  const Input = () => { // Recreated on every render
    const [text, setText] = useState('');
    return <input value={text} onChange={e => setText(e.target.value)} />;
  };
  
  return (
    <div>
      <p>Type something:</p>
      <Input /> {/* Loses state when Component re-renders */}
    </div>
  );
};

// Good: Input maintains state across re-renders
const StandaloneInput = () => {
  const [text, setText] = useState('');
  return <input value={text} onChange={e => setText(e.target.value)} />;
};

const ImprovedComponent = () => {
  return (
    <div>
      <p>Type something:</p>
      <StandaloneInput /> {/* Maintains state properly */}
    </div>
  );
};
```

**Avoid using array indices as keys**:

Using map indices as keys prevents React's reconciliation algorithm from efficiently tracking DOM changes when list items are added, removed, or reordered.

Use unique, stable IDs that identify each element. If data lacks unique IDs, consider generating temporary ones or restructuring data.

```javascript
// Bad: Index as key causes issues with reordering
const BadTodoList = ({ todos }) => {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}> {/* Problems when todos are reordered */}
          <input type="checkbox" />
          {todo.text}
        </li>
      ))}
    </ul>
  );
};

// Good: Unique, stable IDs as keys
const GoodTodoList = ({ todos }) => {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}> {/* Stable across reorders */}
          <input type="checkbox" checked={todo.done} />
          {todo.text}
        </li>
      ))}
    </ul>
  );
};

// If no ID exists, generate one
const TodosWithGeneratedIds = ({ items }) => {
  const todosWithIds = useMemo(() => 
    items.map((item, index) => ({
      ...item,
      id: `todo-${item.text}-${index}` // Generate stable ID
    })),
    [items]
  );
  
  return <GoodTodoList todos={todosWithIds} />;
};
```