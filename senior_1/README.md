# 🔴 Senior Level 1: Arquitectura y Patrones Avanzados

## 🧭 Navegación del Curso

**← Anterior**: [Mid Level 3: Testing](../midLevel_3/README.md)  
**Siguiente →**: [Senior Level 2: Testing Avanzado](../senior_2/README.md)

---

# 🚀 Módulo 7: Arquitectura y Patrones Avanzados

## 📚 Descripción del Módulo

En este módulo aprenderás a diseñar y implementar arquitecturas escalables para aplicaciones React empresariales. Explorarás patrones de diseño avanzados, micro-frontends, arquitectura de capas, y técnicas para crear aplicaciones mantenibles y escalables.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Diseñar arquitecturas escalables para aplicaciones grandes
- ✅ Implementar patrones de diseño empresariales
- ✅ Crear micro-frontends con React
- ✅ Implementar arquitectura de capas (Layered Architecture)
- ✅ Diseñar sistemas de plugins y extensiones
- ✅ Implementar patrones de comunicación entre módulos
- ✅ Crear arquitecturas orientadas a eventos
- ✅ Implementar patrones de inyección de dependencias
- ✅ Diseñar sistemas de configuración dinámica
- ✅ Crear arquitecturas para aplicaciones multi-tenant

## 📖 Contenido Teórico

### 1. Arquitectura de Capas (Layered Architecture)

#### Estructura de capas:
```jsx
// src/
// ├── presentation/     # Capa de presentación (React components)
// ├── application/     # Capa de aplicación (use cases, services)
// ├── domain/          # Capa de dominio (entidades, reglas de negocio)
// ├── infrastructure/  # Capa de infraestructura (APIs, base de datos)
// └── shared/          # Código compartido entre capas

// Domain Layer - Entidades de negocio
// src/domain/entities/User.js
export class User {
  constructor(id, name, email, role) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.role = role;
  }

  canAccess(permission) {
    return this.role.permissions.includes(permission);
  }

  updateProfile(updates) {
    if (updates.name) this.name = updates.name;
    if (updates.email) this.email = updates.email;
  }
}

// Domain Layer - Repositorios (interfaces)
// src/domain/repositories/UserRepository.js
export class UserRepository {
  async findById(id) {
    throw new Error('Método debe ser implementado');
  }

  async save(user) {
    throw new Error('Método debe ser implementado');
  }

  async delete(id) {
    throw new Error('Método debe ser implementado');
  }
}

// Application Layer - Casos de uso
// src/application/useCases/CreateUser.js
export class CreateUserUseCase {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  async execute(userData) {
    // Validar datos
    if (!userData.email || !userData.name) {
      throw new Error('Email y nombre son requeridos');
    }

    // Crear usuario
    const user = new User(
      Date.now().toString(),
      userData.name,
      userData.email,
      userData.role
    );

    // Guardar en repositorio
    const savedUser = await this.userRepository.save(user);

    // Enviar email de bienvenida
    await this.emailService.sendWelcomeEmail(savedUser.email);

    return savedUser;
  }
}

// Infrastructure Layer - Implementación del repositorio
// src/infrastructure/repositories/UserRepositoryImpl.js
export class UserRepositoryImpl extends UserRepository {
  constructor(apiClient) {
    super();
    this.apiClient = apiClient;
  }

  async findById(id) {
    const response = await this.apiClient.get(`/users/${id}`);
    return new User(
      response.data.id,
      response.data.name,
      response.data.email,
      response.data.role
    );
  }

  async save(user) {
    const response = await this.apiClient.post('/users', {
      name: user.name,
      email: user.email,
      role: user.role
    });
    
    return new User(
      response.data.id,
      response.data.name,
      response.data.email,
      response.data.role
    );
  }

  async delete(id) {
    await this.apiClient.delete(`/users/${id}`);
  }
}

// Presentation Layer - React Component
// src/presentation/components/UserForm.jsx
import { useState } from 'react';
import { useCreateUser } from '../hooks/useCreateUser';

function UserForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    role: 'user'
  });

  const { createUser, loading, error } = useCreateUser();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await createUser(formData);
      // Limpiar formulario o redirigir
    } catch (err) {
      console.error('Error al crear usuario:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={formData.name}
        onChange={(e) => setFormData({...formData, name: e.target.value})}
        placeholder="Nombre"
        required
      />
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
        placeholder="Email"
        required
      />
      <select
        value={formData.role}
        onChange={(e) => setFormData({...formData, role: e.target.value})}
      >
        <option value="user">Usuario</option>
        <option value="admin">Administrador</option>
      </select>
      <button type="submit" disabled={loading}>
        {loading ? 'Creando...' : 'Crear Usuario'}
      </button>
      {error && <p className="error">{error}</p>}
    </form>
  );
}
```

### 2. Patrón de Inyección de Dependencias

#### Container de dependencias:
```jsx
// src/infrastructure/container/DependencyContainer.js
class DependencyContainer {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }

  register(name, factory, options = {}) {
    this.services.set(name, { factory, options });
  }

  resolve(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Servicio '${name}' no registrado`);
    }

    if (service.options.singleton) {
      if (!this.singletons.has(name)) {
        this.singletons.set(name, service.factory(this));
      }
      return this.singletons.get(name);
    }

    return service.factory(this);
  }

  getInstance(name) {
    return this.resolve(name);
  }
}

// Configuración del container
// src/infrastructure/container/configureContainer.js
import { DependencyContainer } from './DependencyContainer';
import { UserRepositoryImpl } from '../repositories/UserRepositoryImpl';
import { CreateUserUseCase } from '../../application/useCases/CreateUser';
import { EmailService } from '../services/EmailService';
import { ApiClient } from '../clients/ApiClient';

export function configureContainer() {
  const container = new DependencyContainer();

  // Registrar servicios
  container.register('apiClient', () => new ApiClient(), { singleton: true });
  container.register('emailService', () => new EmailService(), { singleton: true });
  
  container.register('userRepository', (container) => {
    const apiClient = container.getInstance('apiClient');
    return new UserRepositoryImpl(apiClient);
  });

  container.register('createUserUseCase', (container) => {
    const userRepository = container.getInstance('userRepository');
    const emailService = container.getInstance('emailService');
    return new CreateUserUseCase(userRepository, emailService);
  });

  return container;
}

// Hook personalizado que usa el container
// src/presentation/hooks/useCreateUser.js
import { useState } from 'react';
import { useContainer } from './useContainer';

export function useCreateUser() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const container = useContainer();

  const createUser = async (userData) => {
    try {
      setLoading(true);
      setError(null);
      
      const createUserUseCase = container.getInstance('createUserUseCase');
      const result = await createUserUseCase.execute(userData);
      
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { createUser, loading, error };
}

// Provider del container
// src/presentation/providers/ContainerProvider.jsx
import { createContext, useContext } from 'react';

const ContainerContext = createContext();

export function ContainerProvider({ container, children }) {
  return (
    <ContainerContext.Provider value={container}>
      {children}
    </ContainerContext.Provider>
  );
}

export function useContainer() {
  const container = useContext(ContainerContext);
  if (!container) {
    throw new Error('useContainer debe usarse dentro de ContainerProvider');
  }
  return container;
}
```

### 3. Patrón de Eventos (Event-Driven Architecture)

#### Sistema de eventos:
```jsx
// src/shared/events/EventBus.js
class EventBus {
  constructor() {
    this.listeners = new Map();
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  off(event, callback) {
    if (!this.listeners.has(event)) return;
    
    const callbacks = this.listeners.get(event);
    const index = callbacks.indexOf(callback);
    if (index > -1) {
      callbacks.splice(index, 1);
    }
  }

  emit(event, data) {
    if (!this.listeners.has(event)) return;
    
    this.listeners.get(event).forEach(callback => {
      try {
        callback(data);
      } catch (error) {
        console.error(`Error en evento ${event}:`, error);
      }
    });
  }

  once(event, callback) {
    const onceCallback = (data) => {
      callback(data);
      this.off(event, onceCallback);
    };
    this.on(event, onceCallback);
  }
}

// Hook para usar eventos
// src/presentation/hooks/useEventBus.js
import { useEffect, useRef } from 'react';
import { useEventBus } from './useEventBus';

export function useEventBus() {
  const eventBus = useEventBus();
  const listeners = useRef(new Map());

  const on = (event, callback) => {
    eventBus.on(event, callback);
    listeners.current.set(event, callback);
  };

  const off = (event) => {
    const callback = listeners.current.get(event);
    if (callback) {
      eventBus.off(event, callback);
      listeners.current.delete(event);
    }
  };

  useEffect(() => {
    return () => {
      // Limpiar listeners al desmontar
      listeners.current.forEach((callback, event) => {
        eventBus.off(event, callback);
      });
    };
  }, [eventBus]);

  return { on, off, emit: eventBus.emit.bind(eventBus) };
}

// Uso en componentes
// src/presentation/components/UserList.jsx
import { useState, useEffect } from 'react';
import { useEventBus } from '../hooks/useEventBus';

function UserList() {
  const [users, setUsers] = useState([]);
  const { on, off } = useEventBus();

  useEffect(() => {
    // Escuchar eventos de usuario
    on('user:created', handleUserCreated);
    on('user:updated', handleUserUpdated);
    on('user:deleted', handleUserDeleted);

    return () => {
      off('user:created');
      off('user:updated');
      off('user:deleted');
    };
  }, []);

  const handleUserCreated = (user) => {
    setUsers(prev => [...prev, user]);
  };

  const handleUserUpdated = (updatedUser) => {
    setUsers(prev => prev.map(user => 
      user.id === updatedUser.id ? updatedUser : user
    ));
  };

  const handleUserDeleted = (userId) => {
    setUsers(prev => prev.filter(user => user.id !== userId));
  };

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

### 4. Micro-Frontends

#### Configuración de micro-frontends:
```jsx
// src/micro-frontends/container/App.jsx
import { lazy, Suspense } from 'react';
import { MicroFrontendLoader } from './MicroFrontendLoader';

// Lazy loading de micro-frontends
const UserManagement = lazy(() => import('../user-management/UserManagement'));
const ProductCatalog = lazy(() => import('../product-catalog/ProductCatalog'));
const OrderManagement = lazy(() => import('../order-management/OrderManagement'));

function App() {
  return (
    <div className="app">
      <header>
        <h1>Plataforma Empresarial</h1>
        <nav>
          <a href="#users">Usuarios</a>
          <a href="#products">Productos</a>
          <a href="#orders">Órdenes</a>
        </nav>
      </header>

      <main>
        <Suspense fallback={<div>Cargando módulo...</div>}>
          <MicroFrontendLoader
            name="user-management"
            component={UserManagement}
            fallback={<div>Error al cargar gestión de usuarios</div>}
          />
          
          <MicroFrontendLoader
            name="product-catalog"
            component={ProductCatalog}
            fallback={<div>Error al cargar catálogo de productos</div>}
          />
          
          <MicroFrontendLoader
            name="order-management"
            component={OrderManagement}
            fallback={<div>Error al cargar gestión de órdenes</div>}
          />
        </Suspense>
      </main>
    </div>
  );
}

// Loader de micro-frontends
// src/micro-frontends/container/MicroFrontendLoader.jsx
import { useState, useEffect } from 'react';

function MicroFrontendLoader({ name, component: Component, fallback }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Cargar configuración del micro-frontend
    loadMicroFrontendConfig(name)
      .then(() => setIsLoaded(true))
      .catch(err => setError(err));
  }, [name]);

  if (error) return fallback;
  if (!isLoaded) return <div>Cargando {name}...</div>;

  return <Component />;
}

// Configuración de micro-frontends
// src/micro-frontends/config/microFrontendConfig.js
export const microFrontendConfig = {
  'user-management': {
    entry: 'http://localhost:3001/remoteEntry.js',
    scope: 'userManagement',
    module: './UserManagementApp'
  },
  'product-catalog': {
    entry: 'http://localhost:3002/remoteEntry.js',
    scope: 'productCatalog',
    module: './ProductCatalogApp'
  },
  'order-management': {
    entry: 'http://localhost:3003/remoteEntry.js',
    scope: 'orderManagement',
    module: './OrderManagementApp'
  }
};

// Función para cargar configuración
async function loadMicroFrontendConfig(name) {
  const config = microFrontendConfig[name];
  if (!config) {
    throw new Error(`Configuración no encontrada para ${name}`);
  }

  // Aquí se implementaría la lógica de Module Federation
  // o carga dinámica de micro-frontends
  return config;
}
```

### 5. Patrón de Plugins y Extensiones

#### Sistema de plugins:
```jsx
// src/shared/plugins/PluginManager.js
class PluginManager {
  constructor() {
    this.plugins = new Map();
    this.hooks = new Map();
  }

  registerPlugin(name, plugin) {
    if (this.plugins.has(name)) {
      throw new Error(`Plugin '${name}' ya está registrado`);
    }

    this.plugins.set(name, plugin);
    
    // Registrar hooks del plugin
    if (plugin.hooks) {
      Object.keys(plugin.hooks).forEach(hookName => {
        if (!this.hooks.has(hookName)) {
          this.hooks.set(hookName, []);
        }
        this.hooks.get(hookName).push(plugin.hooks[hookName]);
      });
    }

    // Inicializar plugin
    if (plugin.initialize) {
      plugin.initialize();
    }
  }

  unregisterPlugin(name) {
    const plugin = this.plugins.get(name);
    if (!plugin) return;

    // Limpiar hooks
    if (plugin.hooks) {
      Object.keys(plugin.hooks).forEach(hookName => {
        const hooks = this.hooks.get(hookName);
        if (hooks) {
          const index = hooks.indexOf(plugin.hooks[hookName]);
          if (index > -1) {
            hooks.splice(index, 1);
          }
        }
      });
    }

    // Desinicializar plugin
    if (plugin.destroy) {
      plugin.destroy();
    }

    this.plugins.delete(name);
  }

  executeHook(hookName, ...args) {
    const hooks = this.hooks.get(hookName);
    if (!hooks) return;

    return hooks.map(hook => hook(...args));
  }

  getPlugin(name) {
    return this.plugins.get(name);
  }

  getAllPlugins() {
    return Array.from(this.plugins.values());
  }
}

// Ejemplo de plugin
// src/plugins/analytics/AnalyticsPlugin.js
export const AnalyticsPlugin = {
  name: 'analytics',
  
  hooks: {
    'user:login': (user) => {
      // Track user login
      gtag('event', 'login', {
        user_id: user.id,
        method: 'email'
      });
    },
    
    'user:logout': (user) => {
      // Track user logout
      gtag('event', 'logout', {
        user_id: user.id
      });
    }
  },

  initialize() {
    console.log('Analytics plugin inicializado');
  },

  destroy() {
    console.log('Analytics plugin destruido');
  }
};

// Hook para usar plugins
// src/presentation/hooks/usePluginManager.js
import { useContext } from 'react';
import { PluginManagerContext } from '../contexts/PluginManagerContext';

export function usePluginManager() {
  const pluginManager = useContext(PluginManagerContext);
  if (!pluginManager) {
    throw new Error('usePluginManager debe usarse dentro de PluginManagerProvider');
  }
  return pluginManager;
}

// Uso en componentes
// src/presentation/components/UserLogin.jsx
import { usePluginManager } from '../hooks/usePluginManager';

function UserLogin() {
  const pluginManager = usePluginManager();

  const handleLogin = async (credentials) => {
    try {
      const user = await loginUser(credentials);
      
      // Ejecutar hooks de plugins
      pluginManager.executeHook('user:login', user);
      
      // Redirigir al dashboard
      navigate('/dashboard');
    } catch (error) {
      console.error('Error de login:', error);
    }
  };

  return (
    <form onSubmit={handleLogin}>
      {/* Formulario de login */}
    </form>
  );
}
```

### 6. Patrón de Configuración Dinámica

#### Sistema de configuración:
```jsx
// src/shared/config/ConfigurationManager.js
class ConfigurationManager {
  constructor() {
    this.config = new Map();
    this.watchers = new Map();
    this.defaults = new Map();
  }

  set(key, value) {
    const oldValue = this.config.get(key);
    this.config.set(key, value);

    // Notificar cambios
    if (this.watchers.has(key)) {
      this.watchers.get(key).forEach(watcher => {
        watcher(value, oldValue);
      });
    }
  }

  get(key, defaultValue = null) {
    return this.config.get(key) ?? defaultValue;
  }

  watch(key, callback) {
    if (!this.watchers.has(key)) {
      this.watchers.set(key, []);
    }
    this.watchers.get(key).push(callback);

    // Retornar función para desuscribirse
    return () => {
      const callbacks = this.watchers.get(key);
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    };
  }

  setDefaults(defaults) {
    Object.entries(defaults).forEach(([key, value]) => {
      this.defaults.set(key, value);
      if (!this.config.has(key)) {
        this.config.set(key, value);
      }
    });
  }

  reset() {
    this.config.clear();
    this.setDefaults(this.defaults);
  }

  export() {
    return Object.fromEntries(this.config);
  }

  import(config) {
    Object.entries(config).forEach(([key, value]) => {
      this.set(key, value);
    });
  }
}

// Hook para usar configuración
// src/presentation/hooks/useConfiguration.js
import { useState, useEffect } from 'react';
import { useConfigurationManager } from './useConfigurationManager';

export function useConfiguration(key, defaultValue = null) {
  const configManager = useConfigurationManager();
  const [value, setValue] = useState(configManager.get(key, defaultValue));

  useEffect(() => {
    const unsubscribe = configManager.watch(key, (newValue) => {
      setValue(newValue);
    });

    return unsubscribe;
  }, [configManager, key]);

  const setConfigValue = (newValue) => {
    configManager.set(key, newValue);
  };

  return [value, setConfigValue];
}

// Uso en componentes
// src/presentation/components/ThemeToggle.jsx
import { useConfiguration } from '../hooks/useConfiguration';

function ThemeToggle() {
  const [theme, setTheme] = useConfiguration('theme', 'light');

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <button onClick={toggleTheme}>
      Tema actual: {theme}
    </button>
  );
}
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Arquitectura de Capas**
Implementa una aplicación de gestión de productos usando arquitectura de capas con dominio, aplicación e infraestructura.

### **Ejercicio 2: Sistema de Eventos**
Crea un sistema de eventos que permita comunicación entre módulos independientes de una aplicación.

### **Ejercicio 3: Inyección de Dependencias**
Implementa un container de dependencias que gestione servicios y casos de uso de una aplicación.

### **Ejercicio 4: Micro-Frontend Básico**
Crea un micro-frontend simple que se integre con una aplicación principal usando Module Federation.

### **Ejercicio 5: Sistema de Plugins**
Implementa un sistema de plugins que permita extender funcionalidades de una aplicación base.

### **Ejercicio 6: Configuración Dinámica**
Crea un sistema de configuración que permita cambiar comportamientos de la aplicación en tiempo real.

### **Ejercicio 7: Patrón Repository**
Implementa el patrón Repository para diferentes entidades con interfaces y implementaciones.

### **Ejercicio 8: Arquitectura Multi-Tenant**
Diseña una arquitectura que soporte múltiples clientes (tenants) con configuraciones independientes.

### **Ejercicio 9: Sistema de Middleware**
Crea un sistema de middleware que permita interceptar y modificar operaciones de la aplicación.

### **Ejercicio 10: Arquitectura de Monorepo**
Implementa una arquitectura de monorepo con múltiples paquetes y dependencias compartidas.

## 🎯 Proyecto Integrador: Sistema de Gestión Empresarial

### **Descripción del Proyecto**
Construye un sistema de gestión empresarial completo que demuestre todos los patrones arquitectónicos aprendidos, incluyendo micro-frontends, plugins, y configuración dinámica.

### **Requisitos del Proyecto**
- ✅ Arquitectura de capas bien definida
- ✅ Sistema de eventos para comunicación
- ✅ Inyección de dependencias
- ✅ Micro-frontends para módulos independientes
- ✅ Sistema de plugins extensible
- ✅ Configuración dinámica
- ✅ Patrones de diseño empresariales
- ✅ Testing de arquitectura
- ✅ Documentación técnica completa
- ✅ Deployment y CI/CD

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cuáles son las ventajas de la arquitectura de capas?
2. ¿Cómo funciona el patrón de inyección de dependencias?
3. ¿Qué son los micro-frontends y cuándo usarlos?
4. ¿Cómo se implementa un sistema de eventos?
5. ¿Qué es el patrón Repository y cuándo aplicarlo?
6. ¿Cómo se diseña un sistema de plugins?
7. ¿Cuáles son las mejores prácticas para arquitecturas escalables?

### **Criterios de Evaluación**
- ✅ Diseñar arquitecturas escalables
- ✅ Implementar patrones de diseño empresariales
- ✅ Crear micro-frontends funcionales
- ✅ Implementar inyección de dependencias
- ✅ Crear sistemas de eventos
- ✅ Implementar sistemas de plugins
- ✅ Diseñar configuración dinámica
- ✅ Crear documentación técnica

## 🔑 Conceptos Clave a Recordar

- **Arquitectura de capas** separa responsabilidades y facilita mantenimiento
- **Inyección de dependencias** reduce acoplamiento y facilita testing
- **Micro-frontends** permiten desarrollo independiente de módulos
- **Sistema de eventos** facilita comunicación entre componentes
- **Patrón Repository** abstrae el acceso a datos
- **Plugins** permiten extender funcionalidades sin modificar código base
- **Configuración dinámica** permite adaptar la aplicación en tiempo real
- **Arquitectura escalable** debe considerar crecimiento futuro

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre testing avanzado y E2E, incluyendo testing de integración, testing de performance, y herramientas de testing automatizado.

## 📚 Recursos Adicionales

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html)
- [Micro-Frontends](https://micro-frontends.org/)
- [Module Federation](https://webpack.js.org/concepts/module-federation/)
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)

## ✅ Checklist de Competencias

- [ ] Diseño arquitecturas escalables para aplicaciones grandes
- [ ] Implemento patrones de diseño empresariales
- [ ] Creo micro-frontends con React
- [ ] Implemento arquitectura de capas
- [ ] Diseño sistemas de plugins y extensiones
- [ ] Implemento patrones de comunicación entre módulos
- [ ] Creo arquitecturas orientadas a eventos
- [ ] Implemento patrones de inyección de dependencias
- [ ] Diseño sistemas de configuración dinámica
- [ ] Completo el proyecto integrador del sistema de gestión empresarial

---

**¡Excelente progreso! Has dominado la arquitectura avanzada. Estás listo para testing avanzado y E2E.** 🎉
