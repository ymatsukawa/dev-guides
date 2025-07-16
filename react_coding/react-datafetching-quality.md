# Data Fetching & Code Quality
This document describes the essence of Data Fetching & Code Quality in React and the recommendations and deprecations that support it.

## Essence of Data Fetching & Code Quality

In one sentence, Data Fetching & Code Quality is about controlling async operations within React lifecycle while maintaining clean, maintainable code structure. When using this, follow these philosophies:

- [ ] Execute data fetching within React lifecycle for proper control
- [ ] Handle race conditions and errors comprehensively
- [ ] Maintain consistent code style and component structure

## Recommendations

**Apply data fetching best practices**:

Execute multiple data requests concurrently using Promise.all. Resolve race conditions by discarding stale results, using cleanup flags, or canceling previous requests with AbortController.

Create comprehensive error handling with ErrorBoundary components at strategic locations. Use try/catch for async code and re-throw errors in state updates to make them catchable by ErrorBoundaries.

```javascript
// Parallel fetching with Promise.all
const useAllData = () => {
  const [data, setData] = useState({ sidebar: null, issue: null, comments: null });
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const [sidebarRes, issueRes, commentsRes] = await Promise.all([
          fetch('/api/sidebar'),
          fetch('/api/issue'),
          fetch('/api/comments')
        ]);
        
        const [sidebar, issue, comments] = await Promise.all([
          sidebarRes.json(),
          issueRes.json(),
          commentsRes.json()
        ]);
        
        setData({ sidebar, issue, comments });
      } catch (error) {
        console.error('Failed to fetch data:', error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, []);
  
  return { ...data, loading };
};

// Race condition handling with AbortController
const PageWithAbortController = ({ id }) => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/data/${id}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => setData(data))
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      });
    
    return () => {
      controller.abort(); // Cancel on cleanup
    };
  }, [id]);
  
  return data ? <div>{data.content}</div> : <div>Loading...</div>;
};
```

**Write clean and maintainable code**:

Apply EditorConfig for consistent coding style across IDEs. Configure ESLint with appropriate style guides like Airbnb's. Set up Git hooks with Husky to run linters pre-commit.

Use JSX actively for UI structure definition. Keep components small with clear responsibilities. Define prop types clearly and use Stateless Functional Components when appropriate.

```javascript
// Clean component with PropTypes
import PropTypes from 'prop-types';

const UserCard = ({ user, onEdit, onDelete }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="actions">
        <button onClick={() => onEdit(user.id)}>Edit</button>
        <button onClick={() => onDelete(user.id)}>Delete</button>
      </div>
    </div>
  );
};

UserCard.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.string.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }).isRequired,
  onEdit: PropTypes.func.isRequired,
  onDelete: PropTypes.func.isRequired
};

// Package.json with linting setup
{
  "scripts": {
    "lint": "eslint --ext .jsx,.js src",
    "lint:fix": "eslint --ext .jsx,.js src --fix"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint"
    }
  }
}
```

## Deprecations

**Avoid fetching data outside React lifecycle**:

Starting async operations outside components or useEffect causes uncontrolled request firing when JavaScript loads. This consumes browser's parallel request limits and blocks critical initial data fetching.

Control data fetching within React lifecycle, primarily useEffect. Only prefetch critical resources at router level or for lazy-loaded components where request priority is clear.

```javascript
// Bad: Uncontrolled fetch on module load
let userData = null;

// This runs immediately when module loads
fetch('/api/user')
  .then(res => res.json())
  .then(data => { userData = data; });

const Profile = () => {
  // userData might not be loaded yet
  return <div>{userData?.name || 'Loading...'}</div>;
};

// Good: Controlled fetch in component lifecycle
const Profile = () => {
  const [userData, setUserData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setUserData(data);
        setLoading(false);
      })
      .catch(error => {
        console.error('Failed to load user:', error);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  return <div>{userData?.name}</div>;
};
```