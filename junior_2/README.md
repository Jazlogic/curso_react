# 🚀 Módulo 2: Componentes y Hooks Básicos

## 📚 Descripción del Módulo

En este módulo profundizarás en el uso de hooks básicos de React, especialmente `useEffect`, y aprenderás sobre el ciclo de vida de los componentes. También explorarás técnicas para crear componentes más reutilizables y manejar efectos secundarios de manera efectiva.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Entender y usar el hook useEffect correctamente
- ✅ Comprender el ciclo de vida de los componentes funcionales
- ✅ Crear componentes reutilizables y modulares
- ✅ Manejar efectos secundarios (API calls, suscripciones)
- ✅ Implementar limpieza de efectos y suscripciones
- ✅ Crear hooks personalizados básicos
- ✅ Manejar formularios complejos con validación
- ✅ Implementar navegación básica entre componentes
- ✅ Optimizar re-renderizados con React.memo
- ✅ Debuggear problemas comunes de hooks

## 📖 Contenido Teórico

### 1. El Hook useEffect

`useEffect` es un hook que permite ejecutar código después de que el componente se renderiza, y es fundamental para manejar efectos secundarios.

#### Sintaxis básica:
```jsx
import { useEffect } from 'react';

function MiComponente() {
  useEffect(() => {
    // Código que se ejecuta después del renderizado
    console.log('Componente renderizado');
  });

  return <div>Mi componente</div>;
}
```

#### useEffect con array de dependencias:
```jsx
function Contador() {
  const [contador, setContador] = useState(0);
  const [nombre, setNombre] = useState('');

  // Se ejecuta solo cuando cambia 'contador'
  useEffect(() => {
    document.title = `Contador: ${contador}`;
  }, [contador]);

  // Se ejecuta solo cuando cambia 'nombre'
  useEffect(() => {
    console.log(`Nombre cambiado a: ${nombre}`);
  }, [nombre]);

  // Se ejecuta solo una vez (al montar)
  useEffect(() => {
    console.log('Componente montado');
  }, []);

  return (
    <div>
      <p>Contador: {contador}</p>
      <button onClick={() => setContador(contador + 1)}>
        Incrementar
      </button>
      <input
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Tu nombre"
      />
    </div>
  );
}
```

#### useEffect con función de limpieza:
```jsx
function ChatComponent() {
  const [mensajes, setMensajes] = useState([]);

  useEffect(() => {
    // Suscripción a un chat
    const suscripcion = chatService.subscribe((nuevoMensaje) => {
      setMensajes(prev => [...prev, nuevoMensaje]);
    });

    // Función de limpieza (cleanup)
    return () => {
      suscripcion.unsubscribe();
    };
  }, []);

  return (
    <div>
      {mensajes.map((msg, index) => (
        <p key={index}>{msg.texto}</p>
      ))}
    </div>
  );
}
```

### 2. Ciclo de Vida de Componentes Funcionales

Los componentes funcionales tienen un ciclo de vida que se maneja a través de hooks:

#### Fases del ciclo de vida:
```jsx
function ComponenteConCicloDeVida() {
  // 1. Montaje (Mount)
  useEffect(() => {
    console.log('Componente montado');
    
    // Configuración inicial
    const timer = setInterval(() => {
      console.log('Timer ejecutándose');
    }, 1000);

    // Cleanup al desmontar
    return () => {
      console.log('Componente desmontado');
      clearInterval(timer);
    };
  }, []);

  // 2. Actualización (Update)
  const [contador, setContador] = useState(0);
  
  useEffect(() => {
    console.log('Contador actualizado:', contador);
  }, [contador]);

  // 3. Desmontaje (Unmount) - se maneja en el return del useEffect

  return (
    <div>
      <p>Contador: {contador}</p>
      <button onClick={() => setContador(contador + 1)}>
        Incrementar
      </button>
    </div>
  );
}
```

### 3. Llamadas a APIs con useEffect

#### Fetch básico:
```jsx
function ListaUsuarios() {
  const [usuarios, setUsuarios] = useState([]);
  const [cargando, setCargando] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsuarios = async () => {
      try {
        setCargando(true);
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        const data = await response.json();
        setUsuarios(data);
      } catch (err) {
        setError('Error al cargar usuarios');
        console.error(err);
      } finally {
        setCargando(false);
      }
    };

    fetchUsuarios();
  }, []);

  if (cargando) return <div>Cargando usuarios...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {usuarios.map(usuario => (
        <li key={usuario.id}>{usuario.name}</li>
      ))}
    </ul>
  );
}
```

#### Fetch con parámetros:
```jsx
function UsuarioDetalle({ userId }) {
  const [usuario, setUsuario] = useState(null);
  const [cargando, setCargando] = useState(false);

  useEffect(() => {
    if (!userId) return;

    const fetchUsuario = async () => {
      setCargando(true);
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
        const data = await response.json();
        setUsuario(data);
      } catch (err) {
        console.error('Error al cargar usuario:', err);
      } finally {
        setCargando(false);
      }
    };

    fetchUsuario();
  }, [userId]); // Se ejecuta cuando cambia userId

  if (cargando) return <div>Cargando usuario...</div>;
  if (!usuario) return <div>Selecciona un usuario</div>;

  return (
    <div>
      <h2>{usuario.name}</h2>
      <p>Email: {usuario.email}</p>
      <p>Teléfono: {usuario.phone}</p>
    </div>
  );
}
```

### 4. Hooks Personalizados

Los hooks personalizados permiten extraer lógica reutilizable de los componentes.

#### Hook personalizado para formularios:
```jsx
function useFormulario(estadoInicial) {
  const [valores, setValores] = useState(estadoInicial);
  const [errores, setErrores] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValores(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Limpiar error del campo
    if (errores[name]) {
      setErrores(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };

  const handleSubmit = (callback) => (e) => {
    e.preventDefault();
    callback(valores);
  };

  const reset = () => {
    setValores(estadoInicial);
    setErrores({});
  };

  return {
    valores,
    errores,
    handleChange,
    handleSubmit,
    reset,
    setValores,
    setErrores
  };
}

// Uso del hook personalizado
function FormularioLogin() {
  const { valores, errores, handleChange, handleSubmit } = useFormulario({
    email: '',
    password: ''
  });

  const onSubmit = (datos) => {
    console.log('Datos del formulario:', datos);
    // Lógica de login
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        type="email"
        name="email"
        value={valores.email}
        onChange={handleChange}
        placeholder="Email"
      />
      {errores.email && <span>{errores.email}</span>}
      
      <input
        type="password"
        name="password"
        value={valores.password}
        onChange={handleChange}
        placeholder="Contraseña"
      />
      {errores.password && <span>{errores.password}</span>}
      
      <button type="submit">Iniciar sesión</button>
    </form>
  );
}
```

#### Hook personalizado para API calls:
```jsx
function useApi(url) {
  const [data, setData] = useState(null);
  const [cargando, setCargando] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setCargando(true);
        setError(null);
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setCargando(false);
      }
    };

    if (url) {
      fetchData();
    }
  }, [url]);

  return { data, cargando, error };
}

// Uso del hook
function ListaPosts() {
  const { data: posts, cargando, error } = useApi('https://jsonplaceholder.typicode.com/posts');

  if (cargando) return <div>Cargando posts...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </article>
      ))}
    </div>
  );
}
```

### 5. Componentes Reutilizables

#### Componente Button reutilizable:
```jsx
function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium', 
  onClick, 
  disabled = false,
  type = 'button'
}) {
  const baseClasses = 'btn';
  const variantClasses = {
    primary: 'btn-primary',
    secondary: 'btn-secondary',
    danger: 'btn-danger',
    success: 'btn-success'
  };
  const sizeClasses = {
    small: 'btn-sm',
    medium: 'btn-md',
    large: 'btn-lg'
  };

  const classes = [
    baseClasses,
    variantClasses[variant],
    sizeClasses[size]
  ].join(' ');

  return (
    <button
      className={classes}
      onClick={onClick}
      disabled={disabled}
      type={type}
    >
      {children}
    </button>
  );
}

// Uso del componente
function App() {
  return (
    <div>
      <Button variant="primary" onClick={() => alert('¡Hola!')}>
        Botón Primario
      </Button>
      <Button variant="danger" size="large">
        Botón Peligro
      </Button>
      <Button variant="success" disabled>
        Botón Deshabilitado
      </Button>
    </div>
  );
}
```

#### Componente Modal reutilizable:
```jsx
function Modal({ 
  isOpen, 
  onClose, 
  title, 
  children, 
  size = 'medium' 
}) {
  if (!isOpen) return null;

  const sizeClasses = {
    small: 'modal-sm',
    medium: 'modal-md',
    large: 'modal-lg'
  };

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div 
        className={`modal ${sizeClasses[size]}`}
        onClick={(e) => e.stopPropagation()}
      >
        <div className="modal-header">
          <h2>{title}</h2>
          <button className="modal-close" onClick={onClose}>
            ×
          </button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
}

// Uso del modal
function App() {
  const [modalAbierto, setModalAbierto] = useState(false);

  return (
    <div>
      <button onClick={() => setModalAbierto(true)}>
        Abrir Modal
      </button>
      
      <Modal
        isOpen={modalAbierto}
        onClose={() => setModalAbierto(false)}
        title="Mi Modal"
        size="large"
      >
        <p>Este es el contenido del modal.</p>
        <p>Puedes poner cualquier contenido aquí.</p>
      </Modal>
    </div>
  );
}
```

### 6. Optimización con React.memo

`React.memo` evita re-renderizados innecesarios de componentes:

```jsx
const UsuarioCard = React.memo(function UsuarioCard({ usuario, onEdit }) {
  console.log('UsuarioCard renderizado');
  
  return (
    <div className="usuario-card">
      <h3>{usuario.nombre}</h3>
      <p>{usuario.email}</p>
      <button onClick={() => onEdit(usuario.id)}>
        Editar
      </button>
    </div>
  );
});

// Componente padre
function ListaUsuarios() {
  const [usuarios, setUsuarios] = useState([]);
  const [filtro, setFiltro] = useState('');

  const handleEdit = useCallback((userId) => {
    console.log('Editando usuario:', userId);
  }, []);

  const usuariosFiltrados = usuarios.filter(usuario =>
    usuario.nombre.toLowerCase().includes(filtro.toLowerCase())
  );

  return (
    <div>
      <input
        value={filtro}
        onChange={(e) => setFiltro(e.target.value)}
        placeholder="Filtrar usuarios..."
      />
      
      {usuariosFiltrados.map(usuario => (
        <UsuarioCard
          key={usuario.id}
          usuario={usuario}
          onEdit={handleEdit}
        />
      ))}
    </div>
  );
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Timer con useEffect**
Crea un componente `Timer` que muestre un contador que se incrementa cada segundo, con botones para pausar, reanudar y resetear.

### **Ejercicio 2: Hook Personalizado para Contador**
Crea un hook personalizado `useContador` que maneje la lógica de un contador y úsalo en múltiples componentes.

### **Ejercicio 3: Lista de Usuarios con API**
Crea un componente que consuma una API (como JSONPlaceholder) y muestre una lista de usuarios con búsqueda y filtrado.

### **Ejercicio 4: Formulario de Registro**
Crea un formulario de registro completo con validación, usando el hook personalizado `useFormulario`.

### **Ejercicio 5: Componente Modal Reutilizable**
Crea un componente `Modal` completamente reutilizable que pueda mostrar cualquier contenido.

### **Ejercicio 6: Hook para LocalStorage**
Crea un hook personalizado `useLocalStorage` que sincronice el estado con localStorage.

### **Ejercicio 7: Lista de Tareas con Persistencia**
Crea una lista de tareas que persista los datos en localStorage usando useEffect.

### **Ejercicio 8: Componente de Búsqueda**
Crea un componente de búsqueda que use debouncing para evitar llamadas innecesarias a la API.

### **Ejercicio 9: Hook para Ventana**
Crea un hook personalizado `useWindowSize` que detecte cambios en el tamaño de la ventana.

### **Ejercicio 10: Componente de Paginación**
Crea un componente de paginación reutilizable que funcione con cualquier lista de datos.

## 🎯 Proyecto Integrador: Lista de Tareas

### **Descripción del Proyecto**
Construye una aplicación completa de gestión de tareas que incluya creación, edición, eliminación, marcado como completada, filtros, búsqueda y persistencia de datos.

### **Requisitos del Proyecto**
- ✅ CRUD completo de tareas (Crear, Leer, Actualizar, Eliminar)
- ✅ Filtros por estado (todas, pendientes, completadas)
- ✅ Búsqueda en tiempo real
- ✅ Persistencia en localStorage
- ✅ Categorías de tareas
- ✅ Fechas de vencimiento
- ✅ Prioridades (baja, media, alta)
- ✅ Estadísticas de tareas
- ✅ Interfaz responsive y moderna
- ✅ Hooks personalizados para lógica reutilizable

### **Estructura de Componentes Sugerida**
```
App/
├── Header/
├── TaskForm/
├── TaskList/
│   ├── TaskItem/
│   └── TaskFilters/
├── TaskStats/
├── useTasks (hook personalizado)
├── useLocalStorage (hook personalizado)
└── useDebounce (hook personalizado)
```

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cuándo se ejecuta useEffect sin array de dependencias?
2. ¿Cómo se implementa la limpieza en useEffect?
3. ¿Cuál es la diferencia entre useEffect y useLayoutEffect?
4. ¿Cómo se crean hooks personalizados?
5. ¿Qué es React.memo y cuándo se usa?
6. ¿Cómo se manejan las llamadas a APIs con useEffect?
7. ¿Cuáles son las mejores prácticas para useEffect?

### **Criterios de Evaluación**
- ✅ Usar useEffect correctamente para efectos secundarios
- ✅ Implementar limpieza de efectos y suscripciones
- ✅ Crear hooks personalizados reutilizables
- ✅ Manejar llamadas a APIs de manera efectiva
- ✅ Crear componentes reutilizables y modulares
- ✅ Optimizar re-renderizados con React.memo
- ✅ Implementar formularios complejos con validación
- ✅ Manejar estado local y efectos secundarios

## 🔑 Conceptos Clave a Recordar

- **useEffect** maneja efectos secundarios y ciclo de vida
- **Array de dependencias** controla cuándo se ejecuta useEffect
- **Función de limpieza** evita memory leaks
- **Hooks personalizados** extraen lógica reutilizable
- **React.memo** optimiza re-renderizados
- **useCallback** y **useMemo** optimizan funciones y valores
- **Componentes reutilizables** mejoran la mantenibilidad
- **LocalStorage** permite persistencia de datos

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre React Router para navegación entre páginas, manejo de estado más complejo, y cómo estructurar aplicaciones multi-página.

## 📚 Recursos Adicionales

- [useEffect Hook](https://react.dev/reference/react/useEffect)
- [Hooks Personalizados](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [React.memo](https://react.dev/reference/react/memo)
- [useCallback y useMemo](https://react.dev/reference/react/useCallback)
- [Ciclo de Vida en Hooks](https://react.dev/learn/lifecycle-of-reactive-effects)

## ✅ Checklist de Competencias

- [ ] Entiendo y uso useEffect correctamente
- [ ] Comprendo el ciclo de vida de componentes funcionales
- [ ] Creo componentes reutilizables y modulares
- [ ] Manejo efectos secundarios (API calls, suscripciones)
- [ ] Implemento limpieza de efectos y suscripciones
- [ ] Creo hooks personalizados básicos
- [ ] Manejo formularios complejos con validación
- [ ] Implemento navegación básica entre componentes
- [ ] Optimizo re-renderizados con React.memo
- [ ] Completo el proyecto integrador de lista de tareas

---

**¡Excelente progreso! Has dominado los hooks básicos y estás listo para el siguiente nivel.** 🎉
