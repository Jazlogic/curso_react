# 🟢 Junior Level 3: Routing y Estado Local

## 🧭 Navegación del Curso

**← Anterior**: [Junior Level 2: Componentes y Hooks](../junior_2/README.md)  
**Siguiente →**: [Mid Level 1: Estado Global](../midLevel_1/README.md)

---

# 🚀 Módulo 3: Routing y Estado Local

## 📚 Descripción del Módulo

En este módulo aprenderás a implementar navegación entre páginas usando React Router, manejar formularios complejos con validación avanzada, y estructurar aplicaciones multi-página. También explorarás técnicas para manejar estado local más sofisticado y crear experiencias de usuario fluidas.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Configurar y usar React Router para navegación
- ✅ Implementar rutas dinámicas y parámetros de URL
- ✅ Crear layouts y navegación anidada
- ✅ Manejar formularios complejos con validación avanzada
- ✅ Implementar navegación programática
- ✅ Crear páginas protegidas y autenticación básica
- ✅ Manejar estado local complejo con múltiples hooks
- ✅ Implementar búsqueda y filtrado en tiempo real
- ✅ Crear componentes de navegación reutilizables
- ✅ Estructurar aplicaciones multi-página de manera escalable

## 📖 Contenido Teórico

### 1. React Router - Navegación Básica

React Router es la biblioteca estándar para implementar navegación en aplicaciones React.

#### Instalación y configuración básica:
```bash
npm install react-router-dom
```

#### Configuración básica del router:
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

#### Componente de navegación:
```jsx
import { Link, NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <ul>
        <li>
          <Link to="/">Inicio</Link>
        </li>
        <li>
          <NavLink 
            to="/about"
            className={({ isActive }) => isActive ? 'active' : ''}
          >
            Acerca de
          </NavLink>
        </li>
        <li>
          <NavLink 
            to="/contact"
            className={({ isActive }) => isActive ? 'active' : ''}
          >
            Contacto
          </NavLink>
        </li>
      </ul>
    </nav>
  );
}
```

### 2. Rutas Dinámicas y Parámetros

#### Rutas con parámetros:
```jsx
import { useParams, useSearchParams } from 'react-router-dom';

function UserDetail() {
  const { id } = useParams();
  const [searchParams, setSearchParams] = useSearchParams();
  const [usuario, setUsuario] = useState(null);

  useEffect(() => {
    if (id) {
      fetchUsuario(id);
    }
  }, [id]);

  const handleFilterChange = (filter) => {
    setSearchParams({ filter });
  };

  return (
    <div>
      <h2>Usuario {id}</h2>
      {usuario && (
        <div>
          <h3>{usuario.name}</h3>
          <p>Email: {usuario.email}</p>
          <p>Filtro actual: {searchParams.get('filter')}</p>
          <button onClick={() => handleFilterChange('posts')}>
            Ver Posts
          </button>
        </div>
      )}
    </div>
  );
}
```

#### Rutas anidadas:
```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="dashboard" element={<Dashboard />}>
            <Route index element={<DashboardOverview />} />
            <Route path="profile" element={<Profile />} />
            <Route path="settings" element={<Settings />} />
          </Route>
          <Route path="users" element={<Users />}>
            <Route index element={<UserList />} />
            <Route path=":id" element={<UserDetail />} />
            <Route path="new" element={<NewUser />} />
          </Route>
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

function Layout() {
  return (
    <div>
      <Header />
      <main>
        <Outlet />
      </main>
      <Footer />
    </div>
  );
}
```

### 3. Navegación Programática

#### Navegación con useNavigate:
```jsx
import { useNavigate, useLocation } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  const location = useLocation();
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const response = await loginUser(formData);
      if (response.success) {
        // Navegar a la página anterior o al dashboard
        const from = location.state?.from?.pathname || '/dashboard';
        navigate(from, { replace: true });
      }
    } catch (error) {
      console.error('Error de login:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
        placeholder="Email"
        required
      />
      <input
        type="password"
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
        placeholder="Contraseña"
        required
      />
      <button type="submit">Iniciar sesión</button>
    </form>
  );
}
```

#### Navegación con confirmación:
```jsx
function EditUserForm({ userId }) {
  const navigate = useNavigate();
  const [hasChanges, setHasChanges] = useState(false);

  const handleChange = () => {
    setHasChanges(true);
  };

  const handleCancel = () => {
    if (hasChanges) {
      const confirmar = window.confirm('¿Estás seguro de que quieres salir sin guardar?');
      if (confirmar) {
        navigate('/users');
      }
    } else {
      navigate('/users');
    }
  };

  return (
    <form onChange={handleChange}>
      {/* Campos del formulario */}
      <button type="button" onClick={handleCancel}>
        Cancelar
      </button>
      <button type="submit">Guardar</button>
    </form>
  );
}
```

### 4. Formularios Avanzados con Validación

#### Hook personalizado para validación:
```jsx
function useFormValidation(initialValues, validationRules) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const validateField = (name, value) => {
    const rules = validationRules[name];
    if (!rules) return '';

    for (const rule of rules) {
      if (rule.required && !value) {
        return rule.message || `${name} es requerido`;
      }
      if (rule.minLength && value.length < rule.minLength) {
        return rule.message || `${name} debe tener al menos ${rule.minLength} caracteres`;
      }
      if (rule.pattern && !rule.pattern.test(value)) {
        return rule.message || `${name} no tiene el formato correcto`;
      }
      if (rule.custom) {
        const customError = rule.custom(value, values);
        if (customError) return customError;
      }
    }
    return '';
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    if (touched[name]) {
      const error = validateField(name, value);
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  };

  const handleBlur = (e) => {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error }));
  };

  const validateForm = () => {
    const newErrors = {};
    Object.keys(values).forEach(name => {
      newErrors[name] = validateField(name, values[name]);
    });
    setErrors(newErrors);
    return !Object.values(newErrors).some(error => error);
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateForm,
    reset,
    setValues
  };
}

// Uso del hook de validación
const validationRules = {
  email: [
    { required: true, message: 'El email es requerido' },
    { pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: 'Email inválido' }
  ],
  password: [
    { required: true, message: 'La contraseña es requerida' },
    { minLength: 8, message: 'La contraseña debe tener al menos 8 caracteres' },
    { 
      custom: (value) => {
        if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          return 'La contraseña debe contener mayúsculas, minúsculas y números';
        }
        return '';
      }
    }
  ],
  confirmPassword: [
    { required: true, message: 'Confirma tu contraseña' },
    { 
      custom: (value, values) => {
        if (value !== values.password) {
          return 'Las contraseñas no coinciden';
        }
        return '';
      }
    }
  ]
};

function RegistrationForm() {
  const { values, errors, touched, handleChange, handleBlur, validateForm } = 
    useFormValidation({
      email: '',
      password: '',
      confirmPassword: ''
    }, validationRules);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateForm()) {
      console.log('Formulario válido:', values);
      // Enviar datos
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
          className={touched.email && errors.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Contraseña"
          className={touched.password && errors.password ? 'error' : ''}
        />
        {touched.password && errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          value={values.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Confirmar contraseña"
          className={touched.confirmPassword && errors.confirmPassword ? 'error' : ''}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit">Registrarse</button>
    </form>
  );
}
```

### 5. Búsqueda y Filtrado en Tiempo Real

#### Hook personalizado para búsqueda:
```jsx
function useSearch(initialData, searchFields) {
  const [data, setData] = useState(initialData);
  const [searchTerm, setSearchTerm] = useState('');
  const [filteredData, setFilteredData] = useState(initialData);

  useEffect(() => {
    if (!searchTerm.trim()) {
      setFilteredData(data);
      return;
    }

    const filtered = data.filter(item =>
      searchFields.some(field => {
        const value = item[field];
        if (typeof value === 'string') {
          return value.toLowerCase().includes(searchTerm.toLowerCase());
        }
        if (typeof value === 'number') {
          return value.toString().includes(searchTerm);
        }
        return false;
      })
    );

    setFilteredData(filtered);
  }, [searchTerm, data, searchFields]);

  const updateData = (newData) => {
    setData(newData);
  };

  const clearSearch = () => {
    setSearchTerm('');
  };

  return {
    searchTerm,
    setSearchTerm,
    filteredData,
    updateData,
    clearSearch
  };
}

// Componente de búsqueda reutilizable
function SearchBar({ onSearch, placeholder = "Buscar..." }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedSearchTerm, setDebouncedSearchTerm] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedSearchTerm(searchTerm);
    }, 300);

    return () => clearTimeout(timer);
  }, [searchTerm]);

  useEffect(() => {
    onSearch(debouncedSearchTerm);
  }, [debouncedSearchTerm, onSearch]);

  return (
    <div className="search-bar">
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder={placeholder}
        className="search-input"
      />
      {searchTerm && (
        <button 
          onClick={() => setSearchTerm('')}
          className="clear-search"
        >
          ×
        </button>
      )}
    </div>
  );
}

// Uso en un componente
function UserList() {
  const { searchTerm, setSearchTerm, filteredData, updateData } = useSearch(
    users, // datos iniciales
    ['name', 'email', 'role'] // campos para buscar
  );

  const handleSearch = (term) => {
    setSearchTerm(term);
  };

  return (
    <div>
      <SearchBar onSearch={handleSearch} placeholder="Buscar usuarios..." />
      
      <div className="user-list">
        {filteredData.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
      </div>
    </div>
  );
}
```

### 6. Páginas Protegidas y Autenticación

#### Hook para autenticación:
```jsx
function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();

  useEffect(() => {
    // Verificar si hay un usuario en localStorage
    const savedUser = localStorage.getItem('user');
    if (savedUser) {
      setUser(JSON.parse(savedUser));
    }
    setLoading(false);
  }, []);

  const login = async (credentials) => {
    try {
      const response = await loginAPI(credentials);
      if (response.success) {
        setUser(response.user);
        localStorage.setItem('user', JSON.stringify(response.user));
        return { success: true };
      }
      return { success: false, error: response.message };
    } catch (error) {
      return { success: false, error: 'Error de conexión' };
    }
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
    navigate('/login');
  };

  const isAuthenticated = !!user;

  return {
    user,
    loading,
    login,
    logout,
    isAuthenticated
  };
}

// Componente de ruta protegida
function ProtectedRoute({ children, redirectTo = '/login' }) {
  const { isAuthenticated, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return <div>Cargando...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to={redirectTo} state={{ from: location }} replace />;
  }

  return children;
}

// Uso en el router
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/" element={
          <ProtectedRoute>
            <Layout />
          </ProtectedRoute>
        }>
          <Route index element={<Home />} />
          <Route path="dashboard" element={<Dashboard />} />
          <Route path="profile" element={<Profile />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 7. Layouts y Navegación Anidada

#### Layout principal con navegación:
```jsx
function Layout() {
  const { user, logout } = useAuth();
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="layout">
      <Header 
        user={user} 
        onLogout={logout}
        onMenuClick={() => setSidebarOpen(!sidebarOpen)}
      />
      
      <div className="main-container">
        <Sidebar 
          isOpen={sidebarOpen} 
          onClose={() => setSidebarOpen(false)}
        />
        
        <main className="main-content">
          <Outlet />
        </main>
      </div>
    </div>
  );
}

function Header({ user, onLogout, onMenuClick }) {
  return (
    <header className="header">
      <button className="menu-button" onClick={onMenuClick}>
        ☰
      </button>
      
      <div className="header-content">
        <h1>Mi Aplicación</h1>
        
        <div className="user-menu">
          <span>Hola, {user.name}</span>
          <button onClick={onLogout}>Cerrar sesión</button>
        </div>
      </div>
    </header>
  );
}

function Sidebar({ isOpen, onClose }) {
  const location = useLocation();

  const menuItems = [
    { path: '/', label: 'Inicio', icon: '🏠' },
    { path: '/dashboard', label: 'Dashboard', icon: '📊' },
    { path: '/users', label: 'Usuarios', icon: '👥' },
    { path: '/settings', label: 'Configuración', icon: '⚙️' }
  ];

  return (
    <>
      {isOpen && (
        <div className="sidebar-overlay" onClick={onClose} />
      )}
      
      <aside className={`sidebar ${isOpen ? 'open' : ''}`}>
        <nav className="sidebar-nav">
          {menuItems.map(item => (
            <NavLink
              key={item.path}
              to={item.path}
              className={({ isActive }) => 
                `sidebar-item ${isActive ? 'active' : ''}`
              }
              onClick={onClose}
            >
              <span className="icon">{item.icon}</span>
              <span className="label">{item.label}</span>
            </NavLink>
          ))}
        </nav>
      </aside>
    </>
  );
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Navegación Básica**
Crea una aplicación con 3 páginas (Inicio, Acerca de, Contacto) y navegación entre ellas usando React Router.

### **Ejercicio 2: Formulario de Registro con Validación**
Crea un formulario de registro completo con validación en tiempo real para email, contraseña y confirmación.

### **Ejercicio 3: Lista de Usuarios con Búsqueda**
Crea una lista de usuarios con búsqueda en tiempo real y navegación a detalles de usuario.

### **Ejercicio 4: Dashboard con Rutas Anidadas**
Crea un dashboard con navegación anidada (Overview, Profile, Settings) usando rutas anidadas.

### **Ejercicio 5: Formulario de Edición con Confirmación**
Crea un formulario de edición que pida confirmación antes de salir si hay cambios sin guardar.

### **Ejercicio 6: Página de Login con Redirección**
Crea una página de login que redirija al usuario a la página que intentaba acceder originalmente.

### **Ejercicio 7: Filtros Avanzados**
Crea un sistema de filtros múltiples (por categoría, precio, fecha) para una lista de productos.

### **Ejercicio 8: Navegación con Breadcrumbs**
Implementa breadcrumbs que muestren la ruta actual y permitan navegación rápida.

### **Ejercicio 9: Formulario Multi-paso**
Crea un formulario de registro en múltiples pasos con navegación entre pasos y validación.

### **Ejercicio 10: Layout Responsive**
Crea un layout que se adapte a diferentes tamaños de pantalla con navegación móvil.

## 🎯 Proyecto Integrador: Blog Personal

### **Descripción del Proyecto**
Construye un blog personal completo con múltiples páginas, sistema de navegación, formularios de contacto, gestión de posts, y funcionalidades de búsqueda y filtrado.

### **Requisitos del Proyecto**
- ✅ Páginas: Inicio, Blog, Acerca de, Contacto, Admin
- ✅ Sistema de navegación completo con React Router
- ✅ Formulario de contacto con validación
- ✅ Lista de posts con búsqueda y filtros
- ✅ Páginas de detalle para cada post
- ✅ Panel de administración protegido
- ✅ Formulario para crear/editar posts
- ✅ Layout responsive y navegación móvil
- ✅ Breadcrumbs para navegación
- ✅ Sistema de categorías y tags

### **Estructura de Componentes Sugerida**
```
App/
├── Layout/
│   ├── Header/
│   ├── Navigation/
│   ├── Sidebar/
│   └── Footer/
├── Pages/
│   ├── Home/
│   ├── Blog/
│   ├── PostDetail/
│   ├── About/
│   ├── Contact/
│   └── Admin/
├── Components/
│   ├── SearchBar/
│   ├── PostCard/
│   ├── ContactForm/
│   ├── PostForm/
│   └── Breadcrumbs/
└── Hooks/
    ├── useAuth/
    ├── useSearch/
    └── useFormValidation/
```

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cómo se configura React Router en una aplicación?
2. ¿Cuál es la diferencia entre Link y NavLink?
3. ¿Cómo se implementan rutas dinámicas con parámetros?
4. ¿Qué es useNavigate y cuándo se usa?
5. ¿Cómo se implementa validación de formularios en tiempo real?
6. ¿Qué son las rutas anidadas y cómo se implementan?
7. ¿Cómo se crean páginas protegidas con autenticación?

### **Criterios de Evaluación**
- ✅ Configurar React Router correctamente
- ✅ Implementar navegación entre páginas
- ✅ Crear rutas dinámicas y anidadas
- ✅ Manejar formularios complejos con validación
- ✅ Implementar búsqueda y filtrado en tiempo real
- ✅ Crear páginas protegidas y autenticación
- ✅ Estructurar aplicaciones multi-página
- ✅ Implementar navegación programática

## 🔑 Conceptos Clave a Recordar

- **React Router** es la biblioteca estándar para navegación
- **useParams** permite acceder a parámetros de URL
- **useNavigate** permite navegación programática
- **useSearchParams** maneja parámetros de consulta
- **Outlet** renderiza rutas anidadas
- **ProtectedRoute** protege páginas que requieren autenticación
- **Validación en tiempo real** mejora la experiencia del usuario
- **Búsqueda con debouncing** optimiza el rendimiento

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre estado global con Context API, Redux Toolkit, y cómo manejar estado complejo en aplicaciones grandes.

## 📚 Recursos Adicionales

- [React Router Documentation](https://reactrouter.com/)
- [React Router Tutorial](https://reactrouter.com/docs/en/v6/getting-started/tutorial)
- [Form Validation in React](https://react.dev/learn/forms)
- [Protected Routes](https://reactrouter.com/docs/en/v6/examples/auth)
- [Nested Routes](https://reactrouter.com/docs/en/v6/getting-started/concepts)

## ✅ Checklist de Competencias

- [ ] Configuro y uso React Router para navegación
- [ ] Implemento rutas dinámicas y parámetros de URL
- [ ] Creo layouts y navegación anidada
- [ ] Manejo formularios complejos con validación avanzada
- [ ] Implemento navegación programática
- [ ] Creo páginas protegidas y autenticación básica
- [ ] Manejo estado local complejo con múltiples hooks
- [ ] Implemento búsqueda y filtrado en tiempo real
- [ ] Creo componentes de navegación reutilizables
- [ ] Completo el proyecto integrador del blog personal

---

**¡Excelente! Has dominado la navegación y formularios. Estás listo para el siguiente nivel.** 🎉
