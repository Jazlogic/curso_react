# 🟡 Mid Level 2: Hooks Avanzados y Performance

## 🧭 Navegación del Curso

**← Anterior**: [Mid Level 1: Estado Global](../midLevel_1/README.md)  
**Siguiente →**: [Mid Level 3: Testing](../midLevel_3/README.md)

---

# 🚀 Módulo 5: Hooks Avanzados y Performance

## 📚 Descripción del Módulo

En este módulo aprenderás a usar hooks avanzados como `useReducer`, `useCallback`, `useMemo`, y `useRef` para optimizar el rendimiento de tus aplicaciones React. También explorarás técnicas de memoización, lazy loading, y code splitting para crear aplicaciones más eficientes.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Usar useReducer para estado complejo
- ✅ Implementar useCallback y useMemo para optimización
- ✅ Manejar referencias con useRef
- ✅ Implementar lazy loading y code splitting
- ✅ Optimizar re-renderizados con técnicas avanzadas
- ✅ Usar React DevTools para profiling
- ✅ Implementar virtualización para listas grandes
- ✅ Crear componentes optimizados con React.memo
- ✅ Manejar performance con Web Workers
- ✅ Implementar técnicas de bundle optimization

## 📖 Contenido Teórico

### 1. useReducer - Estado Complejo

#### Sintaxis básica:
```jsx
import { useReducer } from 'react';

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <input
        type="number"
        value={state.step}
        onChange={(e) => dispatch({ type: 'setStep', payload: Number(e.target.value) })}
      />
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

### 2. useCallback - Memoización de Funciones

#### Optimización de funciones:
```jsx
import { useCallback, useState } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // Memoizar función para evitar re-renderizados innecesarios
  const handleIncrement = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Dependencias vacías = función nunca cambia

  const handleTextChange = useCallback((newText) => {
    setText(newText);
  }, []);

  return (
    <div>
      <input
        value={text}
        onChange={(e) => handleTextChange(e.target.value)}
        placeholder="Escribe algo..."
      />
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Incrementar</button>
      
      {/* ChildComponent solo se re-renderiza si sus props cambian */}
      <ChildComponent onIncrement={handleIncrement} />
    </div>
  );
}

const ChildComponent = React.memo(function ChildComponent({ onIncrement }) {
  console.log('ChildComponent renderizado');
  return <button onClick={onIncrement}>Incrementar desde hijo</button>;
});
```

### 3. useMemo - Memoización de Valores

#### Cálculos costosos:
```jsx
import { useMemo, useState } from 'react';

function ExpensiveComponent({ items, filter }) {
  // Memoizar cálculo costoso
  const filteredItems = useMemo(() => {
    console.log('Calculando items filtrados...');
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]); // Solo recalcular si items o filter cambian

  // Memoizar valor derivado
  const totalValue = useMemo(() => {
    return filteredItems.reduce((sum, item) => sum + item.price, 0);
  }, [filteredItems]);

  return (
    <div>
      <p>Items filtrados: {filteredItems.length}</p>
      <p>Valor total: ${totalValue}</p>
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name} - ${item.price}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 4. useRef - Referencias Persistentes

#### Referencias a elementos DOM:
```jsx
import { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef(null);
  const buttonRef = useRef(null);

  useEffect(() => {
    // Focus automático al montar
    inputRef.current?.focus();
  }, []);

  const handleFocus = () => {
    inputRef.current?.focus();
  };

  const handleScroll = () => {
    buttonRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  return (
    <div>
      <input ref={inputRef} placeholder="Este input se enfoca automáticamente" />
      <button onClick={handleFocus}>Enfocar input</button>
      <button ref={buttonRef} onClick={handleScroll}>Scroll a este botón</button>
    </div>
  );
}
```

#### Referencias para valores persistentes:
```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <p>Timer: {count}</p>
      <button onClick={startTimer}>Iniciar</button>
      <button onClick={stopTimer}>Detener</button>
    </div>
  );
}
```

### 5. Lazy Loading y Code Splitting

#### Lazy loading de componentes:
```jsx
import { lazy, Suspense } from 'react';

// Lazy loading de componentes pesados
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const ChartComponent = lazy(() => import('./ChartComponent'));

function App() {
  const [showHeavy, setShowHeavy] = useState(false);
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavy(!showHeavy)}>
        {showHeavy ? 'Ocultar' : 'Mostrar'} Componente Pesado
      </button>
      
      <button onClick={() => setShowChart(!showChart)}>
        {showChart ? 'Ocultar' : 'Mostrar'} Gráfico
      </button>

      <Suspense fallback={<div>Cargando...</div>}>
        {showHeavy && <HeavyComponent />}
        {showChart && <ChartComponent />}
      </Suspense>
    </div>
  );
}
```

#### Lazy loading de rutas:
```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
  return (
    <Suspense fallback={<div>Cargando aplicación...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

### 6. Optimización de Re-renderizados

#### React.memo con comparación personalizada:
```jsx
const UserCard = React.memo(function UserCard({ user, onEdit, onDelete }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Editar</button>
      <button onClick={() => onDelete(user.id)}>Eliminar</button>
    </div>
  );
}, (prevProps, nextProps) => {
  // Comparación personalizada para evitar re-renderizados innecesarios
  return (
    prevProps.user.id === nextProps.user.id &&
    prevProps.user.name === nextProps.user.name &&
    prevProps.user.email === nextProps.user.email
  );
});
```

#### useMemo para objetos y arrays:
```jsx
function UserList({ users, searchTerm, sortBy }) {
  // Memoizar usuarios filtrados y ordenados
  const processedUsers = useMemo(() => {
    let filtered = users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
    
    if (sortBy === 'name') {
      filtered.sort((a, b) => a.name.localeCompare(b.name));
    } else if (sortBy === 'email') {
      filtered.sort((a, b) => a.email.localeCompare(b.email));
    }
    
    return filtered;
  }, [users, searchTerm, sortBy]);

  // Memoizar función de callback
  const handleEdit = useCallback((userId) => {
    console.log('Editando usuario:', userId);
  }, []);

  const handleDelete = useCallback((userId) => {
    console.log('Eliminando usuario:', userId);
  }, []);

  return (
    <div>
      {processedUsers.map(user => (
        <UserCard
          key={user.id}
          user={user}
          onEdit={handleEdit}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: useReducer para Formulario Complejo**
Crea un formulario de registro con múltiples campos usando useReducer para manejar el estado complejo.

### **Ejercicio 2: Optimización con useCallback**
Implementa una lista de productos con funcionalidades de agregar al carrito, editar y eliminar, optimizando con useCallback.

### **Ejercicio 3: Cálculos Costosos con useMemo**
Crea un componente que calcule estadísticas complejas de una lista de datos, optimizando con useMemo.

### **Ejercicio 4: Lazy Loading de Componentes**
Implementa lazy loading para diferentes secciones de una aplicación, mostrando spinners de carga.

### **Ejercicio 5: Virtualización de Listas**
Crea una lista virtualizada que pueda manejar miles de elementos sin afectar el rendimiento.

### **Ejercicio 6: React.memo Avanzado**
Implementa React.memo con comparaciones personalizadas para componentes complejos.

### **Ejercicio 7: useRef para Animaciones**
Crea un componente que use useRef para implementar animaciones y transiciones suaves.

### **Ejercicio 8: Code Splitting por Rutas**
Implementa code splitting para diferentes páginas de una aplicación con React Router.

### **Ejercicio 9: Performance Profiling**
Usa React DevTools para identificar y resolver problemas de rendimiento en una aplicación.

### **Ejercicio 10: Bundle Optimization**
Analiza y optimiza el bundle de una aplicación usando herramientas como Webpack Bundle Analyzer.

## 🎯 Proyecto Integrador: Dashboard Analítico

### **Descripción del Proyecto**
Construye un dashboard analítico con múltiples gráficos, tablas de datos, y funcionalidades de filtrado que demuestre todas las técnicas de optimización aprendidas.

### **Requisitos del Proyecto**
- ✅ Múltiples tipos de gráficos (líneas, barras, donas)
- ✅ Tablas de datos con paginación y ordenamiento
- ✅ Filtros avanzados con useMemo
- ✅ Lazy loading de componentes pesados
- ✅ Virtualización para listas grandes
- ✅ Optimización con React.memo y useCallback
- ✅ Code splitting por secciones
- ✅ Performance monitoring
- ✅ Responsive design
- ✅ Tema claro/oscuro

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cuándo deberías usar useReducer vs useState?
2. ¿Cómo funciona useCallback y cuándo es útil?
3. ¿Qué es useMemo y cómo optimiza el rendimiento?
4. ¿Cómo se implementa lazy loading en React?
5. ¿Qué es la virtualización y cuándo se usa?
6. ¿Cómo funciona React.memo con comparaciones personalizadas?
7. ¿Cuáles son las mejores prácticas para optimización?

### **Criterios de Evaluación**
- ✅ Usar useReducer para estado complejo
- ✅ Implementar useCallback y useMemo correctamente
- ✅ Manejar referencias con useRef
- ✅ Implementar lazy loading y code splitting
- ✅ Optimizar re-renderizados
- ✅ Usar React DevTools para profiling
- ✅ Implementar virtualización
- ✅ Crear componentes optimizados

## 🔑 Conceptos Clave a Recordar

- **useReducer** maneja estado complejo con lógica de transiciones
- **useCallback** memoiza funciones para evitar re-renderizados
- **useMemo** memoiza valores calculados costosos
- **useRef** mantiene referencias persistentes
- **Lazy loading** mejora el tiempo de carga inicial
- **Code splitting** divide el bundle en chunks más pequeños
- **React.memo** evita re-renderizados innecesarios
- **Virtualización** mejora el rendimiento de listas grandes

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre testing y debugging avanzado con Jest, React Testing Library, y técnicas de debugging.

## 📚 Recursos Adicionales

- [useReducer Hook](https://react.dev/reference/react/useReducer)
- [useCallback Hook](https://react.dev/reference/react/useCallback)
- [useMemo Hook](https://react.dev/reference/react/useMemo)
- [useRef Hook](https://react.dev/reference/react/useRef)
- [React.memo](https://react.dev/reference/react/memo)
- [Code Splitting](https://react.dev/reference/react/lazy)

## ✅ Checklist de Competencias

- [ ] Uso useReducer para estado complejo
- [ ] Implemento useCallback y useMemo para optimización
- [ ] Manejo referencias con useRef
- [ ] Implemento lazy loading y code splitting
- [ ] Optimizo re-renderizados con técnicas avanzadas
- [ ] Uso React DevTools para profiling
- [ ] Implemento virtualización para listas grandes
- [ ] Creo componentes optimizados con React.memo
- [ ] Manejo performance con Web Workers
- [ ] Completo el proyecto integrador del dashboard analítico

---

**¡Excelente progreso! Has dominado los hooks avanzados y estás listo para testing y debugging.** 🎉
