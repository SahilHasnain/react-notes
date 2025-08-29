# Modern React UI Design Patterns

## Responsive Layout Patterns

Modern web applications mein responsive design zaruri hai. Yahan hum CRM Dashboard UI Kit se responsive layout patterns dekhenge.

### Flexbox Based Layout:

```tsx
// Flexbox based responsive layout
<div className="flex flex-col md:flex-row gap-4">
  {/* Mobile par ye vertically stack honge, desktop par horizontally */}
  <div className="w-full md:w-1/3">Sidebar Content</div>
  <div className="w-full md:w-2/3">Main Content</div>
</div>
```

### Conditional Rendering based on Screen Size:

```tsx
// useWindowSize hook ke sath conditional rendering
function DashboardLayout() {
  const { isMobile } = useWindowSize();
  
  return (
    <div className="dashboard-layout">
      {/* Mobile par sidebar hide hoga */}
      {!isMobile && <Sidebar />}
      
      <main>
        {/* Mobile specific navigation */}
        {isMobile && <MobileNav />}
        
        {/* Content */}
        <div className="content">
          {children}
        </div>
      </main>
    </div>
  );
}
```

## Component Composition with Slots

Modern React applications mein component composition ek powerful pattern hai. Slot pattern ek aisa approach hai jahan parent component children ko place karne ke liye specific "slots" provide karta hai.

### Slot Pattern Example:

```tsx
interface DashboardCardProps {
  title: string;
  actionButton?: React.ReactNode;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

function DashboardCard({ title, actionButton, children, footer }: DashboardCardProps) {
  return (
    <div className="card">
      <div className="card-header">
        <h3 className="card-title">{title}</h3>
        {actionButton && <div className="card-actions">{actionButton}</div>}
      </div>
      
      <div className="card-body">{children}</div>
      
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// Usage
<DashboardCard
  title="Sales Overview"
  actionButton={<Button>View All</Button>}
  footer={<p>Last updated: Today</p>}
>
  <SalesChart data={salesData} />
</DashboardCard>
```

## Compound Components

Compound components ek pattern hai jahan related components ko aik family ke taur par design kiya jata hai. Ye pattern component API ko simplified banata hai aur components ko aik context mein share karne ki ijazat deta hai.

### Compound Component Example:

```tsx
// Tabs compound component system
import React, { createContext, useContext, useState, ReactNode } from 'react';

// Context creation
interface TabsContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = createContext<TabsContextType | undefined>(undefined);

// Root component
interface TabsProps {
  defaultTab: string;
  children: ReactNode;
}

function Tabs({ defaultTab, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs-container">
        {children}
      </div>
    </TabsContext.Provider>
  );
}

// Tab headers container
function TabList({ children }: { children: ReactNode }) {
  return <div className="tab-list">{children}</div>;
}

// Individual tab header
interface TabProps {
  id: string;
  children: ReactNode;
}

function Tab({ id, children }: TabProps) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');
  
  const { activeTab, setActiveTab } = context;
  const isActive = activeTab === id;
  
  return (
    <button 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

// Tab content container
function TabPanels({ children }: { children: ReactNode }) {
  return <div className="tab-panels">{children}</div>;
}

// Individual tab content
function TabPanel({ id, children }: TabProps) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');
  
  const { activeTab } = context;
  if (activeTab !== id) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Attach child components to parent
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;
Tabs.TabPanel = TabPanel;

// Usage example
function DashboardTabs() {
  return (
    <Tabs defaultTab="overview">
      <Tabs.TabList>
        <Tabs.Tab id="overview">Overview</Tabs.Tab>
        <Tabs.Tab id="analytics">Analytics</Tabs.Tab>
        <Tabs.Tab id="settings">Settings</Tabs.Tab>
      </Tabs.TabList>
      
      <Tabs.TabPanels>
        <Tabs.TabPanel id="overview">
          <OverviewContent />
        </Tabs.TabPanel>
        <Tabs.TabPanel id="analytics">
          <AnalyticsContent />
        </Tabs.TabPanel>
        <Tabs.TabPanel id="settings">
          <SettingsContent />
        </Tabs.TabPanel>
      </Tabs.TabPanels>
    </Tabs>
  );
}
```

## Headless Components

Headless components ek modern UI pattern hai jahan component logic aur state management provide kiya jata hai, lekin UI rendering client par chor diya jata hai. Is tarah se, component behavior consistent rehta hai lekin UI/styling flexible hota hai.

### Headless Component Example:

```tsx
// Dropdown as a headless component
import { useState } from 'react';

interface UseDropdownProps {
  initialOpen?: boolean;
}

interface UseDropdownReturn {
  isOpen: boolean;
  toggle: () => void;
  open: () => void;
  close: () => void;
  buttonProps: {
    onClick: () => void;
    'aria-expanded': boolean;
  };
  menuProps: {
    'aria-hidden': boolean;
  };
}

// Hook that provides dropdown functionality without UI
function useDropdown({ initialOpen = false }: UseDropdownProps = {}): UseDropdownReturn {
  const [isOpen, setIsOpen] = useState(initialOpen);
  
  const toggle = () => setIsOpen(prev => !prev);
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  
  // Accessibility props
  const buttonProps = {
    onClick: toggle,
    'aria-expanded': isOpen
  };
  
  const menuProps = {
    'aria-hidden': !isOpen
  };
  
  return {
    isOpen,
    toggle,
    open,
    close,
    buttonProps,
    menuProps
  };
}

// Usage
function CustomDropdown() {
  const { isOpen, buttonProps, menuProps } = useDropdown();
  
  return (
    <div className="dropdown">
      <button 
        className="dropdown-button" 
        {...buttonProps}
      >
        Menu
      </button>
      
      {isOpen && (
        <ul className="dropdown-menu" {...menuProps}>
          <li>Profile</li>
          <li>Settings</li>
          <li>Logout</li>
        </ul>
      )}
    </div>
  );
}
```

## App Layout Structure

CRM Dashboard UI Kit aik complex app layout structure ka use karta hai, jo content ko alag-alag areas mein organize karta hai.

### App Layout Structure Example:

```tsx
function DashboardLayout({ children }: { children: React.ReactNode }) {
  const { isSidebarCollapsed } = useDashboard();
  
  return (
    <div className="layout-container">
      {/* Sidebar */}
      <aside className={`sidebar ${isSidebarCollapsed ? 'collapsed' : ''}`}>
        <SidebarContent />
      </aside>
      
      {/* Main content area */}
      <div className="content-wrapper">
        {/* Header/Topbar */}
        <header className="topbar">
          <Topbar />
        </header>
        
        {/* Main content */}
        <main className="main-content">
          {children}
        </main>
        
        {/* Footer */}
        <footer className="footer">
          <FooterContent />
        </footer>
      </div>
    </div>
  );
}
```

## Theming and Styling Patterns

Modern React applications mein consistent theming aur styling zaruri hai.

### Theme Provider Pattern:

```tsx
// Theme provider
import { createContext, useContext, useState, ReactNode } from 'react';

interface Theme {
  primary: string;
  secondary: string;
  text: string;
  background: string;
}

const lightTheme: Theme = {
  primary: '#0070f3',
  secondary: '#7928ca',
  text: '#333333',
  background: '#ffffff'
};

const darkTheme: Theme = {
  primary: '#3694ff',
  secondary: '#9e50e9',
  text: '#f0f0f0',
  background: '#121212'
};

interface ThemeContextType {
  theme: Theme;
  isDark: boolean;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [isDark, setIsDark] = useState(false);
  const theme = isDark ? darkTheme : lightTheme;
  
  const toggleTheme = () => setIsDark(prev => !prev);
  
  return (
    <ThemeContext.Provider value={{ theme, isDark, toggleTheme }}>
      <div 
        style={{ 
          background: theme.background, 
          color: theme.text,
          transition: 'background 0.3s, color 0.3s' 
        }}
      >
        {children}
      </div>
    </ThemeContext.Provider>
  );
}

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
};

// Usage
function Button({ children }: { children: ReactNode }) {
  const { theme } = useTheme();
  
  return (
    <button 
      style={{ 
        background: theme.primary,
        color: 'white',
        border: 'none',
        padding: '8px 16px',
        borderRadius: '4px'
      }}
    >
      {children}
    </button>
  );
}
```

## Data Display Patterns

CRM Dashboard UI Kit mein multiple data display patterns hai jo various metrics aur information ko show karne mein help karte hain.

### Dashboard Cards Grid:

```tsx
function StatCardsGrid({ stats }: { stats: StatItem[] }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      {stats.map(stat => (
        <StatCard 
          key={stat.id}
          title={stat.title}
          value={stat.value}
          change={stat.change}
          changeType={stat.change > 0 ? 'positive' : 'negative'}
          icon={stat.icon}
        />
      ))}
    </div>
  );
}

interface StatCardProps {
  title: string;
  value: string | number;
  change?: number;
  changeType?: 'positive' | 'negative' | 'neutral';
  icon?: React.ReactNode;
}

function StatCard({ title, value, change, changeType = 'neutral', icon }: StatCardProps) {
  const changeColor = {
    positive: 'text-green-500',
    negative: 'text-red-500',
    neutral: 'text-gray-500'
  };
  
  return (
    <div className="bg-white rounded-lg shadow p-4">
      <div className="flex items-center justify-between">
        <div>
          <p className="text-gray-500 text-sm">{title}</p>
          <h3 className="text-2xl font-bold mt-1">{value}</h3>
          
          {change !== undefined && (
            <p className={`text-sm mt-2 ${changeColor[changeType]}`}>
              {change > 0 ? '+' : ''}{change}%
            </p>
          )}
        </div>
        
        {icon && (
          <div className="bg-blue-50 p-3 rounded-full">
            {icon}
          </div>
        )}
      </div>
    </div>
  );
}
```

## List and Tables Patterns

Dashboard applications mein data ko list ya table form mein display karna common hai.

### Sortable Table Pattern:

```tsx
import { useState } from 'react';

interface TableColumn<T> {
  key: keyof T | 'actions';
  title: string;
  sortable?: boolean;
  render?: (item: T) => React.ReactNode;
}

interface SortableTableProps<T> {
  data: T[];
  columns: TableColumn<T>[];
  defaultSortKey?: keyof T;
}

type SortDirection = 'asc' | 'desc';

function SortableTable<T extends object>({ 
  data, 
  columns, 
  defaultSortKey 
}: SortableTableProps<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(defaultSortKey || null);
  const [sortDirection, setSortDirection] = useState<SortDirection>('asc');
  
  const handleSort = (key: keyof T) => {
    if (sortKey === key) {
      // Toggle direction if already sorting by this key
      setSortDirection(prev => prev === 'asc' ? 'desc' : 'asc');
    } else {
      // Set new sort key and default to ascending
      setSortKey(key);
      setSortDirection('asc');
    }
  };
  
  // Sort the data
  const sortedData = [...data];
  if (sortKey) {
    sortedData.sort((a, b) => {
      const valueA = a[sortKey];
      const valueB = b[sortKey];
      
      if (valueA === valueB) return 0;
      
      // Handle different data types (string, number, date, etc.)
      const result = valueA < valueB ? -1 : 1;
      
      // Reverse if descending
      return sortDirection === 'asc' ? result : -result;
    });
  }
  
  return (
    <div className="overflow-x-auto">
      <table className="min-w-full divide-y divide-gray-200">
        <thead className="bg-gray-50">
          <tr>
            {columns.map(column => (
              <th 
                key={column.key.toString()} 
                className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
              >
                {column.sortable && column.key !== 'actions' ? (
                  <button 
                    className="flex items-center space-x-1"
                    onClick={() => handleSort(column.key as keyof T)}
                  >
                    <span>{column.title}</span>
                    {sortKey === column.key && (
                      <span>
                        {sortDirection === 'asc' ? '↑' : '↓'}
                      </span>
                    )}
                  </button>
                ) : (
                  column.title
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody className="bg-white divide-y divide-gray-200">
          {sortedData.map((item, index) => (
            <tr key={index}>
              {columns.map(column => (
                <td 
                  key={`${index}-${column.key.toString()}`}
                  className="px-6 py-4 whitespace-nowrap"
                >
                  {column.render 
                    ? column.render(item) 
                    : column.key !== 'actions' 
                      ? String(item[column.key]) 
                      : null
                  }
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

// Usage example
interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  lastActive: Date;
}

function UsersTable({ users }: { users: User[] }) {
  const columns: TableColumn<User>[] = [
    { key: 'name', title: 'Name', sortable: true },
    { key: 'email', title: 'Email' },
    { key: 'role', title: 'Role', sortable: true },
    { 
      key: 'lastActive', 
      title: 'Last Active', 
      sortable: true,
      render: (user) => user.lastActive.toLocaleDateString() 
    },
    {
      key: 'actions',
      title: 'Actions',
      render: (user) => (
        <div className="flex space-x-2">
          <button className="text-blue-500">Edit</button>
          <button className="text-red-500">Delete</button>
        </div>
      )
    }
  ];
  
  return <SortableTable data={users} columns={columns} defaultSortKey="name" />;
}
```

## Conclusion

Ye modern UI design patterns CRM Dashboard UI Kit se extract kiye gaye hain. In patterns ko samajhne aur implement karne se aap modern, user-friendly, aur maintainable React applications bana sakte hain. 