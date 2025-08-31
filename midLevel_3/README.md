# 🟡 Mid Level 3: Testing y Debugging

## 🧭 Navegación del Curso

**← Anterior**: [Mid Level 2: Hooks Avanzados](../midLevel_2/README.md)  
**Siguiente →**: [Senior Level 1: Arquitectura](../senior_1/README.md)

---

# 🚀 Módulo 6: Testing y Debugging

## 📚 Descripción del Módulo

En este módulo aprenderás a implementar testing completo en aplicaciones React usando Jest y React Testing Library. También explorarás técnicas avanzadas de debugging, profiling, y herramientas para identificar y resolver problemas de rendimiento y bugs.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Configurar y usar Jest para testing de React
- ✅ Implementar testing de componentes con React Testing Library
- ✅ Crear tests unitarios, de integración y E2E
- ✅ Mockear dependencias y APIs externas
- ✅ Implementar testing de hooks personalizados
- ✅ Usar React DevTools para debugging
- ✅ Implementar error boundaries y manejo de errores
- ✅ Crear tests de accesibilidad
- ✅ Implementar testing de performance
- ✅ Configurar CI/CD para testing automatizado

## 📖 Contenido Teórico

### 1. Configuración de Jest y React Testing Library

#### Instalación y configuración:
```bash
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

#### Configuración de Jest (jest.config.js):
```javascript
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapping: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

#### setupTests.js:
```javascript
import '@testing-library/jest-dom';
import 'jest-environment-jsdom';

// Mock de matchMedia para tests
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

### 2. Testing de Componentes Básicos

#### Test de componente simple:
```jsx
// Button.jsx
function Button({ children, onClick, variant = 'primary', disabled = false }) {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button Component', () => {
  test('renderiza correctamente con texto', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  test('aplica la clase CSS correcta según variant', () => {
    render(<Button variant="secondary">Button</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-secondary');
  });

  test('llama onClick cuando se hace clic', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('está deshabilitado cuando disabled es true', () => {
    render(<Button disabled>Button</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

#### Test de componente con estado:
```jsx
// Counter.jsx
import { useState } from 'react';

function Counter({ initialValue = 0 }) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);

  return (
    <div>
      <p data-testid="count">Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Counter.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

describe('Counter Component', () => {
  test('renderiza con valor inicial', () => {
    render(<Counter initialValue={5} />);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 5');
  });

  test('incrementa el contador', () => {
    render(<Counter />);
    const incrementButton = screen.getByText('+');
    
    fireEvent.click(incrementButton);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 1');
  });

  test('decrementa el contador', () => {
    render(<Counter initialValue={5} />);
    const decrementButton = screen.getByText('-');
    
    fireEvent.click(decrementButton);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 4');
  });

  test('resetea el contador', () => {
    render(<Counter initialValue={10} />);
    const incrementButton = screen.getByText('+');
    const resetButton = screen.getByText('Reset');
    
    fireEvent.click(incrementButton);
    fireEvent.click(resetButton);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 10');
  });
});
```

### 3. Testing de Hooks Personalizados

#### Test de hook personalizado:
```jsx
// useCounter.js
import { useState, useCallback } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
}

// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter Hook', () => {
  test('retorna valor inicial', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  test('incrementa el contador', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  test('decrementa el contador', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  test('resetea el contador', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

### 4. Testing de Componentes con Context

#### Test de componente con Context:
```jsx
// ThemeContext.jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme debe usarse dentro de ThemeProvider');
  }
  return context;
}

// ThemeToggle.jsx
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme} data-testid="theme-toggle">
      Current theme: {theme}
    </button>
  );
}

// ThemeToggle.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeProvider } from './ThemeContext';
import ThemeToggle from './ThemeToggle';

function renderWithTheme(component) {
  return render(
    <ThemeProvider>
      {component}
    </ThemeProvider>
  );
}

describe('ThemeToggle Component', () => {
  test('renderiza con tema inicial', () => {
    renderWithTheme(<ThemeToggle />);
    expect(screen.getByTestId('theme-toggle')).toHaveTextContent('Current theme: light');
  });

  test('cambia el tema al hacer clic', () => {
    renderWithTheme(<ThemeToggle />);
    const button = screen.getByTestId('theme-toggle');
    
    fireEvent.click(button);
    expect(button).toHaveTextContent('Current theme: dark');
    
    fireEvent.click(button);
    expect(button).toHaveTextContent('Current theme: light');
  });
});
```

### 5. Testing de APIs y Mocking

#### Test con API mockeada:
```jsx
// UserList.jsx
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError('Error al cargar usuarios');
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// UserList.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import UserList from './UserList';

// Mock de fetch global
global.fetch = jest.fn();

describe('UserList Component', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('muestra loading inicialmente', () => {
    render(<UserList />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  test('muestra usuarios cuando la API responde exitosamente', async () => {
    const mockUsers = [
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Smith' }
    ];

    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUsers
    });

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Jane Smith')).toBeInTheDocument();
    });
  });

  test('muestra error cuando la API falla', async () => {
    fetch.mockRejectedValueOnce(new Error('API Error'));

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('Error: Error al cargar usuarios')).toBeInTheDocument();
    });
  });
});
```

### 6. Testing de Formularios

#### Test de formulario:
```jsx
// LoginForm.jsx
import { useState } from 'react';

function LoginForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Limpiar error del campo
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validación básica
    const newErrors = {};
    if (!formData.email) newErrors.email = 'Email es requerido';
    if (!formData.password) newErrors.password = 'Contraseña es requerida';
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit} data-testid="login-form">
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
          data-testid="email-input"
        />
        {errors.email && <span data-testid="email-error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Contraseña"
          data-testid="password-input"
        />
        {errors.password && <span data-testid="password-error">{errors.password}</span>}
      </div>
      
      <button type="submit" data-testid="submit-button">
        Iniciar sesión
      </button>
    </form>
  );
}

// LoginForm.test.jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from './LoginForm';

describe('LoginForm Component', () => {
  test('renderiza todos los campos', () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    
    expect(screen.getByTestId('email-input')).toBeInTheDocument();
    expect(screen.getByTestId('password-input')).toBeInTheDocument();
    expect(screen.getByTestId('submit-button')).toBeInTheDocument();
  });

  test('valida campos requeridos', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={jest.fn()} />);
    
    const submitButton = screen.getByTestId('submit-button');
    await user.click(submitButton);
    
    expect(screen.getByTestId('email-error')).toHaveTextContent('Email es requerido');
    expect(screen.getByTestId('password-error')).toHaveTextContent('Contraseña es requerida');
  });

  test('llama onSubmit con datos válidos', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    const emailInput = screen.getByTestId('email-input');
    const passwordInput = screen.getByTestId('password-input');
    const submitButton = screen.getByTestId('submit-button');
    
    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'password123');
    await user.click(submitButton);
    
    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });

  test('limpia errores al escribir', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={jest.fn()} />);
    
    // Generar error
    const submitButton = screen.getByTestId('submit-button');
    await user.click(submitButton);
    
    expect(screen.getByTestId('email-error')).toBeInTheDocument();
    
    // Escribir en el campo
    const emailInput = screen.getByTestId('email-input');
    await user.type(emailInput, 'test@example.com');
    
    // El error debería desaparecer
    expect(screen.queryByTestId('email-error')).not.toBeInTheDocument();
  });
});
```

### 7. Testing de Accesibilidad

#### Test de accesibilidad:
```jsx
// AccessibleButton.jsx
function AccessibleButton({ children, onClick, ariaLabel, disabled = false }) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      aria-label={ariaLabel}
      aria-pressed={false}
    >
      {children}
    </button>
  );
}

// AccessibleButton.test.jsx
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import AccessibleButton from './AccessibleButton';

expect.extend(toHaveNoViolations);

describe('AccessibleButton Component', () => {
  test('no tiene violaciones de accesibilidad', async () => {
    const { container } = render(
      <AccessibleButton ariaLabel="Botón de acción">
        Click me
      </AccessibleButton>
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  test('tiene aria-label correcto', () => {
    render(
      <AccessibleButton ariaLabel="Botón de acción">
        Click me
      </AccessibleButton>
    );
    
    expect(screen.getByRole('button')).toHaveAttribute('aria-label', 'Botón de acción');
  });

  test('tiene aria-pressed cuando es apropiado', () => {
    render(
      <AccessibleButton ariaLabel="Botón toggle" ariaPressed={true}>
        Toggle
      </AccessibleButton>
    );
    
    expect(screen.getByRole('button')).toHaveAttribute('aria-pressed', 'true');
  });
});
```

### 8. Testing de Performance

#### Test de performance:
```jsx
// PerformanceTest.jsx
import { useState, useCallback } from 'react';

function PerformanceTest() {
  const [count, setCount] = useState(0);
  
  const expensiveOperation = useCallback(() => {
    // Simular operación costosa
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.random();
    }
    return result;
  }, []);

  const handleClick = () => {
    const result = expensiveOperation();
    setCount(prev => prev + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Perform Expensive Operation</button>
    </div>
  );
}

// PerformanceTest.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import PerformanceTest from './PerformanceTest';

describe('PerformanceTest Component', () => {
  test('renderiza sin problemas de performance', () => {
    const startTime = performance.now();
    
    render(<PerformanceTest />);
    
    const endTime = performance.now();
    const renderTime = endTime - startTime;
    
    // El renderizado debería tomar menos de 100ms
    expect(renderTime).toBeLessThan(100);
  });

  test('maneja operaciones costosas sin bloquear', () => {
    render(<PerformanceTest />);
    
    const button = screen.getByRole('button');
    const startTime = performance.now();
    
    fireEvent.click(button);
    
    const endTime = performance.now();
    const clickTime = endTime - startTime;
    
    // La operación debería completarse en un tiempo razonable
    expect(clickTime).toBeLessThan(1000);
  });
});
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Test de Componente de Lista**
Crea tests completos para un componente de lista que incluya renderizado, interacciones y edge cases.

### **Ejercicio 2: Test de Hook Personalizado**
Implementa tests para un hook personalizado que maneje formularios con validación.

### **Ejercicio 3: Test de Componente con Context**
Crea tests para un componente que use Context API para estado compartido.

### **Ejercicio 4: Test de Formulario Complejo**
Implementa tests exhaustivos para un formulario multi-paso con validación.

### **Ejercicio 5: Test de API Mockeada**
Crea tests para componentes que consuman APIs externas usando mocking.

### **Ejercicio 6: Test de Accesibilidad**
Implementa tests de accesibilidad usando jest-axe para componentes complejos.

### **Ejercicio 7: Test de Performance**
Crea tests que verifiquen el rendimiento de componentes y operaciones costosas.

### **Ejercicio 8: Test de Error Boundaries**
Implementa tests para componentes que manejen errores y excepciones.

### **Ejercicio 9: Test de Integración**
Crea tests de integración que prueben múltiples componentes trabajando juntos.

### **Ejercicio 10: Test de E2E Básico**
Implementa tests end-to-end usando Cypress para flujos de usuario completos.

## 🎯 Proyecto Integrador: Sistema de Usuarios

### **Descripción del Proyecto**
Construye un sistema completo de gestión de usuarios con testing exhaustivo que incluya CRUD, validaciones, autenticación y manejo de errores.

### **Requisitos del Proyecto**
- ✅ CRUD completo de usuarios
- ✅ Formularios con validación
- ✅ Autenticación y autorización
- ✅ Manejo de errores y loading states
- ✅ Tests unitarios para todos los componentes
- ✅ Tests de integración para flujos completos
- ✅ Tests de accesibilidad
- ✅ Tests de performance
- ✅ Coverage de código > 90%
- ✅ CI/CD pipeline para testing

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cómo se configura Jest para testing de React?
2. ¿Cuál es la diferencia entre render y screen en React Testing Library?
3. ¿Cómo se testean hooks personalizados?
4. ¿Qué es mocking y cuándo se usa?
5. ¿Cómo se implementan tests de accesibilidad?
6. ¿Qué son los tests de integración vs unitarios?
7. ¿Cómo se mide la coverage de código?

### **Criterios de Evaluación**
- ✅ Configurar Jest y React Testing Library
- ✅ Crear tests unitarios para componentes
- ✅ Testear hooks personalizados
- ✅ Implementar mocking de APIs
- ✅ Crear tests de accesibilidad
- ✅ Implementar tests de performance
- ✅ Alcanzar coverage > 90%
- ✅ Configurar CI/CD para testing

## 🔑 Conceptos Clave a Recordar

- **Jest** es el framework de testing principal para React
- **React Testing Library** facilita testing de componentes
- **Mocking** simula dependencias externas para tests aislados
- **Coverage** mide qué porcentaje del código está cubierto por tests
- **Tests unitarios** prueban componentes individuales
- **Tests de integración** prueban múltiples componentes juntos
- **Tests de accesibilidad** verifican que la app sea usable para todos
- **CI/CD** automatiza el proceso de testing

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre arquitectura avanzada, patrones de diseño, y cómo crear aplicaciones escalables y mantenibles.

## 📚 Recursos Adicionales

- [Jest Documentation](https://jestjs.io/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Testing Best Practices](https://react.dev/learn/testing)
- [Accessibility Testing](https://github.com/nickcolley/jest-axe)
- [Performance Testing](https://web.dev/performance-testing/)

## ✅ Checklist de Competencias

- [ ] Configuro y uso Jest para testing de React
- [ ] Implemento testing de componentes con React Testing Library
- [ ] Creo tests unitarios, de integración y E2E
- [ ] Mockeo dependencias y APIs externas
- [ ] Implemento testing de hooks personalizados
- [ ] Uso React DevTools para debugging
- [ ] Implemento error boundaries y manejo de errores
- [ ] Creo tests de accesibilidad
- [ ] Implemento testing de performance
- [ ] Completo el proyecto integrador del sistema de usuarios

---

**¡Excelente progreso! Has dominado testing y debugging. Estás listo para arquitectura avanzada.** 🎉
