# Memoization & Optimization
This document describes the essence of Memoization & Optimization in React and the recommendations and deprecations that support it.

## Essence of Memoization & Optimization

In one sentence, Memoization & Optimization is about preventing unnecessary re-renders and computations through strategic caching of values and components. When using this, follow these philosophies:

- [ ] Memoize only when performance profiling indicates a need
- [ ] Ensure all props to memoized components are stable
- [ ] Use composition patterns before reaching for memoization

## Recommendations

**Ensure reference stability with useMemo and useCallback**:

Objects, arrays, and functions are compared by reference in JavaScript. Unstable references in hook dependencies or props to memoized components cause unnecessary re-renders.

Memoize non-primitive values used as dependencies in hooks or props to React.memo components. Always memoize functions, objects, and JSX elements including children.

```javascript
// Bad: New object and function created every render
const Component = () => {
  const data = { id: '1' }; // New object reference
  const onChange = () => { /* ... */ }; // New function reference
  
  return <ChildMemo data={data} onChange={onChange} />;
};

// Good: Memoized references remain stable
const Component = () => {
  const data = useMemo(() => ({ id: '1' }), []); // Stable object
  const onChange = useCallback(() => { /* ... */ }, []); // Stable function
  
  return <ChildMemo data={data} onChange={onChange} />;
};

// Memoizing children example
const ComponentWithMemoizedChildren = () => {
  const content = useMemo(() => <div>Some text here</div>, []);
  return <ChildMemo>{content}</ChildMemo>;
};
```

**Apply React.memo strategically**:

React.memo memoizes components and skips re-rendering when props haven't changed during parent re-renders. This is powerful as a last resort for performance optimization.

Only apply to components where all props are guaranteed to be memoized. First consider composition-based optimizations before using React.memo.

```javascript
// Simple example of React.memo usage
const ExpensiveList = React.memo(({ items, onItemClick }) => {
  console.log('ExpensiveList rendered');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

// Parent component must memoize all props
const Parent = () => {
  const [count, setCount] = useState(0);
  const items = useMemo(() => [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
  ], []);
  
  const handleItemClick = useCallback((id) => {
    console.log('Clicked:', id);
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveList items={items} onItemClick={handleItemClick} />
    </div>
  );
};
```

## Deprecations

**Avoid unmemoized props to React.memo components**:

React.memo performs reference comparison on props. Even one unmemoized prop invalidates the entire memoization benefit.

Ensure all props to React.memo components have stable references. Avoid spreading props from other components. Memoize children as they're treated like any other prop.

```javascript
// Bad: Unmemoized props defeat React.memo
const Child = React.memo(({ data, onChange }) => {
  console.log('Child rendered');
  return <div onClick={onChange}>{JSON.stringify(data)}</div>;
});

const BadParent = () => {
  const [count, setCount] = useState(0);
  
  // These create new references every render
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child 
        data={{ value: 'test' }} // New object every render
        onChange={() => console.log('clicked')} // New function every render
      />
    </div>
  );
};

// Good: All props are memoized
const GoodParent = () => {
  const [count, setCount] = useState(0);
  
  const data = useMemo(() => ({ value: 'test' }), []);
  const onChange = useCallback(() => console.log('clicked'), []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child data={data} onChange={onChange} />
    </div>
  );
};
```

**Avoid overusing useMemo for vague "expensive calculations"**:

"Expensive calculation" is subjective and only measurable through profiling. useMemo adds caching overhead that can slow initial renders and increase code complexity.

Apply useMemo only after measuring actual performance bottlenecks. Prioritize composition-based optimizations over memoization.

```javascript
// Bad: Unnecessary memoization for simple calculation
const Component = ({ items }) => {
  // This is not expensive enough to warrant memoization
  const totalCount = useMemo(() => {
    return items.length;
  }, [items]);
  
  return <div>Total: {totalCount}</div>;
};

// Good: Direct calculation for simple operations
const ImprovedComponent = ({ items }) => {
  const totalCount = items.length; // Simple, no memoization needed
  
  return <div>Total: {totalCount}</div>;
};

// Good: Memoize only truly expensive operations
const ExpensiveComponent = ({ data }) => {
  const processedData = useMemo(() => {
    // Only if profiling shows this is a bottleneck
    return data.map(item => 
      complexTransformation(item)
    ).filter(item => 
      expensiveFilter(item)
    );
  }, [data]);
  
  return <DataVisualization data={processedData} />;
};
```