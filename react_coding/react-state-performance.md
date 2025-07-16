# State Management & Performance
This document describes the essence of State Management & Performance in React and the recommendations and deprecations that support it.

## Essence of State Management & Performance

In one sentence, State Management & Performance is about minimizing unnecessary re-renders by strategically placing state and maintaining data immutability. When using this, follow these philosophies:

- [ ] Place state as close as possible to where it's needed
- [ ] Maintain single sources of truth for data
- [ ] Always create new instances when updating state

## Recommendations

**Move state down to lower components**:

State updates trigger re-renders for the component holding the state and all nested components within it. Moving state to the lowest common parent minimizes re-render scope.

Extract state and dependent components to the lowest possible common parent component. This simple composition technique is highly effective for fixing performance issues in large applications.

```javascript
// Bad: App component manages modal state, causing unnecessary re-renders
const App = () => {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}
      <VerySlowComponent /> {/* Re-renders when isOpen changes */}
      <BunchOfStuff />
    </div>
  );
};

// Good: State moved to dedicated component
const ButtonWithModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <>
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}
    </>
  );
};

const App = () => {
  return (
    <div>
      <ButtonWithModalDialog /> {/* Only this re-renders */}
      <VerySlowComponent /> {/* Unaffected by modal state */}
      <BunchOfStuff />
    </div>
  );
};
```

**Pass elements as props**:

When parent components re-render due to state updates, elements passed as props won't re-render unless their reference changes. This prevents unnecessary re-renders and improves performance.

Use the `children` prop for content independent of component logic. Consider elements as props for layout components that don't depend on internal rendering content.

```javascript
// Bad: Children defined inside scrollable component re-render on scroll
const MainScrollableArea = () => {
  const [position, setPosition] = useState(300);
  const onScroll = (e) => {
    setPosition(getPosition(e.target.scrollTop));
  };
  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      <VerySlowComponent /> {/* Re-renders on every scroll */}
      <BunchOfStuff />
    </div>
  );
};

// Good: Children passed as props remain stable
const ScrollableWithMovingBlock = ({ children }) => {
  const [position, setPosition] = useState(300);
  const onScroll = (e) => {
    setPosition(getPosition(e.target.scrollTop));
  };
  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {children} {/* Not re-rendered on scroll */}
    </div>
  );
};

const App = () => {
  return (
    <ScrollableWithMovingBlock>
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </ScrollableWithMovingBlock>
  );
};
```

## Deprecations

**Avoid initializing state from props**:

This creates duplicate "sources of truth" between props and state, making it unclear which value is authoritative. State won't update when props change after initialization.

For one-time initialization, use clear naming like `initialCount`. Otherwise, use props directly and avoid state duplication.

```javascript
// Bad: Confusing duplication of truth
const Counter = ({ count }) => {
  const [internalCount, setInternalCount] = useState(count);
  // If parent changes count prop, internalCount doesn't update
  
  return (
    <div>
      <p>Count: {internalCount}</p>
      <button onClick={() => setInternalCount(internalCount + 1)}>
        Increment
      </button>
    </div>
  );
};

// Good: Clear one-time initialization
const Counter = ({ initialCount }) => {
  const [count, setCount] = useState(initialCount);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
};

// Good: Use props directly when appropriate
const Display = ({ count }) => {
  return <p>Count: {count}</p>; // No state needed
};
```

**Avoid mutating state directly**:

Direct mutations to state objects don't trigger re-renders, causing UI inconsistencies. PureComponent's shallow comparison won't detect mutations, losing optimization benefits.

Always use setState methods. Create new instances when updating reference types using concat() or spread syntax to maintain immutability.

```javascript
// Bad: Direct state mutation
class BadList extends React.Component {
  state = { items: [1, 2, 3] };
  
  handleAdd = () => {
    this.state.items.push(4); // Direct mutation
    this.setState({ items: this.state.items }); // Same reference
  };
  
  render() {
    return (
      <div>
        <button onClick={this.handleAdd}>Add Item</button>
        <ul>
          {this.state.items.map(item => <li key={item}>{item}</li>)}
        </ul>
      </div>
    );
  }
}

// Good: Create new array instance
const GoodList = () => {
  const [items, setItems] = useState([1, 2, 3]);
  
  const handleAdd = () => {
    setItems([...items, items.length + 1]); // New array
    // Or: setItems(items.concat(items.length + 1));
  };
  
  return (
    <div>
      <button onClick={handleAdd}>Add Item</button>
      <ul>
        {items.map(item => <li key={item}>{item}</li>)}
      </ul>
    </div>
  );
};
```