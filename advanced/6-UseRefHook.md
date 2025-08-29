# useRef Hook in React

## useRef Kya Hai?

`useRef` React ka built-in hook hai jo component ke rendering cycles ke beech persistent reference create karta hai. 

Ye ek mutable object return karta hai jiska `.current` property initially passed argument se initialize hota hai. Returned object component ki puri lifetime ke liye persist rehta hai.

Unlike state, useRef ka value change karne se component re-render nahi hota.

## useRef Kyu Use Karte Hain?

1. **DOM Elements ka Direct Access**: React components ke DOM nodes ko directly access karna
2. **Mutable Values Store Karna**: Values store karna jo rendering ko trigger nahi karein
3. **Previous Values Remember Karna**: Pichle renders ke values ko track karna
4. **Cleanup Timers and Subscriptions**: Timers, intervals aur subscriptions ko store karna aur unhe cleanup karna

## Basic Syntax

```jsx
import React, { useRef } from 'react';

function MyComponent() {
  const myRef = useRef(initialValue);
  
  // Access the current value
  console.log(myRef.current);
  
  // Update the ref value (doesn't cause re-render)
  myRef.current = newValue;
  
  return <div ref={myRef}>Element</div>;
}
```

## DOM Elements Access with useRef

useRef ka most common use case hai DOM elements ko access karna:

```jsx
import React, { useRef, useEffect } from 'react';

function TextInputWithFocus() {
  // Create a ref
  const inputRef = useRef(null);
  
  // Focus the input element when component mounts
  useEffect(() => {
    // Access the DOM node through current
    inputRef.current.focus();
  }, []); // Empty dependency array means run once after mounting
  
  return (
    <div>
      <input 
        ref={inputRef} 
        type="text" 
        placeholder="This input will be focused on mount" 
      />
    </div>
  );
}
```

## Storing Mutable Values

useRef component re-renders ke beech values ko preserve karna:

```jsx
import React, { useState, useRef } from 'react';

function StopWatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  // Store interval ID in a ref so it persists between renders
  // and doesn't cause re-renders when changed
  const intervalRef = useRef(null);
  
  const startTimer = () => {
    if (!isRunning) {
      setIsRunning(true);
      
      intervalRef.current = setInterval(() => {
        setTime(prevTime => prevTime + 1);
      }, 1000);
    }
  };
  
  const stopTimer = () => {
    if (isRunning) {
      clearInterval(intervalRef.current);
      setIsRunning(false);
    }
  };
  
  const resetTimer = () => {
    clearInterval(intervalRef.current);
    setIsRunning(false);
    setTime(0);
  };
  
  // Clean up interval on unmount
  useEffect(() => {
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);
  
  // Format time as mm:ss
  const formatTime = () => {
    const minutes = Math.floor(time / 60);
    const seconds = time % 60;
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  };
  
  return (
    <div>
      <h2>Stopwatch</h2>
      <div className="time">{formatTime()}</div>
      
      <div className="controls">
        {!isRunning ? (
          <button onClick={startTimer}>Start</button>
        ) : (
          <button onClick={stopTimer}>Pause</button>
        )}
        <button onClick={resetTimer}>Reset</button>
      </div>
    </div>
  );
}
```

## Previous Value Tracking

useRef se previous state values track karne ka pattern:

```jsx
import React, { useState, useRef, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // Store previous count value
  const prevCountRef = useRef();
  
  useEffect(() => {
    // Update ref after render
    prevCountRef.current = count;
  }, [count]);
  
  // Get previous value (initially undefined)
  const prevCount = prevCountRef.current;
  
  return (
    <div>
      <h2>Counter: {count}</h2>
      <p>Previous value: {prevCount !== undefined ? prevCount : 'None yet'}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

## Custom Hook for Previous Value

Previous value tracking ko reusable custom hook me convert karna:

```jsx
import { useEffect, useRef } from 'react';

// Custom hook to keep track of previous value
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage in component
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <h2>Now: {count}, Before: {prevCount}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

## Form Handling with useRef

Uncontrolled form components me useRef use karna:

```jsx
import React, { useRef } from 'react';

function UncontrolledForm() {
  // Create refs for form inputs
  const nameRef = useRef(null);
  const emailRef = useRef(null);
  const messageRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Access input values directly through refs
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
      message: messageRef.current.value
    };
    
    console.log('Form submitted:', formData);
    
    // Reset form
    e.target.reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name:</label>
        <input
          id="name"
          type="text"
          ref={nameRef}
          required
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          ref={emailRef}
          required
        />
      </div>
      
      <div>
        <label htmlFor="message">Message:</label>
        <textarea
          id="message"
          ref={messageRef}
          required
        />
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## useRef with forwardRef

Parent components se child components ke DOM elements ko access karna:

```jsx
import React, { useRef, forwardRef, useImperativeHandle } from 'react';

// Child component with forwarded ref
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// More complex example with custom imperative handle
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);
  
  // Expose only specific methods to parent
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    },
    getValue: () => {
      return inputRef.current.value;
    }
  }));
  
  return <input ref={inputRef} {...props} />;
});

// Parent component
function FormWithRefs() {
  const simpleInputRef = useRef(null);
  const customInputRef = useRef(null);
  
  const focusSimpleInput = () => {
    simpleInputRef.current.focus();
  };
  
  const focusCustomInput = () => {
    customInputRef.current.focus();
  };
  
  const clearCustomInput = () => {
    customInputRef.current.clear();
  };
  
  const getCustomInputValue = () => {
    alert(customInputRef.current.getValue());
  };
  
  return (
    <div>
      <h2>Form with Refs</h2>
      
      <div>
        <Input 
          ref={simpleInputRef}
          type="text"
          placeholder="Simple forwarded ref"
        />
        <button onClick={focusSimpleInput}>
          Focus Simple Input
        </button>
      </div>
      
      <div>
        <CustomInput 
          ref={customInputRef}
          type="text"
          placeholder="Custom imperative handle"
        />
        <button onClick={focusCustomInput}>Focus</button>
        <button onClick={clearCustomInput}>Clear</button>
        <button onClick={getCustomInputValue}>Get Value</button>
      </div>
    </div>
  );
}
```

## Managing Focus in Complex UIs

Complex forms me focus management:

```jsx
import React, { useRef } from 'react';

function MultiStepForm() {
  const step1Ref = useRef(null);
  const step2Ref = useRef(null);
  const step3Ref = useRef(null);
  const submitRef = useRef(null);
  
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    address: '',
    phone: ''
  });
  
  // Handle input changes
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  // Navigate to next step
  const nextStep = () => {
    if (currentStep < 3) {
      setCurrentStep(currentStep + 1);
      
      // Focus on the first input of the next step
      setTimeout(() => {
        if (currentStep === 1) step2Ref.current.focus();
        if (currentStep === 2) step3Ref.current.focus();
      }, 0);
    }
  };
  
  // Navigate to previous step
  const prevStep = () => {
    if (currentStep > 1) {
      setCurrentStep(currentStep - 1);
      
      // Focus on the first input of the previous step
      setTimeout(() => {
        if (currentStep === 2) step1Ref.current.focus();
        if (currentStep === 3) step2Ref.current.focus();
      }, 0);
    }
  };
  
  // Submit the form
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
    // Process form data...
  };
  
  // Conditionally render steps
  const renderStep = () => {
    switch (currentStep) {
      case 1:
        return (
          <div className="step">
            <h3>Personal Information</h3>
            <div>
              <label htmlFor="name">Name:</label>
              <input
                id="name"
                name="name"
                type="text"
                ref={step1Ref}
                value={formData.name}
                onChange={handleChange}
                required
              />
            </div>
            <div>
              <label htmlFor="email">Email:</label>
              <input
                id="email"
                name="email"
                type="email"
                value={formData.email}
                onChange={handleChange}
                required
              />
            </div>
          </div>
        );
      case 2:
        return (
          <div className="step">
            <h3>Contact Information</h3>
            <div>
              <label htmlFor="address">Address:</label>
              <input
                id="address"
                name="address"
                type="text"
                ref={step2Ref}
                value={formData.address}
                onChange={handleChange}
                required
              />
            </div>
            <div>
              <label htmlFor="phone">Phone Number:</label>
              <input
                id="phone"
                name="phone"
                type="tel"
                value={formData.phone}
                onChange={handleChange}
                required
              />
            </div>
          </div>
        );
      case 3:
        return (
          <div className="step">
            <h3>Review Information</h3>
            <div ref={step3Ref} tabIndex="-1">
              <p><strong>Name:</strong> {formData.name}</p>
              <p><strong>Email:</strong> {formData.email}</p>
              <p><strong>Address:</strong> {formData.address}</p>
              <p><strong>Phone:</strong> {formData.phone}</p>
            </div>
          </div>
        );
      default:
        return null;
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Multi-Step Form</h2>
      
      {renderStep()}
      
      <div className="navigation">
        {currentStep > 1 && (
          <button type="button" onClick={prevStep}>
            Previous
          </button>
        )}
        
        {currentStep < 3 ? (
          <button type="button" onClick={nextStep}>
            Next
          </button>
        ) : (
          <button type="submit" ref={submitRef}>
            Submit
          </button>
        )}
      </div>
    </form>
  );
}
```

## Measuring DOM Elements

useRef ka use element sizes aur positions measure karne ke liye:

```jsx
import React, { useRef, useState, useEffect } from 'react';

function MeasureExample() {
  const divRef = useRef(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  const [position, setPosition] = useState({ top: 0, left: 0 });
  
  // Measure on mount and when window resizes
  useEffect(() => {
    const measureElement = () => {
      if (divRef.current) {
        const { offsetWidth, offsetHeight } = divRef.current;
        const { top, left } = divRef.current.getBoundingClientRect();
        
        setDimensions({
          width: offsetWidth,
          height: offsetHeight
        });
        
        setPosition({
          top: top + window.scrollY,
          left: left + window.scrollX
        });
      }
    };
    
    // Measure on mount
    measureElement();
    
    // Re-measure on window resize
    window.addEventListener('resize', measureElement);
    
    // Cleanup
    return () => {
      window.removeEventListener('resize', measureElement);
    };
  }, []);
  
  return (
    <div>
      <h2>Element Measurements</h2>
      
      <div
        ref={divRef}
        style={{
          width: '100%',
          maxWidth: '400px',
          padding: '20px',
          margin: '20px 0',
          border: '2px solid blue',
          backgroundColor: '#e0e0ff'
        }}
      >
        This div's dimensions and position are being measured
      </div>
      
      <div>
        <h3>Dimensions:</h3>
        <p>Width: {dimensions.width}px</p>
        <p>Height: {dimensions.height}px</p>
        
        <h3>Position:</h3>
        <p>Top: {position.top}px</p>
        <p>Left: {position.left}px</p>
      </div>
    </div>
  );
}
```

## Implementing Scroll to Top Button

useRef ka use scroll position track karne aur scroll to top functionality implement karne ke liye:

```jsx
import React, { useState, useEffect, useRef } from 'react';

function ScrollToTopExample() {
  const [isVisible, setIsVisible] = useState(false);
  const containerRef = useRef(null);
  
  // Track scroll position and show/hide button
  useEffect(() => {
    const container = containerRef.current;
    
    const handleScroll = () => {
      // Show button when scrolled down 200px or more
      setIsVisible(container.scrollTop > 200);
    };
    
    // Add scroll event listener
    container.addEventListener('scroll', handleScroll);
    
    // Cleanup
    return () => {
      container.removeEventListener('scroll', handleScroll);
    };
  }, []);
  
  // Scroll to top function
  const scrollToTop = () => {
    containerRef.current.scrollTo({
      top: 0,
      behavior: 'smooth'
    });
  };
  
  return (
    <div>
      <h2>Scroll To Top Example</h2>
      
      <div
        ref={containerRef}
        style={{
          height: '300px',
          overflow: 'auto',
          border: '1px solid #ccc',
          padding: '10px'
        }}
      >
        {/* Generate lots of content to enable scrolling */}
        {Array.from({ length: 20 }, (_, i) => (
          <div key={i} style={{ marginBottom: '30px' }}>
            <h3>Section {i + 1}</h3>
            <p>
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
              Nullam euismod, nisi vel consectetur euismod, nisi nisl
              consectetur nisi, nec tincidunt nisi nisl euismod.
            </p>
          </div>
        ))}
      </div>
      
      {isVisible && (
        <button
          onClick={scrollToTop}
          style={{
            position: 'fixed',
            bottom: '20px',
            right: '20px',
            padding: '10px 15px',
            backgroundColor: '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '5px',
            cursor: 'pointer'
          }}
        >
          ↑ Top
        </button>
      )}
    </div>
  );
}
```

## Implementing a Reusable Click Outside Handler

useRef ka use modal, dropdown, ya popover components ko click outside event se close karne ke liye:

```jsx
import React, { useState, useRef, useEffect } from 'react';

// Custom hook for detecting clicks outside an element
function useClickOutside(callback) {
  const ref = useRef(null);
  
  useEffect(() => {
    const handleClickOutside = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        callback();
      }
    };
    
    document.addEventListener('mousedown', handleClickOutside);
    
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [callback]);
  
  return ref;
}

// Dropdown component using click outside detection
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  
  const closeDropdown = () => {
    setIsOpen(false);
  };
  
  const dropdownRef = useClickOutside(closeDropdown);
  
  return (
    <div className="dropdown-container" ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Close Menu' : 'Open Menu'}
      </button>
      
      {isOpen && (
        <div className="dropdown-menu">
          <ul>
            <li>Profile</li>
            <li>Settings</li>
            <li>Logout</li>
          </ul>
        </div>
      )}
    </div>
  );
}

// Modal component using click outside detection
function Modal({ isOpen, onClose, children }) {
  const modalRef = useClickOutside(onClose);
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div className="modal-content" ref={modalRef}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}

// App using both components
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div>
      <h2>Click Outside Examples</h2>
      
      <div style={{ marginBottom: '20px' }}>
        <Dropdown />
      </div>
      
      <div>
        <button onClick={() => setIsModalOpen(true)}>
          Open Modal
        </button>
        
        <Modal 
          isOpen={isModalOpen} 
          onClose={() => setIsModalOpen(false)}
        >
          <h3>Modal Content</h3>
          <p>This modal closes when you click outside of it.</p>
        </Modal>
      </div>
    </div>
  );
}
```

## Video/Audio Player Control

useRef ka use media elements ko control karne ke liye:

```jsx
import React, { useRef, useState } from 'react';

function VideoPlayer() {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);
  const [progress, setProgress] = useState(0);
  const [volume, setVolume] = useState(1);
  
  // Play/pause video
  const togglePlay = () => {
    if (videoRef.current.paused) {
      videoRef.current.play();
      setIsPlaying(true);
    } else {
      videoRef.current.pause();
      setIsPlaying(false);
    }
  };
  
  // Update progress bar as video plays
  const handleTimeUpdate = () => {
    const video = videoRef.current;
    if (!video) return;
    
    const progressValue = (video.currentTime / video.duration) * 100;
    setProgress(progressValue);
  };
  
  // Seek video to a specific position
  const handleSeek = (e) => {
    const video = videoRef.current;
    const seekPosition = (e.nativeEvent.offsetX / e.target.clientWidth) * video.duration;
    video.currentTime = seekPosition;
  };
  
  // Handle volume change
  const handleVolumeChange = (e) => {
    const value = e.target.value;
    setVolume(value);
    videoRef.current.volume = value;
  };
  
  // Handle mute toggle
  const toggleMute = () => {
    const video = videoRef.current;
    video.muted = !video.muted;
    setVolume(video.muted ? 0 : video.volume);
  };
  
  return (
    <div className="video-player">
      <video
        ref={videoRef}
        onTimeUpdate={handleTimeUpdate}
        onClick={togglePlay}
        onEnded={() => setIsPlaying(false)}
        src="https://example.com/sample-video.mp4"
        style={{ width: '100%', maxWidth: '600px' }}
      />
      
      <div className="video-controls">
        <button onClick={togglePlay}>
          {isPlaying ? 'Pause' : 'Play'}
        </button>
        
        <div 
          className="progress-bar" 
          onClick={handleSeek}
          style={{ 
            height: '10px',
            backgroundColor: '#ddd',
            cursor: 'pointer',
            width: '100%'
          }}
        >
          <div 
            className="progress-fill"
            style={{
              width: `${progress}%`,
              height: '100%',
              backgroundColor: '#007bff'
            }}
          />
        </div>
        
        <div className="volume-controls">
          <button onClick={toggleMute}>
            {volume === 0 ? 'Unmute' : 'Mute'}
          </button>
          <input
            type="range"
            min="0"
            max="1"
            step="0.1"
            value={volume}
            onChange={handleVolumeChange}
          />
        </div>
      </div>
    </div>
  );
}
```

## useState vs useRef

useState aur useRef ke beech difference:

| Feature | useState | useRef |
|---------|----------|--------|
| **Re-renders** | State change causes re-render | Ref change doesn't cause re-render |
| **Value Access** | Only in render phase | Anytime (current property) |
| **Updates** | Asynchronous batched updates | Synchronous immediate updates |
| **Use Case** | UI-related data | DOM references, instance values |
| **Initial Value** | Function or value | Value only |

```jsx
import React, { useState, useRef } from 'react';

function StateVsRef() {
  const [stateCount, setStateCount] = useState(0);
  const refCount = useRef(0);
  
  const incrementState = () => {
    setStateCount(stateCount + 1);
  };
  
  const incrementRef = () => {
    refCount.current += 1;
    console.log('refCount:', refCount.current);
    // No re-render happens here
  };
  
  return (
    <div>
      <h2>useState vs useRef</h2>
      
      <div>
        <p>State count: {stateCount}</p>
        <button onClick={incrementState}>
          Increment State (causes re-render)
        </button>
      </div>
      
      <div>
        <p>Ref count: {refCount.current}</p>
        <button onClick={incrementRef}>
          Increment Ref (doesn't cause re-render)
        </button>
        <p>
          Note: Ref value updates in console, but you won't see it in the UI
          until the next render caused by something else.
        </p>
      </div>
    </div>
  );
}
```

## Common Anti-patterns and Solutions

### 1. Changing Refs During Render

❌ **Anti-pattern:**

```jsx
function BadComponent() {
  const ref = useRef(0);
  
  // Bad: Modifying ref during render
  ref.current += 1;
  
  return <div>{ref.current}</div>; // Will cause infinite re-renders in StrictMode
}
```

✅ **Better approach:**

```jsx
function GoodComponent() {
  const ref = useRef(0);
  
  // Use useEffect to modify refs after render
  useEffect(() => {
    ref.current += 1;
  });
  
  return <div>{ref.current}</div>;
}
```

### 2. Using useRef for Derived State

❌ **Anti-pattern:**

```jsx
function DerivedStateWithRef({ value }) {
  const prevValueRef = useRef(value);
  
  // Updating ref directly during render
  const hasChanged = value !== prevValueRef.current;
  prevValueRef.current = value; // BAD: updating during render
  
  return <div>{hasChanged ? 'Changed' : 'Not Changed'}</div>;
}
```

✅ **Better approach:**

```jsx
function DerivedStateWithUseEffect({ value }) {
  const prevValueRef = useRef(value);
  const [hasChanged, setHasChanged] = useState(false);
  
  useEffect(() => {
    if (value !== prevValueRef.current) {
      setHasChanged(true);
    }
    prevValueRef.current = value;
  }, [value]);
  
  return <div>{hasChanged ? 'Changed' : 'Not Changed'}</div>;
}
```

## Summary

- useRef ek mutable object provide karta hai jo render cycles ke beech persist rehta hai
- DOM elements ko directly access karne ke liye, useRef most common approach hai
- Ref changes component ko re-render nahi karate, unlike state changes
- Common use cases: DOM manipulation, storing instance values, timers, previous value tracking
- forwardRef ke sath combine karke child components ke elements ko parent se access kar sakte hain
- useRef aur useState different use cases ke liye hain - ref for non-reactive data, state for reactive UI data
``` 
</rewritten_file>