# Introduction to React

## React Kya Hai?

React ek JavaScript library hai jo UI (User Interface) banane ke liye use hoti hai — especially single-page applications (SPA) ke liye.

Isko Facebook ne develop kiya hai, aur aaj duniya ke top companies isko use karti hain (Facebook, Instagram, Netflix, Zomato, etc.)

## React ke Features

### 1. Component-Based Architecture
- Tum apni app ko chhote chhote blocks (components) me tod dete ho
- Jaise Lego blocks — har block ek chhoti functionality handle karta hai
- Components reusable hote hain

### 2. JSX (JavaScript + XML)
- JavaScript me HTML likhne ka tarika
- Isse code clean aur readable banta hai

```jsx
const element = <h1>Hello World</h1>;
```

### 3. Virtual DOM
- React real DOM ko directly touch nahi karta
- Pehle ek virtual copy me changes karta hai, phir sirf jo changes huye wo hi real DOM me apply karta hai = fast performance

### 4. Unidirectional Data Flow
- Data sirf ek direction me flow karta hai: parent → child
- Isse bugs trace karna easy hota hai

## React Install Kaise Karte Hain?

React app create karne ke liye hum use karenge:

```bash
npx create-react-app my-app
```

### Step by Step:
1. Terminal ya VS Code open karo
2. Run karo:
```bash
npx create-react-app my-first-react-app
```
3. Jab complete ho jaye, folder me jao:
```bash
cd my-first-react-app
```
4. Project ko start karo:
```bash
npm start
```

Ye command tumhara React app browser me open kar dega on http://localhost:3000

## Folder Structure Explanation

```
my-first-react-app/
├── node_modules/          → Saari installed libraries hoti hain
├── public/
│   └── index.html         → React yahin pe inject hota hai
├── src/
│   ├── App.js             → Tumhara main component
│   ├── index.js           → Yahan se React DOM render hota hai
├── package.json           → Saari dependencies + scripts ki list
```

## React Flow Samjho

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

Yeh kya kar raha hai?
- App component ko render kar raha hai
- Wo App.js se import hua hai
- index.html ke `<div id="root"></div>` me inject ho raha hai

## Summary

- React ek JavaScript library hai
- Components architecture par based hai
- Virtual DOM use karta hai performance ke liye
- JSX syntax se HTML-like code likh sakte hain
- Unidirectional data flow follow karta hai
- create-react-app se easily setup kar sakte hain 