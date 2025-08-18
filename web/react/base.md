# Complete React Cheatsheet

## Table of Contents
1. [Setup & Installation](#setup--installation)
2. [Components](#components)
3. [JSX](#jsx)
4. [Props](#props)
5. [State & useState](#state--usestate)
6. [Event Handling](#event-handling)
7. [Conditional Rendering](#conditional-rendering)
8. [Lists & Keys](#lists--keys)
9. [Forms](#forms)
10. [Hooks](#hooks)
11. [Context API](#context-api)
12. [Error Boundaries](#error-boundaries)
13. [Performance Optimization](#performance-optimization)
14. [Testing](#testing)
15. [Common Patterns](#common-patterns)
16. [Best Practices](#best-practices)

---

## Setup & Installation

### Create React App
```bash
# Create new app
npx create-react-app my-app
cd my-app
npm start

# With TypeScript
npx create-react-app my-app --template typescript
```

### Vite (Faster alternative)
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

### Essential Dependencies
```bash
# Routing
npm install react-router-dom

# State Management
npm install redux @reduxjs/toolkit react-redux
npm install zustand

# HTTP Requests
npm install axios
npm install @tanstack/react-query

# UI Libraries
npm install @mui/material
npm install tailwindcss
```

---

## Components

### Functional Components
```jsx
// Basic functional component
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// Arrow function component
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};

// With props
const Greeting = ({ name, age = 18 }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
    </div>
  );
};

// Component with children
const Container = ({ children, className }) => {
  return <div className={className}>{children}</div>;
};
```

### Component Export/Import
```jsx
// Named export
export const Button = () => <button>Click me</button>;

// Default export
const App = () => <div>My App</div>;
export default App;

// Import
import App from './App';
import { Button } from './Button';
```

---

## JSX

### JSX Syntax Rules
```jsx
// Must return single element
const Component = () => {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
};

// React Fragments
const Component = () => {
  return (
    <React.Fragment>
      <h1>Title</h1>
      <p>Content</p>
    </React.Fragment>
  );
};

// Short syntax
const Component = () => {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
};
```

### JSX Expressions
```jsx
const Component = () => {
  const name = "John";
  const user = { firstName: "Jane", lastName: "Doe" };
  const items = ["apple", "banana", "orange"];

  return (
    <div>
      {/* Variables */}
      <h1>Hello, {name}!</h1>
      
      {/* Object properties */}
      <p>{user.firstName} {user.lastName}</p>
      
      {/* Function calls */}
      <p>{new Date().toLocaleDateString()}</p>
      
      {/* Conditional expressions */}
      <p>{name ? `Hello, ${name}` : 'Hello, Guest'}</p>
      
      {/* Array methods */}
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
};
```

### JSX Attributes
```jsx
const Component = () => {
  const isActive = true;
  const customClass = "highlight";

  return (
    <div>
      {/* String attributes */}
      <img src="image.jpg" alt="Description" />
      
      {/* Dynamic attributes */}
      <div className={customClass} />
      <div className={isActive ? 'active' : 'inactive'} />
      
      {/* Template literals */}
      <div className={`btn ${isActive ? 'active' : ''}`} />
      
      {/* Boolean attributes */}
      <input disabled={true} />
      <input disabled /> {/* Same as above */}
      
      {/* Event handlers */}
      <button onClick={() => console.log('clicked')}>
        Click me
      </button>
    </div>
  );
};
```

---

## Props

### Basic Props
```jsx
// Parent component
const App = () => {
  return (
    <div>
      <UserCard 
        name="John Doe" 
        age={25} 
        isActive={true}
        hobbies={['reading', 'coding']}
      />
    </div>
  );
};

// Child component
const UserCard = ({ name, age, isActive, hobbies }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {isActive ? 'Active' : 'Inactive'}</p>
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
};
```

### Default Props & Destructuring
```jsx
// Default parameters
const Button = ({ 
  text = 'Click me', 
  variant = 'primary', 
  onClick = () => {} 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {text}
    </button>
  );
};

// Props object
const Button = (props) => {
  return <button onClick={props.onClick}>{props.text}</button>;
};

// Rest props
const Input = ({ label, ...inputProps }) => {
  return (
    <div>
      <label>{label}</label>
      <input {...inputProps} />
    </div>
  );
};

// Usage
<Input 
  label="Email" 
  type="email" 
  placeholder="Enter email"
  required 
/>
```

### Children Props
```jsx
// Basic children
const Card = ({ children, title }) => {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
};

// Usage
<Card title="My Card">
  <p>This is the content</p>
  <button>Action</button>
</Card>

// Function as children (render props)
const DataFetcher = ({ children }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchData().then(result => {
      setData(result);
      setLoading(false);
    });
  }, []);

  return children({ data, loading });
};

// Usage
<DataFetcher>
  {({ data, loading }) => (
    loading ? <div>Loading...</div> : <div>{data}</div>
  )}
</DataFetcher>
```

---

## State & useState

### Basic useState
```jsx
import { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
};
```

### Complex State
```jsx
// Object state
const UserForm = () => {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  return (
    <form>
      <input 
        value={user.name}
        onChange={(e) => updateUser('name', e.target.value)}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={(e) => updateUser('email', e.target.value)}
        placeholder="Email"
      />
      <input 
        type="number"
        value={user.age}
        onChange={(e) => updateUser('age', parseInt(e.target.value))}
        placeholder="Age"
      />
    </form>
  );
};

// Array state
const TodoList = () => {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos(prevTodos => [...prevTodos, {
        id: Date.now(),
        text: inputValue,
        completed: false
      }]);
      setInputValue('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(prevTodos => 
      prevTodos.map(todo => 
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
  };

  return (
    <div>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add Todo</button>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span 
              style={{ 
                textDecoration: todo.completed ? 'line-through' : 'none' 
              }}
              onClick={() => toggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### Functional Updates
```jsx
const Component = () => {
  const [count, setCount] = useState(0);

  // Use functional update when new state depends on previous state
  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Multiple state updates
  const incrementTwice = () => {
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={incrementTwice}>+2</button>
    </div>
  );
};
```

---

## Event Handling

### Common Events
```jsx
const EventExamples = () => {
  const [inputValue, setInputValue] = useState('');

  const handleClick = (e) => {
    console.log('Button clicked:', e.target);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted with:', inputValue);
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  };

  const handleMouseEnter = () => {
    console.log('Mouse entered');
  };

  const handleFocus = (e) => {
    e.target.style.backgroundColor = '#f0f0f0';
  };

  const handleBlur = (e) => {
    e.target.style.backgroundColor = 'white';
  };

  return (
    <div>
      <button onClick={handleClick}>Click me</button>
      
      <form onSubmit={handleSubmit}>
        <input 
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={handleKeyPress}
          onFocus={handleFocus}
          onBlur={handleBlur}
          placeholder="Type something..."
        />
        <button type="submit">Submit</button>
      </form>
      
      <div 
        onMouseEnter={handleMouseEnter}
        onMouseLeave={() => console.log('Mouse left')}
        style={{ padding: '20px', border: '1px solid #ccc' }}
      >
        Hover over me
      </div>
    </div>
  );
};
```

### Event Parameters & Binding
```jsx
const ButtonList = () => {
  const items = ['Item 1', 'Item 2', 'Item 3'];

  const handleItemClick = (item, index) => {
    console.log(`Clicked ${item} at index ${index}`);
  };

  return (
    <div>
      {items.map((item, index) => (
        <button
          key={index}
          onClick={() => handleItemClick(item, index)}
        >
          {item}
        </button>
      ))}
    </div>
  );
};
```

---

## Conditional Rendering

### If/Else Patterns
```jsx
const ConditionalExample = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // Ternary operator
  return (
    <div>
      {user ? (
        <div>Welcome, {user.name}!</div>
      ) : (
        <div>Please log in</div>
      )}
    </div>
  );
};

// Logical AND operator
const Notification = ({ message, type }) => {
  return (
    <div>
      {message && (
        <div className={`alert alert-${type}`}>
          {message}
        </div>
      )}
    </div>
  );
};

// Multiple conditions
const UserProfile = ({ user, loading, error }) => {
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

// Switch-like pattern
const StatusMessage = ({ status }) => {
  const renderStatus = () => {
    switch (status) {
      case 'loading':
        return <div>Loading...</div>;
      case 'success':
        return <div>Success!</div>;
      case 'error':
        return <div>Error occurred</div>;
      default:
        return <div>Unknown status</div>;
    }
  };

  return <div>{renderStatus()}</div>;
};
```

---

## Lists & Keys

### Rendering Lists
```jsx
const ItemList = () => {
  const fruits = ['apple', 'banana', 'orange', 'grape'];
  
  const users = [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' },
    { id: 3, name: 'Bob', email: 'bob@example.com' }
  ];

  return (
    <div>
      {/* Simple array */}
      <ul>
        {fruits.map((fruit, index) => (
          <li key={index}>{fruit}</li>
        ))}
      </ul>

      {/* Array of objects */}
      <div>
        {users.map(user => (
          <div key={user.id}>
            <h3>{user.name}</h3>
            <p>{user.email}</p>
          </div>
        ))}
      </div>

      {/* With filtering */}
      <div>
        {users
          .filter(user => user.name.startsWith('J'))
          .map(user => (
            <div key={user.id}>{user.name}</div>
          ))
        }
      </div>
    </div>
  );
};
```

### Key Prop Best Practices
```jsx
const TodoList = ({ todos }) => {
  return (
    <ul>
      {todos.map(todo => (
        // ✅ Good: Use unique, stable ID
        <li key={todo.id}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
};

// ❌ Avoid: Using array index when list can change
const BadExample = ({ items }) => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
};

// ✅ Better: Create stable keys
const GoodExample = ({ items }) => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={`item-${item}-${index}`}>{item}</li>
      ))}
    </ul>
  );
};
```

---

## Forms

### Controlled Components
```jsx
const ContactForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
    subscribe: false,
    country: 'us'
  });

  const handleInputChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name:</label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleInputChange}
          required
        />
      </div>

      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          required
        />
      </div>

      <div>
        <label htmlFor="message">Message:</label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleInputChange}
          rows={4}
        />
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="subscribe"
            checked={formData.subscribe}
            onChange={handleInputChange}
          />
          Subscribe to newsletter
        </label>
      </div>

      <div>
        <label htmlFor="country">Country:</label>
        <select
          id="country"
          name="country"
          value={formData.country}
          onChange={handleInputChange}
        >
          <option value="us">United States</option>
          <option value="ca">Canada</option>
          <option value="uk">United Kingdom</option>
        </select>
      </div>

      <button type="submit">Submit</button>
    </form>
  );
};
```

### Form Validation
```jsx
const ValidatedForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    confirmPassword: ''
  });
  const [errors, setErrors] = useState({});

  const validateForm = () => {
    const newErrors = {};

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }

    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }

    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validateForm();
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Form is valid:', formData);
      setErrors({});
    } else {
      setErrors(newErrors);
    }
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          placeholder="Email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <input
          type="password"
          name="password"
          placeholder="Password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          placeholder="Confirm Password"
          value={formData.confirmPassword}
          onChange={handleChange}
        />
        {errors.confirmPassword && (
          <span className="error">{errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit">Register</button>
    </form>
  );
};
```

---

## Hooks

### useEffect
```jsx
import { useState, useEffect } from 'react';

// Basic useEffect
const DataFetcher = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Effect runs after component mounts
    fetch('/api/data')
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, []); // Empty dependency array = run once on mount

  return loading ? <div>Loading...</div> : <div>{JSON.stringify(data)}</div>;
};

// Effect with cleanup
const Timer = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);

    // Cleanup function
    return () => clearInterval(interval);
  }, []); // Run once on mount

  return <div>Timer: {count}</div>;
};

// Effect with dependencies
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);

  useEffect(() => {
    if (userId) {
      fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(setUser);
    }
  }, [userId]); // Re-run when userId changes

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
};

// Multiple effects
const Component = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Effect for document title
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  // Effect for localStorage
  useEffect(() => {
    const savedName = localStorage.getItem('name');
    if (savedName) {
      setName(savedName);
    }
  }, []);

  useEffect(() => {
    if (name) {
      localStorage.setItem('name', name);
    }
  }, [name]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Enter name"
      />
    </div>
  );
};
```

### useReducer
```jsx
import { useReducer } from 'react';

// Reducer function
const counterReducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
};

// Complex state with useReducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case 'add':
      return [
        ...state,
        {
          id: Date.now(),
          text: action.payload,
          completed: false
        }
      ];
    case 'toggle':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'delete':
      return state.filter(todo => todo.id !== action.payload);
    case 'clear_completed':
      return state.filter(todo => !todo.completed);
    default:
      return state;
  }
};

const TodoApp = () => {
  const [todos, dispatch] = useReducer(todoReducer, []);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      dispatch({ type: 'add', payload: inputValue });
      setInputValue('');
    }
  };

  return (
    <div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add Todo</button>
      <button onClick={() => dispatch({ type: 'clear_completed' })}>
        Clear Completed
      </button>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}
              onClick={() => dispatch({ type: 'toggle', payload: todo.id })}
            >
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'delete', payload: todo.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### Custom Hooks
```jsx
// useLocalStorage hook
const useLocalStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.log(error);
    }
  };

  return [storedValue, setValue];
};

// Usage
const Settings = () => {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
};

// useFetch hook
const useFetch = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (url) {
      fetchData();
    }
  }, [url]);

  return { data, loading, error };
};

// Usage
const UserList = () => {
  const { data: users, loading, error } = useFetch('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

// useDebounce hook
const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
};

// Usage
const SearchInput = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);

  useEffect(() => {
    if (debouncedSearchTerm) {
      // Perform search
      console.log('Searching for:', debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      type="text"
      placeholder="Search..."
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
    />
  );
};
```

### useRef
```jsx
import { useRef, useEffect } from 'react';

// Focus input on mount
const AutoFocusInput = () => {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} placeholder="This will be focused" />;
};

// Accessing DOM elements
const ScrollToTop = () => {
  const topRef = useRef(null);

  const scrollToTop = () => {
    topRef.current.scrollIntoView({ behavior: 'smooth' });
  };

  return (
    <div>
      <div ref={topRef}>Top of the page</div>
      {/* ... lots of content ... */}
      <button onClick={scrollToTop}>Scroll to Top</button>
    </div>
  );
};

// Storing mutable values
const Timer = () => {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  useEffect(() => {
    return () => stopTimer(); // Cleanup on unmount
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
};

// Previous value tracking
const usePrevious = (value) => {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
};

// Usage
const Counter = () => {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
};
```

### useMemo & useCallback
```jsx
import { useMemo, useCallback, useState } from 'react';

// useMemo for expensive calculations
const ExpensiveComponent = ({ items, searchTerm }) => {
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => 
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);

  const expensiveValue = useMemo(() => {
    console.log('Computing expensive value...');
    return filteredItems.reduce((sum, item) => sum + item.value, 0);
  }, [filteredItems]);

  return (
    <div>
      <p>Expensive value: {expensiveValue}</p>
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
};

// useCallback for event handlers
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  const [todos, setTodos] = useState([]);

  const handleIncrement = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);

  const addTodo = useCallback((text) => {
    setTodos(prevTodos => [...prevTodos, {
      id: Date.now(),
      text,
      completed: false
    }]);
  }, []);

  const toggleTodo = useCallback((id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);

  return (
    <div>
      <Counter count={count} onIncrement={handleIncrement} />
      <TodoList 
        todos={todos} 
        onAdd={addTodo} 
        onToggle={toggleTodo} 
      />
    </div>
  );
};

// Child components that benefit from useCallback
const Counter = React.memo(({ count, onIncrement }) => {
  console.log('Counter rendered');
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={onIncrement}>+</button>
    </div>
  );
});

const TodoList = React.memo(({ todos, onAdd, onToggle }) => {
  console.log('TodoList rendered');
  const [inputValue, setInputValue] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      onAdd(inputValue);
      setInputValue('');
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
        />
        <button type="submit">Add</button>
      </form>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span
              onClick={() => onToggle(todo.id)}
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}
            >
              {todo.text}
            </span>
          </li>
        ))}
      </ul>
    </div>
  );
});
```

---

## Context API

### Creating and Using Context
```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();
const UserContext = createContext();

// Theme Provider
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    toggleTheme
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

// User Provider
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const login = (userData) => {
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  const value = {
    user,
    login,
    logout,
    isAuthenticated: !!user
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};

// Custom hooks for context
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
};

// Components using context
const Header = () => {
  const { theme, toggleTheme } = useTheme();
  const { user, logout, isAuthenticated } = useUser();

  return (
    <header className={`header ${theme}`}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} theme
      </button>
      
      {isAuthenticated ? (
        <div>
          <span>Welcome, {user.name}!</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <button>Login</button>
      )}
    </header>
  );
};

const MainContent = () => {
  const { theme } = useTheme();
  
  return (
    <main className={`main ${theme}`}>
      <h2>Main Content</h2>
      <p>This content respects the theme context.</p>
    </main>
  );
};

// App with providers
const App = () => {
  return (
    <ThemeProvider>
      <UserProvider>
        <div className="app">
          <Header />
          <MainContent />
        </div>
      </UserProvider>
    </ThemeProvider>
  );
};
```

### Advanced Context Patterns
```jsx
// Context with reducer
const AppStateContext = createContext();
const AppDispatchContext = createContext();

const appReducer = (state, action) => {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    case 'SET_DATA':
      return { ...state, data: action.payload, loading: false, error: null };
    default:
      return state;
  }
};

const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, {
    data: null,
    loading: false,
    error: null
  });

  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  );
};

// Custom hooks for state and dispatch
const useAppState = () => {
  const context = useContext(AppStateContext);
  if (!context) {
    throw new Error('useAppState must be used within AppProvider');
  }
  return context;
};

const useAppDispatch = () => {
  const context = useContext(AppDispatchContext);
  if (!context) {
    throw new Error('useAppDispatch must be used within AppProvider');
  }
  return context;
};

// Component using context with actions
const DataComponent = () => {
  const { data, loading, error } = useAppState();
  const dispatch = useAppDispatch();

  const fetchData = async () => {
    dispatch({ type: 'SET_LOADING', payload: true });
    try {
      const response = await fetch('/api/data');
      const result = await response.json();
      dispatch({ type: 'SET_DATA', payload: result });
    } catch (err) {
      dispatch({ type: 'SET_ERROR', payload: err.message });
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <button onClick={fetchData}>Fetch Data</button>
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
};
```

---

## Error Boundaries

```jsx
import { Component } from 'react';

// Class component error boundary
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log the error to an error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
    this.setState({
      error,
      errorInfo
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          {this.props.fallback || (
            <details style={{ whiteSpace: 'pre-wrap' }}>
              <summary>Error details</summary>
              {this.state.error && this.state.error.toString()}
              <br />
              {this.state.errorInfo.componentStack}
            </details>
          )}
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Hook-based error boundary (requires external library like react-error-boundary)
import { ErrorBoundary } from 'react-error-boundary';

const ErrorFallback = ({ error, resetErrorBoundary }) => {
  return (
    <div role="alert" className="error-fallback">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
};

// Usage
const App = () => {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, errorInfo) => {
        console.error('Error logged:', error, errorInfo);
      }}
    >
      <Header />
      <MainContent />
      <Footer />
    </ErrorBoundary>
  );
};

// Component that might throw an error
const BuggyComponent = () => {
  const [throwError, setThrowError] = useState(false);

  if (throwError) {
    throw new Error('I crashed!');
  }

  return (
    <div>
      <button onClick={() => setThrowError(true)}>
        Throw Error
      </button>
    </div>
  );
};
```

---

## Performance Optimization

### React.memo
```jsx
// Basic memoization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  console.log('ExpensiveComponent rendered');
  
  return (
    <div>
      <h3>{data.title}</h3>
      <button onClick={onUpdate}>Update</button>
    </div>
  );
});

// Custom comparison function
const UserCard = React.memo(({ user, theme }) => {
  return (
    <div className={`card ${theme}`}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  // Return true if props are equal (skip re-render)
  return (
    prevProps.user.id === nextProps.user.id &&
    prevProps.user.name === nextProps.user.name &&
    prevProps.theme === nextProps.theme
  );
});

// Memoizing with complex objects
const ProductList = React.memo(({ products, filters }) => {
  const filteredProducts = useMemo(() => {
    return products.filter(product => {
      return filters.category === 'all' || product.category === filters.category;
    });
  }, [products, filters]);

  return (
    <div>
      {filteredProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
});
```

### Code Splitting & Lazy Loading
```jsx
import { lazy, Suspense } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Profile = lazy(() => import('./Profile'));

// Component with lazy loading
const App = () => {
  const [currentView, setCurrentView] = useState('dashboard');

  const renderView = () => {
    switch (currentView) {
      case 'dashboard':
        return <Dashboard />;
      case 'settings':
        return <Settings />;
      case 'profile':
        return <Profile />;
      default:
        return <Dashboard />;
    }
  };

  return (
    <div>
      <nav>
        <button onClick={() => setCurrentView('dashboard')}>Dashboard</button>
        <button onClick={() => setCurrentView('settings')}>Settings</button>
        <button onClick={() => setCurrentView('profile')}>Profile</button>
      </nav>

      <Suspense fallback={<div>Loading...</div>}>
        {renderView()}
      </Suspense>
    </div>
  );
};

// Route-based code splitting (with React Router)
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

const AppWithRouting = () => {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
};
```

### Virtual Scrolling Pattern
```jsx
const VirtualizedList = ({ items, itemHeight = 50 }) => {
  const [scrollTop, setScrollTop] = useState(0);
  const containerHeight = 400;
  const visibleCount = Math.ceil(containerHeight / itemHeight);
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleCount + 1, items.length);
  const visibleItems = items.slice(startIndex, endIndex);

  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };

  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems.map((item, index) => (
          <div
            key={startIndex + index}
            style={{
              position: 'absolute',
              top: (startIndex + index) * itemHeight,
              height: itemHeight,
              width: '100%',
              display: 'flex',
              alignItems: 'center',
              padding: '0 16px',
              borderBottom: '1px solid #eee'
            }}
          >
            {item.name}
          </div>
        ))}
      </div>
    </div>
  );
};
```

---

## Testing

### Testing with Jest & React Testing Library
```jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';

// Component to test
const Counter = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue);

  return (
    <div>
      <span data-testid="count">Count: {count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
};

// Tests
describe('Counter', () => {
  test('renders initial count', () => {
    render(<Counter initialValue={5} />);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 5');
  });

  test('increments count when + button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    const incrementButton = screen.getByText('+');
    await user.click(incrementButton);
    
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 1');
  });

  test('decrements count when - button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={5} />);
    
    const decrementButton = screen.getByText('-');
    await user.click(decrementButton);
    
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 4');
  });

  test('resets count to 0 when reset button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialValue={10} />);
    
    const resetButton = screen.getByText('Reset');
    await user.click(resetButton);
    
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 0');
  });
});

// Testing forms
const ContactForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  const handleChange = (e) => {
    setFormData(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        placeholder="Name"
        value={formData.name}
        onChange={handleChange}
      />
      <input
        name="email"
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={handleChange}
      />
      <textarea
        name="message"
        placeholder="Message"
        value={formData.message}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// Form tests
describe('ContactForm', () => {
  test('calls onSubmit with form data when submitted', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<ContactForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByPlaceholderText('Name'), 'John Doe');
    await user.type(screen.getByPlaceholderText('Email'), 'john@example.com');
    await user.type(screen.getByPlaceholderText('Message'), 'Hello world');
    
    await user.click(screen.getByText('Submit'));
    
    expect(handleSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      message: 'Hello world'
    });
  });
});

// Testing async operations
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

// Async tests with mocking
describe('UserProfile', () => {
  beforeEach(() => {
    fetch.resetMocks();
  });

  test('displays loading state initially', () => {
    fetch.mockResponseOnce(JSON.stringify({ name: 'John', email: 'john@example.com' }));
    render(<UserProfile userId={1} />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  test('displays user data after successful fetch', async () => {
    fetch.mockResponseOnce(JSON.stringify({ 
      name: 'John Doe', 
      email: 'john@example.com' 
    }));

    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });
  });

  test('displays error message on fetch failure', async () => {
    fetch.mockRejectOnce(new Error('API Error'));
    
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
      expect(screen.getByText('Error: API Error')).toBeInTheDocument();
    });
  });
});
```

---

## Common Patterns

### Higher-Order Components (HOCs)
```jsx
// Basic HOC
const withLoading = (WrappedComponent) => {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <WrappedComponent {...props} />;
  };
};

// Usage
const UserList = ({ users }) => (
  <ul>
    {users.map(user => <li key={user.id}>{user.name}</li>)}
  </ul>
);

const UserListWithLoading = withLoading(UserList);

// HOC with authentication
const withAuth = (WrappedComponent) => {
  return function WithAuthComponent(props) {
    const { user, isAuthenticated } = useUser();
    
    if (!isAuthenticated) {
      return <div>Please log in to access this page.</div>;
    }
    
    return <WrappedComponent {...props} user={user} />;
  };
};

const Dashboard = withAuth(({ user }) => (
  <div>
    <h1>Welcome to Dashboard, {user.name}!</h1>
  </div>
));
```

### Render Props Pattern
```jsx
const DataFetcher = ({ url, render }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return render({ data, loading, error });
};

// Usage
const App = () => (
  <div>
    <DataFetcher
      url="/api/users"
      render={({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        return (
          <ul>
            {data?.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        );
      }}
    />
  </div>
);
```

### Compound Components Pattern
```jsx
const Modal = ({ children, isOpen, onClose }) => {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
};

const ModalHeader = ({ children }) => (
  <div className="modal-header">{children}</div>
);

const ModalBody = ({ children }) => (
  <div className="modal-body">{children}</div>
);

const ModalFooter = ({ children }) => (
  <div className="modal-footer">{children}</div>
);

// Attach sub-components
Modal.Header = ModalHeader;
Modal.Body = ModalBody;
Modal.Footer = ModalFooter;

// Usage
const App = () => {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
      
      <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
        <Modal.Header>
          <h2>Modal Title</h2>
        </Modal.Header>
        <Modal.Body>
          <p>Modal content goes here...</p>
        </Modal.Body>
        <Modal.Footer>
          <button onClick={() => setIsModalOpen(false)}>Close</button>
        </Modal.Footer>
      </Modal>
    </div>
  );
};
```

### State Reducer Pattern
```jsx
const useControllableState = ({
  value: controlledValue,
  defaultValue,
  onChange
}) => {
  const [internalValue, setInternalValue] = useState(defaultValue);
  const isControlled = controlledValue !== undefined;
  const value = isControlled ? controlledValue : internalValue;

  const setValue = useCallback((newValue) => {
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onChange?.(newValue);
  }, [isControlled, onChange]);

  return [value, setValue];
};

// Usage in a reusable component
const Toggle = ({ 
  value: controlledValue, 
  defaultValue = false, 
  onChange 
}) => {
  const [isOn, setIsOn] = useControllableState({
    value: controlledValue,
    defaultValue,
    onChange
  });

  return (
    <button
      onClick={() => setIsOn(!isOn)}
      style={{
        backgroundColor: isOn ? 'green' : 'gray',
        color: 'white',
        border: 'none',
        padding: '10px 20px',
        borderRadius: '4px'
      }}
    >
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
};

// Can be used as controlled or uncontrolled
const App = () => {
  const [controlledValue, setControlledValue] = useState(false);

  return (
    <div>
      {/* Uncontrolled */}
      <Toggle defaultValue={true} onChange={(value) => console.log(value)} />
      
      {/* Controlled */}
      <Toggle 
        value={controlledValue} 
        onChange={setControlledValue} 
      />
    </div>
  );
};
```

---

## Best Practices

### Component Organization
```jsx
// ✅ Good: Small, focused components
const UserAvatar = ({ user, size = 'medium' }) => (
  <img 
    src={user.avatar} 
    alt={user.name}
    className={`avatar avatar-${size}`}
  />
);

const UserInfo = ({ user }) => (
  <div className="user-info">
    <h3>{user.name}</h3>
    <p>{user.email}</p>
  </div>
);

const UserCard = ({ user }) => (
  <div className="user-card">
    <UserAvatar user={user} />
    <UserInfo user={user} />
  </div>
);

// ❌ Bad: Large, monolithic component
const UserCard = ({ user }) => (
  <div className="user-card">
    <img 
      src={user.avatar} 
      alt={user.name}
      className="avatar"
    />
    <div className="user-info">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="user-stats">
        <span>Posts: {user.posts}</span>
        <span>Followers: {user.followers}</span>
      </div>
      <div className="user-actions">
        <button>Follow</button>
        <button>Message</button>
        <button>Block</button>
      </div>
    </div>
  </div>
);
```

### State Management Best Practices
```jsx
// ✅ Good: Keep state close to where it's used
const TodoItem = ({ todo, onToggle, onDelete }) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleSave = () => {
    onToggle(todo.id, editText);
    setIsEditing(false);
  };

  return (
    <li>
      {isEditing ? (
        <input
          value={editText}
          onChange={(e) => setEditText(e.target.value)}
          onBlur={handleSave}
          onKeyPress={(e) => e.key === 'Enter' && handleSave()}
        />
      ) : (
        <span onClick={() => setIsEditing(true)}>{todo.text}</span>
      )}
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
};

// ✅ Good: Lift state up when needed by multiple components
const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  const addTodo = (text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  };

  const filteredTodos = todos.filter(todo => {
    if (filter === 'completed') return todo.completed;
    if (filter === 'active') return !todo.completed;
    return true;
  });

  return (
    <div>
      <TodoInput onAdd={addTodo} />
      <TodoFilter filter={filter} onChange={setFilter} />
      <TodoList todos={filteredTodos} />
    </div>
  );
};

// ✅ Good: Use useReducer for complex state logic
const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
    
    case 'CLEAR_CART':
      return { ...state, items: [] };
    
    default:
      return state;
  }
};
```

### Performance Best Practices
```jsx
// ✅ Good: Memoize expensive calculations
const ProductList = ({ products, searchTerm, sortBy }) => {
  const filteredAndSortedProducts = useMemo(() => {
    let result = products.filter(product =>
      product.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
    
    result.sort((a, b) => {
      if (sortBy === 'price') return a.price - b.price;
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      return 0;
    });
    
    return result;
  }, [products, searchTerm, sortBy]);

  return (
    <div>
      {filteredAndSortedProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};

// ✅ Good: Memoize event handlers
const TodoList = ({ todos, onToggle, onDelete }) => {
  const handleToggle = useCallback((id) => {
    onToggle(id);
  }, [onToggle]);

  const handleDelete = useCallback((id) => {
    onDelete(id);
  }, [onDelete]);

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggle}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
};

// ✅ Good: Extract static objects outside component
const FILTER_OPTIONS = [
  { value: 'all', label: 'All' },
  { value: 'active', label: 'Active' },
  { value: 'completed', label: 'Completed' }
];

const TodoFilter = ({ filter, onChange }) => (
  <div>
    {FILTER_OPTIONS.map(option => (
      <button
        key={option.value}
        onClick={() => onChange(option.value)}
        className={filter === option.value ? 'active' : ''}
      >
        {option.label}
      </button>
    ))}
  </div>
);

// ❌ Bad: Creating objects in render
const TodoFilter = ({ filter, onChange }) => (
  <div>
    {[
      { value: 'all', label: 'All' },
      { value: 'active', label: 'Active' },
      { value: 'completed', label: 'Completed' }
    ].map(option => (
      <button key={option.value}>
        {option.label}
      </button>
    ))}
  </div>
);
```

### Error Handling Best Practices
```jsx
// ✅ Good: Handle errors gracefully
const DataComponent = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch('/api/data');
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
      console.error('Failed to fetch data:', err);
    } finally {
      setLoading(false);
    }
  }, []);

  const retry = () => {
    fetchData();
  };

  if (loading) return <LoadingSpinner />;
  
  if (error) {
    return (
      <ErrorMessage 
        message={error} 
        onRetry={retry}
      />
    );
  }

  return data ? <DataDisplay data={data} /> : <NoData />;
};

// ✅ Good: Validate props with default values
const Button = ({ 
  children, 
  variant = 'primary', 
  size = 'medium', 
  disabled = false,
  onClick = () => {},
  ...props 
}) => {
  if (!children) {
    console.warn('Button component requires children');
    return null;
  }

  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
      {...props}
    >
      {children}
    </button>
  );
};
```

### Accessibility Best Practices
```jsx
// ✅ Good: Proper semantic HTML and ARIA attributes
const Modal = ({ isOpen, onClose, title, children }) => {
  const modalRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      modalRef.current?.focus();
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = 'unset';
    }

    return () => {
      document.body.style.overflow = 'unset';
    };
  }, [isOpen]);

  const handleKeyDown = (e) => {
    if (e.key === 'Escape') {
      onClose();
    }
  };

  if (!isOpen) return null;

  return (
    <div
      className="modal-overlay"
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      onClick={onClose}
      onKeyDown={handleKeyDown}
      tabIndex={-1}
      ref={modalRef}
    >
      <div 
        className="modal-content" 
        onClick={(e) => e.stopPropagation()}
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button
            className="modal-close"
            onClick={onClose}
            aria-label="Close modal"
          >
            ×
          </button>
        </header>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
};

// ✅ Good: Form accessibility
const FormField = ({ 
  label, 
  id, 
  required = false, 
  error, 
  children 
}) => (
  <div className="form-field">
    <label htmlFor={id} className={required ? 'required' : ''}>
      {label}
      {required && <span aria-label="required">*</span>}
    </label>
    {children}
    {error && (
      <div 
        className="error-message" 
        role="alert"
        id={`${id}-error`}
      >
        {error}
      </div>
    )}
  </div>
);

const LoginForm = () => {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState({});

  return (
    <form role="form" aria-label="Login form">
      <FormField 
        label="Email" 
        id="email" 
        required 
        error={errors.email}
      >
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={(e) => setFormData(prev => ({ 
            ...prev, 
            email: e.target.value 
          }))}
          aria-describedby={errors.email ? 'email-error' : undefined}
          aria-invalid={!!errors.email}
          required
        />
      </FormField>

      <FormField 
        label="Password" 
        id="password" 
        required 
        error={errors.password}
      >
        <input
          id="password"
          type="password"
          value={formData.password}
          onChange={(e) => setFormData(prev => ({ 
            ...prev, 
            password: e.target.value 
          }))}
          aria-describedby={errors.password ? 'password-error' : undefined}
          aria-invalid={!!errors.password}
          required
        />
      </FormField>

      <button type="submit">Login</button>
    </form>
  );
};
```

### File Structure Best Practices
```
src/
├── components/           # Reusable UI components
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.js
│   │   ├── Button.module.css
│   │   └── index.js
│   ├── Modal/
│   └── Form/
├── pages/               # Page components
│   ├── Home/
│   ├── About/
│   └── Contact/
├── hooks/               # Custom hooks
│   ├── useLocalStorage.js
│   ├── useFetch.js
│   └── useDebounce.js
├── context/             # Context providers
│   ├── AuthContext.js
│   ├── ThemeContext.js
│   └── AppContext.js
├── services/            # API calls and business logic
│   ├── api.js
│   ├── auth.js
│   └── utils.js
├── utils/               # Helper functions
│   ├── constants.js
│   ├── formatters.js
│   └── validators.js
├── styles/              # Global styles
│   ├── globals.css
│   ├── variables.css
│   └── components.css
└── tests/               # Test utilities
    ├── setup.js
    └── mocks.js
```

### Code Style Best Practices
```jsx
// ✅ Good: Consistent naming conventions
const UserProfile = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [userData, setUserData] = useState(null);
  
  const handleUserUpdate = (newData) => {
    setUserData(newData);
  };

  const fetchUserData = async (userId) => {
    // API call
  };

  return (
    <div className="user-profile">
      {/* Component JSX */}
    </div>
  );
};

// ✅ Good: Destructure props early
const UserCard = (props) => {
  const { 
    user, 
    onEdit, 
    onDelete, 
    isEditable = true,
    className = ''
  } = props;

  return (
    <div className={`user-card ${className}`}>
      <h3>{user.name}</h3>
      {isEditable && (
        <div className="actions">
          <button onClick={() => onEdit(user.id)}>Edit</button>
          <button onClick={() => onDelete(user.id)}>Delete</button>
        </div>
      )}
    </div>
  );
};

// ✅ Good: Use early returns for conditionals
const UserProfile = ({ userId }) => {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) {
    return <LoadingSpinner />;
  }

  if (error) {
    return <ErrorMessage error={error} />;
  }

  if (!user) {
    return <NotFound message="User not found" />;
  }

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

// ✅ Good: Extract complex JSX into variables
const Dashboard = ({ user, notifications, tasks }) => {
  const headerContent = (
    <header className="dashboard-header">
      <h1>Welcome back, {user.name}!</h1>
      <NotificationBell count={notifications.length} />
    </header>
  );

  const sidebarContent = (
    <aside className="dashboard-sidebar">
      <UserProfile user={user} />
      <QuickActions />
    </aside>
  );

  const mainContent = (
    <main className="dashboard-main">
      <TaskList tasks={tasks} />
      <RecentActivity />
    </main>
  );

  return (
    <div className="dashboard">
      {headerContent}
      <div className="dashboard-body">
        {sidebarContent}
        {mainContent}
      </div>
    </div>
  );
};
```

---

## Quick Reference

### Common Hooks Summary
```jsx
// State
const [state, setState] = useState(initialValue);

// Side effects
useEffect(() => {
  // Effect logic
  return () => {
    // Cleanup
  };
}, [dependencies]);

// Reducer
const [state, dispatch] = useReducer(reducer, initialState);

// Memoization
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);

// Refs
const ref = useRef(initialValue);

// Context
const value = useContext(MyContext);
```

### Event Handler Patterns
```jsx
// Basic event handler
const handleClick = () => {
  // Handle click
};

// Event with parameters
const handleItemClick = (id) => {
  // Handle item click
};

// Event object access
const handleInputChange = (e) => {
  const { name, value } = e.target;
  // Handle input change
};

// Prevent default
const handleFormSubmit = (e) => {
  e.preventDefault();
  // Handle form submission
};
```

### Conditional Rendering Patterns
```jsx
// Ternary operator
{condition ? <ComponentA /> : <ComponentB />}

// Logical AND
{condition && <Component />}

// Nullish coalescing
{data ?? <LoadingSpinner />}

// Multiple conditions
{loading ? (
  <LoadingSpinner />
) : error ? (
  <ErrorMessage />
) : (
  <DataDisplay data={data} />
)}
```
