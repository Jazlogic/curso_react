# 🚀 Módulo 4: Estado Global y Context API

## 📚 Descripción del Módulo

En este módulo aprenderás a manejar estado global en aplicaciones React usando Context API y Redux Toolkit. Explorarás técnicas para compartir estado entre componentes distantes, implementar patrones de gestión de estado, y crear aplicaciones escalables con arquitectura de estado bien definida.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Entender cuándo usar estado global vs estado local
- ✅ Implementar Context API para estado compartido
- ✅ Configurar y usar Redux Toolkit para estado complejo
- ✅ Crear stores y slices de Redux
- ✅ Implementar middleware personalizado
- ✅ Manejar estado asíncrono con Redux Thunk
- ✅ Crear selectores optimizados con Reselect
- ✅ Implementar persistencia de estado
- ✅ Crear hooks personalizados para estado global
- ✅ Diseñar arquitectura de estado escalable

## 📖 Contenido Teórico

### 1. ¿Cuándo Usar Estado Global?

#### Estado Local vs Estado Global:
```jsx
// Estado Local - Solo para el componente
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Contador: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

// Estado Global - Compartido entre componentes
// Ejemplo: Usuario autenticado, tema de la aplicación, carrito de compras
```

#### Casos de uso para estado global:
- **Autenticación**: Usuario logueado, tokens, permisos
- **Tema**: Modo claro/oscuro, colores, idioma
- **Carrito de compras**: Productos, cantidades, total
- **Configuración**: Preferencias del usuario, ajustes de la app
- **Datos compartidos**: Lista de usuarios, productos, posts

### 2. Context API - Estado Compartido Simple

#### Crear un Context:
```jsx
import { createContext, useContext, useReducer } from 'react';

// 1. Definir el estado inicial
const initialState = {
  user: null,
  isAuthenticated: false,
  theme: 'light'
};

// 2. Definir los tipos de acciones
const actionTypes = {
  LOGIN: 'LOGIN',
  LOGOUT: 'LOGOUT',
  TOGGLE_THEME: 'TOGGLE_THEME'
};

// 3. Crear el reducer
function authReducer(state, action) {
  switch (action.type) {
    case actionTypes.LOGIN:
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true
      };
    case actionTypes.LOGOUT:
      return {
        ...state,
        user: null,
        isAuthenticated: false
      };
    case actionTypes.TOGGLE_THEME:
      return {
        ...state,
        theme: state.theme === 'light' ? 'dark' : 'light'
      };
    default:
      return state;
  }
}

// 4. Crear el Context
const AuthContext = createContext();

// 5. Crear el Provider
function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, initialState);

  const login = (userData) => {
    dispatch({ type: actionTypes.LOGIN, payload: userData });
  };

  const logout = () => {
    dispatch({ type: actionTypes.LOGOUT });
  };

  const toggleTheme = () => {
    dispatch({ type: actionTypes.TOGGLE_THEME });
  };

  const value = {
    ...state,
    login,
    logout,
    toggleTheme
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// 6. Crear hook personalizado para usar el context
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth debe usarse dentro de AuthProvider');
  }
  return context;
}

// 7. Envolver la aplicación
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/" element={<Layout />}>
            <Route index element={<Home />} />
            <Route path="/profile" element={<Profile />} />
            <Route path="/settings" element={<Settings />} />
          </Route>
        </Routes>
      </Router>
    </AuthProvider>
  );
}
```

#### Usar el Context en componentes:
```jsx
function Header() {
  const { user, isAuthenticated, logout, theme, toggleTheme } = useAuth();

  return (
    <header className={`header ${theme}`}>
      <h1>Mi Aplicación</h1>
      
      <div className="header-actions">
        <button onClick={toggleTheme}>
          {theme === 'light' ? '🌙' : '☀️'}
        </button>
        
        {isAuthenticated ? (
          <div className="user-menu">
            <span>Hola, {user.name}</span>
            <button onClick={logout}>Cerrar sesión</button>
          </div>
        ) : (
          <Link to="/login">Iniciar sesión</Link>
        )}
      </div>
    </header>
  );
}

function Profile() {
  const { user, isAuthenticated } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isAuthenticated) {
      navigate('/login');
    }
  }, [isAuthenticated, navigate]);

  if (!isAuthenticated) return null;

  return (
    <div className="profile">
      <h2>Perfil de {user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Rol: {user.role}</p>
    </div>
  );
}
```

### 3. Context API Avanzado - Múltiples Contexts

#### Separar concerns en múltiples contexts:
```jsx
// Theme Context
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [accentColor, setAccentColor] = useState('#007bff');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const changeAccentColor = (color) => {
    setAccentColor(color);
  };

  const value = {
    theme,
    accentColor,
    toggleTheme,
    changeAccentColor
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Cart Context
const CartContext = createContext();

function CartProvider({ children }) {
  const [items, setItems] = useState([]);
  const [isOpen, setIsOpen] = useState(false);

  const addItem = (product, quantity = 1) => {
    setItems(prev => {
      const existingItem = prev.find(item => item.id === product.id);
      if (existingItem) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + quantity }
            : item
        );
      }
      return [...prev, { ...product, quantity }];
    });
  };

  const removeItem = (productId) => {
    setItems(prev => prev.filter(item => item.id !== productId));
  };

  const updateQuantity = (productId, quantity) => {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    setItems(prev =>
      prev.map(item =>
        item.id === productId ? { ...item, quantity } : item
      )
    );
  };

  const clearCart = () => {
    setItems([]);
  };

  const total = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  const itemCount = items.reduce((sum, item) => sum + item.quantity, 0);

  const value = {
    items,
    isOpen,
    total,
    itemCount,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    setIsOpen
  };

  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// Combinar providers
function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          <Router>
            <Routes>
              <Route path="/" element={<Layout />}>
                <Route index element={<Home />} />
                <Route path="/products" element={<Products />} />
                <Route path="/cart" element={<Cart />} />
              </Route>
            </Routes>
          </Router>
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

### 4. Redux Toolkit - Estado Global Complejo

#### Instalación y configuración:
```bash
npm install @reduxjs/toolkit react-redux
```

#### Configurar el store:
```jsx
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import cartReducer from './slices/cartSlice';
import productsReducer from './slices/productsSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer,
    products: productsReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    })
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

#### Crear slices:
```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk para login
export const loginUser = createAsyncThunk(
  'auth/loginUser',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }
      
      return await response.json();
    } catch (error) {
      return rejectWithValue('Error de conexión');
    }
  }
);

// Auth slice
const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    isAuthenticated: false,
    loading: false,
    error: null
  },
  reducers: {
    logout: (state) => {
      state.user = null;
      state.isAuthenticated = false;
      state.error = null;
    },
    clearError: (state) => {
      state.error = null;
    },
    updateProfile: (state, action) => {
      state.user = { ...state.user, ...action.payload };
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.isAuthenticated = true;
        state.error = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { logout, clearError, updateProfile } = authSlice.actions;
export default authSlice.reducer;
```

#### Cart slice con lógica compleja:
```jsx
import { createSlice, createSelector } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    isOpen: false
  },
  reducers: {
    addItem: (state, action) => {
      const { product, quantity = 1 } = action.payload;
      const existingItem = state.items.find(item => item.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        state.items.push({ ...product, quantity });
      }
    },
    removeItem: (state, action) => {
      const productId = action.payload;
      state.items = state.items.filter(item => item.id !== productId);
    },
    updateQuantity: (state, action) => {
      const { productId, quantity } = action.payload;
      const item = state.items.find(item => item.id === productId);
      
      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter(item => item.id !== productId);
        } else {
          item.quantity = quantity;
        }
      }
    },
    clearCart: (state) => {
      state.items = [];
    },
    toggleCart: (state) => {
      state.isOpen = !state.isOpen;
    }
  }
});

// Selectores optimizados
export const selectCartItems = (state) => state.cart.items;
export const selectCartIsOpen = (state) => state.cart.isOpen;

export const selectCartTotal = createSelector(
  [selectCartItems],
  (items) => items.reduce((sum, item) => sum + (item.price * item.quantity), 0)
);

export const selectCartItemCount = createSelector(
  [selectCartItems],
  (items) => items.reduce((sum, item) => sum + item.quantity, 0)
);

export const selectCartItemById = createSelector(
  [selectCartItems, (state, productId) => productId],
  (items, productId) => items.find(item => item.id === productId)
);

export const { addItem, removeItem, updateQuantity, clearCart, toggleCart } = cartSlice.actions;
export default cartSlice.reducer;
```

### 5. Usar Redux en Componentes

#### Hooks de Redux:
```jsx
import { useSelector, useDispatch } from 'react-redux';
import { 
  selectCartItems, 
  selectCartTotal, 
  selectCartItemCount,
  addItem, 
  removeItem, 
  toggleCart 
} from '../store/slices/cartSlice';

function CartIcon() {
  const dispatch = useDispatch();
  const itemCount = useSelector(selectCartItemCount);
  const isOpen = useSelector(selectCartIsOpen);

  return (
    <div className="cart-icon" onClick={() => dispatch(toggleCart())}>
      🛒 <span className="cart-count">{itemCount}</span>
    </div>
  );
}

function ProductCard({ product }) {
  const dispatch = useDispatch();
  const cartItem = useSelector(state => selectCartItemById(state, product.id));

  const handleAddToCart = () => {
    dispatch(addItem({ product, quantity: 1 }));
  };

  const handleRemoveFromCart = () => {
    dispatch(removeItem(product.id));
  };

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      
      {cartItem ? (
        <div className="cart-controls">
          <button onClick={() => dispatch(updateQuantity({ 
            productId: product.id, 
            quantity: cartItem.quantity - 1 
          }))}>
            -
          </button>
          <span>{cartItem.quantity}</span>
          <button onClick={() => dispatch(updateQuantity({ 
            productId: product.id, 
            quantity: cartItem.quantity + 1 
          }))}>
            +
          </button>
          <button onClick={handleRemoveFromCart}>Eliminar</button>
        </div>
      ) : (
        <button onClick={handleAddToCart}>Agregar al carrito</button>
      )}
    </div>
  );
}

function Cart() {
  const dispatch = useDispatch();
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);
  const isOpen = useSelector(selectCartIsOpen);

  if (!isOpen) return null;

  return (
    <div className="cart-overlay">
      <div className="cart">
        <div className="cart-header">
          <h2>Carrito de Compras</h2>
          <button onClick={() => dispatch(toggleCart())}>×</button>
        </div>
        
        {items.length === 0 ? (
          <p>Tu carrito está vacío</p>
        ) : (
          <>
            <div className="cart-items">
              {items.map(item => (
                <div key={item.id} className="cart-item">
                  <img src={item.image} alt={item.name} />
                  <div className="item-details">
                    <h4>{item.name}</h4>
                    <p>${item.price} x {item.quantity}</p>
                  </div>
                  <button onClick={() => dispatch(removeItem(item.id))}>
                    Eliminar
                  </button>
                </div>
              ))}
            </div>
            
            <div className="cart-footer">
              <div className="cart-total">
                <strong>Total: ${total}</strong>
              </div>
              <button onClick={() => dispatch(clearCart())}>
                Vaciar carrito
              </button>
              <button className="checkout-btn">Proceder al checkout</button>
            </div>
          </>
        )}
      </div>
    </div>
  );
}
```

### 6. Middleware Personalizado

#### Logger middleware:
```jsx
import { createListenerMiddleware } from '@reduxjs/toolkit';

export const listenerMiddleware = createListenerMiddleware();

listenerMiddleware.startListening({
  predicate: (action, currentState, previousState) => {
    // Solo loggear acciones que cambien el estado
    return currentState !== previousState;
  },
  effect: (action, listenerApi) => {
    console.group('Redux Action');
    console.log('Action:', action);
    console.log('Previous State:', listenerApi.getState());
    console.log('Current State:', listenerApi.getState());
    console.groupEnd();
  }
});

// Agregar al store
export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer,
    products: productsReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .prepend(listenerMiddleware.middleware)
});
```

#### Persistencia con Redux Persist:
```jsx
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';

const persistConfig = {
  key: 'root',
  storage,
  whitelist: ['auth', 'cart'] // Solo persistir estos reducers
};

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    })
});

export const persistor = persistStore(store);

// En el componente raíz
import { PersistGate } from 'redux-persist/integration/react';

function App() {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <Router>
          <Routes>
            <Route path="/" element={<Layout />}>
              <Route index element={<Home />} />
              <Route path="/products" element={<Products />} />
              <Route path="/cart" element={<Cart />} />
            </Route>
          </Routes>
        </Router>
      </PersistGate>
    </Provider>
  );
}
```

### 7. Hooks Personalizados para Estado Global

#### Hook para Redux:
```jsx
import { useSelector, useDispatch } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (state: RootState) => T) => 
  useSelector<RootState, T>(selector);

// Hook personalizado para productos
export function useProducts() {
  const dispatch = useAppDispatch();
  const products = useAppSelector(state => state.products.items);
  const loading = useAppSelector(state => state.products.loading);
  const error = useAppSelector(state => state.products.error);

  const fetchProducts = () => dispatch(fetchProductsAsync());
  const addProduct = (product) => dispatch(addProductAsync(product));
  const updateProduct = (id, updates) => dispatch(updateProductAsync({ id, updates }));
  const deleteProduct = (id) => dispatch(deleteProductAsync(id));

  return {
    products,
    loading,
    error,
    fetchProducts,
    addProduct,
    updateProduct,
    deleteProduct
  };
}

// Hook personalizado para carrito
export function useCart() {
  const dispatch = useAppDispatch();
  const items = useAppSelector(selectCartItems);
  const total = useAppSelector(selectCartTotal);
  const itemCount = useAppSelector(selectCartItemCount);
  const isOpen = useAppSelector(selectCartIsOpen);

  const addItem = (product, quantity = 1) => 
    dispatch(addItem({ product, quantity }));
  const removeItem = (productId) => dispatch(removeItem(productId));
  const updateQuantity = (productId, quantity) => 
    dispatch(updateQuantity({ productId, quantity }));
  const clearCart = () => dispatch(clearCart());
  const toggleCart = () => dispatch(toggleCart());

  return {
    items,
    total,
    itemCount,
    isOpen,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    toggleCart
  };
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Context API para Tema**
Crea un sistema de tema usando Context API que permita cambiar entre modo claro/oscuro y colores de acento.

### **Ejercicio 2: Carrito de Compras con Context**
Implementa un carrito de compras completo usando Context API con funcionalidades de agregar, eliminar y actualizar cantidades.

### **Ejercicio 3: Redux Store Básico**
Crea un store de Redux con slices para usuarios, posts y comentarios, implementando CRUD completo.

### **Ejercicio 4: Async Actions con Redux Thunk**
Implementa acciones asíncronas para cargar datos de una API, con estados de loading y error.

### **Ejercicio 5: Selectores Optimizados**
Crea selectores optimizados con Reselect para filtrar y ordenar listas de productos.

### **Ejercicio 6: Middleware de Logging**
Implementa un middleware personalizado que loggee todas las acciones y cambios de estado.

### **Ejercicio 7: Persistencia de Estado**
Configura Redux Persist para guardar el estado del carrito y preferencias del usuario.

### **Ejercicio 8: Hook Personalizado para Redux**
Crea hooks personalizados que encapsulen la lógica de Redux para diferentes entidades.

### **Ejercicio 9: Estado Global vs Local**
Analiza una aplicación y decide qué estado debe ser global vs local, implementando ambos enfoques.

### **Ejercicio 10: Migración de Context a Redux**
Toma una aplicación que use Context API y migra el estado complejo a Redux Toolkit.

## 🎯 Proyecto Integrador: Gestor de Proyectos

### **Descripción del Proyecto**
Construye una aplicación de gestión de proyectos que incluya gestión de usuarios, proyectos, tareas, y un dashboard con métricas en tiempo real.

### **Requisitos del Proyecto**
- ✅ Gestión de usuarios con roles y permisos
- ✅ CRUD completo de proyectos y tareas
- ✅ Sistema de asignación de tareas
- ✅ Dashboard con métricas y gráficos
- ✅ Filtros y búsqueda avanzada
- ✅ Notificaciones en tiempo real
- ✅ Sistema de comentarios en tareas
- ✅ Archivos adjuntos
- ✅ Reportes y exportación
- ✅ Estado global bien estructurado

### **Estructura de Estado Sugerida**
```
Store/
├── auth/
│   ├── user, isAuthenticated, loading, error
│   └── actions: login, logout, updateProfile
├── projects/
│   ├── items, loading, error, filters
│   └── actions: fetch, create, update, delete
├── tasks/
│   ├── items, loading, error, filters
│   └── actions: fetch, create, update, delete, assign
├── users/
│   ├── items, loading, error
│   └── actions: fetch, create, update, delete
└── ui/
    ├── theme, sidebar, modals, notifications
    └── actions: toggleTheme, toggleSidebar, showModal
```

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cuándo deberías usar estado global vs estado local?
2. ¿Cuáles son las ventajas y desventajas de Context API vs Redux?
3. ¿Cómo funciona el patrón reducer en Redux?
4. ¿Qué son los selectores y por qué son importantes?
5. ¿Cómo se maneja el estado asíncrono en Redux?
6. ¿Qué es el middleware y cómo se implementa?
7. ¿Cómo se optimiza el rendimiento en aplicaciones con estado global?

### **Criterios de Evaluación**
- ✅ Implementar Context API para estado compartido
- ✅ Configurar y usar Redux Toolkit correctamente
- ✅ Crear slices y actions de Redux
- ✅ Implementar selectores optimizados
- ✅ Manejar estado asíncrono con thunks
- ✅ Crear middleware personalizado
- ✅ Implementar persistencia de estado
- ✅ Diseñar arquitectura de estado escalable

## 🔑 Conceptos Clave a Recordar

- **Context API** es ideal para estado simple y compartido
- **Redux Toolkit** es mejor para estado complejo y asíncrono
- **Selectores** optimizan el rendimiento evitando re-renderizados
- **Middleware** permite interceptar y modificar acciones
- **Redux Persist** mantiene el estado entre sesiones
- **Async thunks** manejan operaciones asíncronas
- **Normalización** mejora la estructura del estado
- **Hooks personalizados** encapsulan lógica de estado

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre hooks avanzados como useReducer, useCallback, useMemo, y técnicas de optimización de rendimiento.

## 📚 Recursos Adicionales

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [React Context API](https://react.dev/reference/react/createContext)
- [Redux Best Practices](https://redux.js.org/style-guide/)
- [Redux Persist](https://github.com/rt2zz/redux-persist)
- [Reselect](https://github.com/reduxjs/reselect)

## ✅ Checklist de Competencias

- [ ] Entiendo cuándo usar estado global vs estado local
- [ ] Implemento Context API para estado compartido
- [ ] Configuro y uso Redux Toolkit para estado complejo
- [ ] Creo stores y slices de Redux
- [ ] Implemento middleware personalizado
- [ ] Manejo estado asíncrono con Redux Thunk
- [ ] Creo selectores optimizados con Reselect
- [ ] Implemento persistencia de estado
- [ ] Creo hooks personalizados para estado global
- [ ] Completo el proyecto integrador del gestor de proyectos

---

**¡Excelente progreso! Has dominado el estado global y estás listo para técnicas avanzadas de optimización.** 🎉
