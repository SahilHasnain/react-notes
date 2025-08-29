# Forms in React

## Forms Kya Hote Hain?

React me forms HTML ke standard forms jaisa hi kaam karte hain, lekin React ke state management capabilities ke sath combine hoke powerful features provide karte hain.

Forms commonly input fields, checkboxes, radio buttons, selects, aur buttons ko contain karte hain, jisse user data submit kar sakta hai.

## Controlled vs Uncontrolled Components

React me forms handle karne ke do main approaches hain:

### 1. Controlled Components:

- Form elements ki values ko React state ke through control kiya jata hai
- React "single source of truth" maintain karta hai
- har input change par state update hota hai
- More direct form validation

### 2. Uncontrolled Components:

- Form data ko DOM me handle kiya jata hai (React state ke bahar)
- Form values access karne ke liye refs use karte hain
- Less code in simple forms, but less control

## Basic Controlled Form Example

```jsx
import React, { useState } from 'react';

function SimpleForm() {
  const [name, setName] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevents page reload
    alert(`Submitted name: ${name}`);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input 
          type="text" 
          value={name} 
          onChange={(e) => setName(e.target.value)} 
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Handling Multiple Form Inputs

Multiple inputs ko handle karne ke liye ek object state use karna efficient hota hai:

```jsx
import React, { useState } from 'react';

function RegisterForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData, // Previous state ko preserve karo
      [name]: value // Dynamic field update
    });
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
      </div>
      
      <button type="submit">Register</button>
    </form>
  );
}
```

## Form Input Types

React me alag alag input types ko handle karna:

### Text Input

```jsx
<input
  type="text"
  name="firstName"
  value={formData.firstName}
  onChange={handleChange}
/>
```

### Checkbox

```jsx
function CheckboxExample() {
  const [isChecked, setIsChecked] = useState(false);
  
  const handleCheckboxChange = (e) => {
    setIsChecked(e.target.checked);
  };
  
  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={isChecked}
          onChange={handleCheckboxChange}
        />
        Subscribe to newsletter
      </label>
      <p>Subscribed: {isChecked ? 'Yes' : 'No'}</p>
    </div>
  );
}
```

### Multiple Checkboxes

```jsx
function HobbiesForm() {
  const [hobbies, setHobbies] = useState({
    reading: false,
    sports: false,
    coding: false
  });
  
  const handleCheckboxChange = (e) => {
    const { name, checked } = e.target;
    setHobbies({
      ...hobbies,
      [name]: checked
    });
  };
  
  return (
    <div>
      <h3>Select Your Hobbies:</h3>
      <label>
        <input
          type="checkbox"
          name="reading"
          checked={hobbies.reading}
          onChange={handleCheckboxChange}
        />
        Reading
      </label>
      <br />
      <label>
        <input
          type="checkbox"
          name="sports"
          checked={hobbies.sports}
          onChange={handleCheckboxChange}
        />
        Sports
      </label>
      <br />
      <label>
        <input
          type="checkbox"
          name="coding"
          checked={hobbies.coding}
          onChange={handleCheckboxChange}
        />
        Coding
      </label>
    </div>
  );
}
```

### Radio Buttons

```jsx
function GenderSelection() {
  const [gender, setGender] = useState('');
  
  const handleGenderChange = (e) => {
    setGender(e.target.value);
  };
  
  return (
    <div>
      <h3>Select Gender:</h3>
      <label>
        <input
          type="radio"
          name="gender"
          value="male"
          checked={gender === 'male'}
          onChange={handleGenderChange}
        />
        Male
      </label>
      <br />
      <label>
        <input
          type="radio"
          name="gender"
          value="female"
          checked={gender === 'female'}
          onChange={handleGenderChange}
        />
        Female
      </label>
      <br />
      <label>
        <input
          type="radio"
          name="gender"
          value="other"
          checked={gender === 'other'}
          onChange={handleGenderChange}
        />
        Other
      </label>
    </div>
  );
}
```

### Select Dropdown

```jsx
function CountrySelect() {
  const [country, setCountry] = useState('');
  
  const handleSelectChange = (e) => {
    setCountry(e.target.value);
  };
  
  return (
    <div>
      <label htmlFor="country">Select Country:</label>
      <select
        id="country"
        value={country}
        onChange={handleSelectChange}
      >
        <option value="">--Select a country--</option>
        <option value="usa">USA</option>
        <option value="canada">Canada</option>
        <option value="uk">UK</option>
        <option value="australia">Australia</option>
        <option value="india">India</option>
        <option value="pakistan">Pakistan</option>
      </select>
      {country && <p>Selected: {country}</p>}
    </div>
  );
}
```

### Multi-select Dropdown

```jsx
function LanguageSelect() {
  const [languages, setLanguages] = useState([]);
  
  const handleMultiSelectChange = (e) => {
    const values = Array.from(
      e.target.selectedOptions,
      option => option.value
    );
    setLanguages(values);
  };
  
  return (
    <div>
      <label htmlFor="languages">Select Languages:</label>
      <select
        id="languages"
        multiple
        value={languages}
        onChange={handleMultiSelectChange}
      >
        <option value="javascript">JavaScript</option>
        <option value="python">Python</option>
        <option value="java">Java</option>
        <option value="csharp">C#</option>
        <option value="php">PHP</option>
      </select>
      <p>Selected: {languages.join(', ')}</p>
    </div>
  );
}
```

### Textarea

```jsx
function CommentForm() {
  const [comment, setComment] = useState('');
  
  return (
    <div>
      <label htmlFor="comment">Comment:</label>
      <textarea
        id="comment"
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        rows="4"
        cols="50"
      />
    </div>
  );
}
```

### File Input (Uncontrolled)

File inputs ko typically uncontrolled components ke tarah handle karte hain:

```jsx
function FileUploadForm() {
  const fileInputRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const file = fileInputRef.current.files[0];
    if (file) {
      console.log('Selected file:', file.name);
      // Handle file upload logic
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="file">Select File:</label>
      <input
        type="file"
        id="file"
        ref={fileInputRef}
      />
      <button type="submit">Upload</button>
    </form>
  );
}
```

## Form Validation

React forms me validation implement karna:

```jsx
import React, { useState } from 'react';

function ValidatedForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors({
        ...errors,
        [name]: ''
      });
    }
  };
  
  const validateForm = () => {
    let tempErrors = {};
    let isValid = true;
    
    if (!formData.username.trim()) {
      tempErrors.username = 'Username is required';
      isValid = false;
    }
    
    if (!formData.email) {
      tempErrors.email = 'Email is required';
      isValid = false;
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      tempErrors.email = 'Email is invalid';
      isValid = false;
    }
    
    if (!formData.password) {
      tempErrors.password = 'Password is required';
      isValid = false;
    } else if (formData.password.length < 6) {
      tempErrors.password = 'Password must be at least 6 characters';
      isValid = false;
    }
    
    setErrors(tempErrors);
    return isValid;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (validateForm()) {
      console.log('Form submitted successfully:', formData);
      // Submit to server or process data
    } else {
      console.log('Form validation failed');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
        />
        {errors.username && <p className="error">{errors.username}</p>}
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <p className="error">{errors.email}</p>}
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <p className="error">{errors.password}</p>}
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Form Submission and Handling

Form data ko submit karne ka example:

```jsx
import React, { useState } from 'react';

function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitSuccess, setSubmitSuccess] = useState(false);
  const [submitError, setSubmitError] = useState('');
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    setSubmitSuccess(false);
    setSubmitError('');
    
    try {
      // API call simulation
      await new Promise(resolve => setTimeout(resolve, 1500));
      
      // Success case
      console.log('Form submitted:', formData);
      setSubmitSuccess(true);
      setFormData({ name: '', email: '', message: '' }); // Reset form
    } catch (error) {
      // Error case
      setSubmitError('Failed to submit form. Please try again.');
      console.error('Submission error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <div>
      <h2>Contact Us</h2>
      
      {submitSuccess && (
        <div className="success-message">
          Thank you for your message! We'll get back to you soon.
        </div>
      )}
      
      {submitError && (
        <div className="error-message">{submitError}</div>
      )}
      
      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="name">Name:</label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
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
            onChange={handleChange}
            required
          />
        </div>
        
        <div>
          <label htmlFor="message">Message:</label>
          <textarea
            id="message"
            name="message"
            value={formData.message}
            onChange={handleChange}
            required
            rows="5"
          />
        </div>
        
        <button 
          type="submit" 
          disabled={isSubmitting}
        >
          {isSubmitting ? 'Submitting...' : 'Submit'}
        </button>
      </form>
    </div>
  );
}
```

## Uncontrolled Components with useRef

Uncontrolled components approach using the `useRef` hook:

```jsx
import React, { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef();
  const emailRef = useRef();
  const messageRef = useRef();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
      message: messageRef.current.value
    };
    
    console.log('Form data:', formData);
    // Process the form data
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name:</label>
        <input
          type="text"
          id="name"
          ref={nameRef}
          defaultValue=""
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          ref={emailRef}
          defaultValue=""
        />
      </div>
      
      <div>
        <label htmlFor="message">Message:</label>
        <textarea
          id="message"
          ref={messageRef}
          defaultValue=""
        />
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Form Libraries

Bade ya complex forms ke liye third-party libraries helpful hoti hain:

- **React Hook Form**: Minimal re-renders, performance focused
- **Formik**: Complete solution for forms with validation
- **Final Form**: High-performance form state management

These libraries automate many form handling tasks like state management, validation, error handling, aur form submission.

## Summary

- Forms React me user input collect karne ke primary way hain
- Controlled components React state ko source of truth maintain karte hain
- Uncontrolled components DOM state ko use karte hain with refs
- Different input types (text, checkbox, radio, select) ke liye specific handling needed hai
- Form validation React me easily implement ki ja sakti hai
- Complex forms ke liye specialized libraries (React Hook Form, Formik) more efficient ho sakti hain
- Form submission ke time loading states aur error handling important hain 