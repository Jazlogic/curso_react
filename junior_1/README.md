# 🚀 Módulo 1: Fundamentos de React y JSX

## 📚 Descripción del Módulo

En este módulo aprenderás los fundamentos esenciales de React, desde qué es React y por qué es tan popular, hasta crear tu primer componente y entender el JSX. Establecerás una base sólida que te permitirá construir aplicaciones React complejas en los siguientes módulos.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Entender qué es React y sus ventajas
- ✅ Crear y configurar un proyecto React
- ✅ Comprender y usar JSX correctamente
- ✅ Crear componentes funcionales básicos
- ✅ Pasar y recibir props entre componentes
- ✅ Manejar estado local básico con useState
- ✅ Renderizar listas y elementos condicionales
- ✅ Implementar eventos básicos (onClick, onChange)
- ✅ Estructurar una aplicación React simple
- ✅ Debuggear problemas comunes de React

## 📖 Contenido Teórico

### 1. ¿Qué es React?

React es una biblioteca de JavaScript desarrollada por Facebook (Meta) para construir interfaces de usuario interactivas y reutilizables. Es declarativa, eficiente y flexible.

**Características principales:**
- **Componentes**: Bloques de construcción reutilizables
- **Virtual DOM**: Renderizado eficiente y rápido
- **Unidireccional**: Flujo de datos predecible
- **Declarativo**: Describes el resultado, no los pasos
- **Ecosistema rico**: Herramientas y bibliotecas complementarias

### 2. Configuración del Entorno

#### Crear un proyecto React con Vite:
```bash
npm create vite@latest mi-app-react -- --template react
cd mi-app-react
npm install
npm run dev
```

#### Estructura básica de un proyecto React:
```
mi-app-react/
├── public/
│   └── index.html
├── src/
│   ├── App.jsx
│   ├── main.jsx
│   └── components/
├── package.json
└── vite.config.js
```

### 3. JSX (JavaScript XML)

JSX es una extensión de sintaxis para JavaScript que permite escribir HTML dentro de JavaScript.

#### Sintaxis básica:
```jsx
// JSX básico
const element = <h1>¡Hola, React!</h1>;

// JSX con expresiones JavaScript
const name = "React";
const element = <h1>¡Hola, {name}!</h1>;

// JSX con atributos
const element = <img src="logo.png" alt="Logo" />;

// JSX anidado
const element = (
  <div>
    <h1>Título</h1>
    <p>Párrafo</p>
  </div>
);
```

#### Reglas importantes de JSX:
- **Un solo elemento raíz**: JSX debe tener un solo elemento padre
- **Atributos en camelCase**: `className` en lugar de `class`
- **Expresiones JavaScript**: Usar `{}` para código JavaScript
- **Cerrar todas las etiquetas**: `<img />` o `<div></div>`

### 4. Componentes Funcionales

Los componentes son funciones que retornan JSX y pueden recibir props como parámetros.

#### Componente básico:
```jsx
function Saludo() {
  return <h1>¡Hola desde React!</h1>;
}

// Uso del componente
<Saludo />
```

#### Componente con props:
```jsx
function Saludo(props) {
  return <h1>¡Hola, {props.nombre}!</h1>;
}

// Uso con props
<Saludo nombre="Juan" />
```

#### Destructuring de props:
```jsx
function Saludo({ nombre, edad }) {
  return (
    <div>
      <h1>¡Hola, {nombre}!</h1>
      <p>Tienes {edad} años</p>
    </div>
  );
}

// Uso
<Saludo nombre="María" edad={25} />
```

### 5. Estado Local con useState

useState es un hook que permite agregar estado local a componentes funcionales.

#### Sintaxis básica:
```jsx
import { useState } from 'react';

function Contador() {
  const [contador, setContador] = useState(0);
  
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

#### Múltiples estados:
```jsx
function Formulario() {
  const [nombre, setNombre] = useState('');
  const [email, setEmail] = useState('');
  
  return (
    <form>
      <input
        type="text"
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
    </form>
  );
}
```

### 6. Renderizado Condicional

#### Renderizado con operador ternario:
```jsx
function Saludo({ usuario }) {
  return (
    <div>
      {usuario ? (
        <h1>¡Bienvenido, {usuario}!</h1>
      ) : (
        <h1>¡Bienvenido, invitado!</h1>
      )}
    </div>
  );
}
```

#### Renderizado con operador lógico:
```jsx
function Mensaje({ mostrar, texto }) {
  return (
    <div>
      {mostrar && <p>{texto}</p>}
    </div>
  );
}
```

### 7. Renderizado de Listas

#### Renderizar arrays con map:
```jsx
function ListaTareas({ tareas }) {
  return (
    <ul>
      {tareas.map((tarea, index) => (
        <li key={index}>{tarea}</li>
      ))}
    </ul>
  );
}

// Uso
const tareas = ['Aprender React', 'Practicar JSX', 'Crear componentes'];
<ListaTareas tareas={tareas} />
```

**Importante**: Siempre usar `key` única para cada elemento de la lista.

### 8. Manejo de Eventos

#### Eventos básicos:
```jsx
function Boton() {
  const handleClick = () => {
    alert('¡Botón clickeado!');
  };
  
  return <button onClick={handleClick}>Haz clic</button>;
}
```

#### Eventos con parámetros:
```jsx
function ListaBotones() {
  const handleClick = (id) => {
    console.log(`Botón ${id} clickeado`);
  };
  
  return (
    <div>
      <button onClick={() => handleClick(1)}>Botón 1</button>
      <button onClick={() => handleClick(2)}>Botón 2</button>
    </div>
  );
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Mi Primer Componente**
Crea un componente `MiPrimerComponente` que muestre tu nombre y una breve descripción sobre ti.

### **Ejercicio 2: Calculadora Simple**
Crea un componente `Calculadora` que permita sumar dos números usando inputs y un botón.

### **Ejercicio 3: Lista de Compras**
Crea un componente `ListaCompras` que muestre una lista de productos y permita agregar nuevos productos.

### **Ejercicio 4: Contador con Botones**
Crea un componente `Contador` con botones para incrementar, decrementar y resetear el contador.

### **Ejercicio 5: Formulario de Usuario**
Crea un componente `FormularioUsuario` que capture nombre, email y edad, y muestre los datos ingresados.

### **Ejercicio 6: Tarjeta de Producto**
Crea un componente `TarjetaProducto` que reciba props para nombre, precio e imagen, y muestre una tarjeta estilizada.

### **Ejercicio 7: Toggle de Tema**
Crea un componente `ToggleTema` que cambie entre tema claro y oscuro usando estado local.

### **Ejercicio 8: Lista de Tareas Interactiva**
Crea un componente `ListaTareas` que permita agregar, eliminar y marcar tareas como completadas.

### **Ejercicio 9: Galería de Imágenes**
Crea un componente `GaleriaImagenes` que muestre una lista de imágenes con navegación entre ellas.

### **Ejercicio 10: Formulario de Validación**
Crea un componente `FormularioValidacion` que valide email y contraseña, mostrando mensajes de error apropiados.

## 🎯 Proyecto Integrador: Calculadora React

### **Descripción del Proyecto**
Construye una calculadora completa que incluya operaciones básicas (suma, resta, multiplicación, división), operaciones avanzadas (potencia, raíz cuadrada), y funcionalidades como historial de operaciones y cambio de tema.

### **Requisitos del Proyecto**
- ✅ Interfaz de usuario intuitiva y responsive
- ✅ Operaciones matemáticas básicas y avanzadas
- ✅ Historial de operaciones recientes
- ✅ Cambio entre tema claro y oscuro
- ✅ Manejo de errores (división por cero, etc.)
- ✅ Diseño moderno y profesional
- ✅ Componentes reutilizables y bien estructurados

### **Estructura de Componentes Sugerida**
```
App/
├── Calculadora/
│   ├── Display/
│   ├── TecladoNumerico/
│   ├── TecladoOperaciones/
│   ├── Historial/
│   └── ToggleTema/
└── Estilos/
```

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Qué es JSX y cuáles son sus reglas principales?
2. ¿Cómo se crea un componente funcional en React?
3. ¿Cuál es la diferencia entre props y estado?
4. ¿Cómo funciona el hook useState?
5. ¿Por qué es importante usar keys en listas?
6. ¿Cómo se manejan eventos en React?
7. ¿Qué es el renderizado condicional y cómo se implementa?

### **Criterios de Evaluación**
- ✅ Crear componentes funcionales básicos
- ✅ Usar JSX correctamente
- ✅ Manejar props y estado local
- ✅ Implementar renderizado condicional
- ✅ Renderizar listas con keys únicas
- ✅ Manejar eventos básicos
- ✅ Estructurar código de manera clara

## 🔑 Conceptos Clave a Recordar

- **React** es una biblioteca para construir interfaces de usuario
- **JSX** permite escribir HTML en JavaScript
- **Componentes** son funciones que retornan JSX
- **Props** son datos que se pasan a componentes
- **Estado** es información que puede cambiar en un componente
- **useState** es un hook para manejar estado local
- **Keys** son necesarias para renderizar listas eficientemente
- **Eventos** se manejan con funciones y props como onClick

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre hooks más avanzados como `useEffect`, el ciclo de vida de los componentes, y cómo crear aplicaciones más complejas con múltiples componentes interactuando entre sí.

## 📚 Recursos Adicionales

- [Documentación oficial de React](https://react.dev/)
- [React Tutorial oficial](https://react.dev/learn)
- [JSX en profundidad](https://react.dev/learn/writing-markup-with-jsx)
- [Componentes y Props](https://react.dev/learn/passing-props-to-a-component)
- [Estado y Ciclo de Vida](https://react.dev/learn/state-a-components-memory)

## ✅ Checklist de Competencias

- [ ] Entiendo qué es React y sus ventajas
- [ ] Puedo crear y configurar un proyecto React
- [ ] Comprendo y uso JSX correctamente
- [ ] Creo componentes funcionales básicos
- [ ] Paso y recibo props entre componentes
- [ ] Manejo estado local básico con useState
- [ ] Renderizo listas y elementos condicionales
- [ ] Implemento eventos básicos
- [ ] Estructuro una aplicación React simple
- [ ] Completo el proyecto integrador de la calculadora

---

**¡Felicidades! Has completado el primer módulo del curso de React. Estás listo para continuar con el siguiente nivel.** 🎉
