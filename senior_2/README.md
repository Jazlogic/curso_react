# 🚀 Módulo 8: Testing Avanzado y E2E

## 📚 Descripción del Módulo

En este módulo aprenderás a implementar testing avanzado para aplicaciones React, incluyendo testing de integración, testing end-to-end (E2E) con Cypress, testing de performance, y estrategias de testing para aplicaciones empresariales complejas.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Implementar testing de integración completo
- ✅ Configurar y usar Cypress para E2E testing
- ✅ Crear tests de performance y rendimiento
- ✅ Implementar testing de APIs y servicios
- ✅ Crear tests de accesibilidad avanzados
- ✅ Implementar testing de seguridad básico
- ✅ Configurar testing automatizado en CI/CD
- ✅ Crear estrategias de testing para aplicaciones grandes
- ✅ Implementar testing de micro-frontends
- ✅ Crear tests de regresión automatizados

## 📖 Contenido Teórico

### 1. Testing de Integración

#### Configuración de testing de integración:
```jsx
// src/tests/integration/setup.js
import { configure } from '@testing-library/react';
import { server } from '../mocks/server';

// Configurar testing library
configure({ testIdAttribute: 'data-testid' });

// Configurar MSW para mocking de APIs
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Configurar testing de rutas
export const renderWithRouter = (ui, { route = '/' } = {}) => {
  window.history.pushState({}, 'Test page', route);
  return render(ui, { wrapper: BrowserRouter });
};

// Configurar testing con providers
export const renderWithProviders = (ui, { 
  preloadedState = {}, 
  store = configureStore({
    reducer: rootReducer,
    preloadedState
  })
} = {}) => {
  const Wrapper = ({ children }) => {
    return (
      <Provider store={store}>
        {children}
      </Provider>
    );
  };
  return { store, ...render(ui, { wrapper: Wrapper }) };
};
```

#### Test de integración de flujo completo:
```jsx
// src/tests/integration/UserManagementFlow.test.jsx
import { render, screen, waitFor, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithProviders } from '../setup';
import UserManagement from '../../presentation/components/UserManagement';

describe('User Management Flow', () => {
  test('flujo completo de creación de usuario', async () => {
    const user = userEvent.setup();
    const { store } = renderWithProviders(<UserManagement />);

    // 1. Verificar estado inicial
    expect(screen.getByText('Gestión de Usuarios')).toBeInTheDocument();
    expect(screen.getByText('Crear Usuario')).toBeInTheDocument();

    // 2. Abrir formulario de creación
    const createButton = screen.getByText('Crear Usuario');
    await user.click(createButton);

    // 3. Verificar que el formulario esté visible
    expect(screen.getByTestId('user-form')).toBeInTheDocument();

    // 4. Llenar formulario
    const nameInput = screen.getByTestId('name-input');
    const emailInput = screen.getByTestId('email-input');
    const roleSelect = screen.getByTestId('role-select');

    await user.type(nameInput, 'John Doe');
    await user.type(emailInput, 'john@example.com');
    await user.selectOptions(roleSelect, 'admin');

    // 5. Enviar formulario
    const submitButton = screen.getByTestId('submit-button');
    await user.click(submitButton);

    // 6. Verificar loading state
    expect(screen.getByText('Creando usuario...')).toBeInTheDocument();

    // 7. Verificar que el usuario se creó
    await waitFor(() => {
      expect(screen.getByText('Usuario creado exitosamente')).toBeInTheDocument();
    });

    // 8. Verificar que aparece en la lista
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText('admin')).toBeInTheDocument();

    // 9. Verificar estado del store
    const state = store.getState();
    expect(state.users.items).toHaveLength(1);
    expect(state.users.items[0].name).toBe('John Doe');
  });

  test('flujo de edición de usuario', async () => {
    const user = userEvent.setup();
    
    // Renderizar con usuario preexistente
    const preloadedState = {
      users: {
        items: [
          { id: '1', name: 'John Doe', email: 'john@example.com', role: 'user' }
        ],
        loading: false,
        error: null
      }
    };

    renderWithProviders(<UserManagement />, { preloadedState });

    // 1. Encontrar usuario en la lista
    expect(screen.getByText('John Doe')).toBeInTheDocument();

    // 2. Hacer clic en editar
    const editButton = screen.getByTestId('edit-user-1');
    await user.click(editButton);

    // 3. Verificar formulario de edición
    expect(screen.getByTestId('edit-form')).toBeInTheDocument();
    expect(screen.getByDisplayValue('John Doe')).toBeInTheDocument();

    // 4. Modificar datos
    const nameInput = screen.getByTestId('edit-name-input');
    await user.clear(nameInput);
    await user.type(nameInput, 'John Smith');

    // 5. Guardar cambios
    const saveButton = screen.getByTestId('save-button');
    await user.click(saveButton);

    // 6. Verificar cambios
    await waitFor(() => {
      expect(screen.getByText('Usuario actualizado exitosamente')).toBeInTheDocument();
    });

    expect(screen.getByText('John Smith')).toBeInTheDocument();
  });

  test('flujo de eliminación de usuario', async () => {
    const user = userEvent.setup();
    
    const preloadedState = {
      users: {
        items: [
          { id: '1', name: 'John Doe', email: 'john@example.com', role: 'user' }
        ],
        loading: false,
        error: null
      }
    };

    renderWithProviders(<UserManagement />, { preloadedState });

    // 1. Verificar usuario existe
    expect(screen.getByText('John Doe')).toBeInTheDocument();

    // 2. Hacer clic en eliminar
    const deleteButton = screen.getByTestId('delete-user-1');
    await user.click(deleteButton);

    // 3. Verificar confirmación
    expect(screen.getByText('¿Estás seguro de que quieres eliminar este usuario?')).toBeInTheDocument();

    // 4. Confirmar eliminación
    const confirmButton = screen.getByText('Eliminar');
    await user.click(confirmButton);

    // 5. Verificar usuario eliminado
    await waitFor(() => {
      expect(screen.queryByText('John Doe')).not.toBeInTheDocument();
    });

    expect(screen.getByText('Usuario eliminado exitosamente')).toBeInTheDocument();
  });
});
```

### 2. Testing E2E con Cypress

#### Configuración de Cypress:
```javascript
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    
    setupNodeEvents(on, config) {
      // Configurar plugins
      on('task', {
        log(message) {
          console.log(message);
          return null;
        },
        
        table(message) {
          console.table(message);
          return null;
        }
      });
    },
  },
  
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite'
    }
  }
});

// cypress/support/e2e.js
import './commands';

// Configuración global
beforeEach(() => {
  // Limpiar localStorage y sessionStorage
  cy.clearLocalStorage();
  cy.clearCookies();
  
  // Interceptar requests de analytics para evitar ruido
  cy.intercept('https://www.google-analytics.com/**', {});
  cy.intercept('https://api.segment.io/**', {});
});

// Configurar viewport responsive
Cypress.Commands.add('setViewport', (size) => {
  const sizes = {
    mobile: [375, 667],
    tablet: [768, 1024],
    desktop: [1280, 720],
    large: [1920, 1080]
  };
  
  const [width, height] = sizes[size] || sizes.desktop;
  cy.viewport(width, height);
});

// Comando personalizado para login
Cypress.Commands.add('login', (email = 'test@example.com', password = 'password123') => {
  cy.visit('/login');
  cy.get('[data-testid=email-input]').type(email);
  cy.get('[data-testid=password-input]').type(password);
  cy.get('[data-testid=login-button]').click();
  cy.url().should('include', '/dashboard');
});

// Comando para crear usuario de prueba
Cypress.Commands.add('createTestUser', (userData = {}) => {
  const defaultUser = {
    name: 'Test User',
    email: `test${Date.now()}@example.com`,
    role: 'user',
    ...userData
  };

  cy.request('POST', '/api/users', defaultUser).then((response) => {
    return response.body;
  });
});
```

#### Tests E2E de flujos completos:
```javascript
// cypress/e2e/user-management.cy.js
describe('User Management E2E', () => {
  beforeEach(() => {
    cy.login();
    cy.visit('/users');
  });

  it('should create a new user successfully', () => {
    // 1. Navegar a la página de usuarios
    cy.get('[data-testid=user-list]').should('be.visible');
    
    // 2. Hacer clic en crear usuario
    cy.get('[data-testid=create-user-button]').click();
    
    // 3. Verificar formulario
    cy.get('[data-testid=user-form]').should('be.visible');
    
    // 4. Llenar formulario
    const testEmail = `test${Date.now()}@example.com`;
    cy.get('[data-testid=name-input]').type('Test User');
    cy.get('[data-testid=email-input]').type(testEmail);
    cy.get('[data-testid=role-select]').select('admin');
    
    // 5. Enviar formulario
    cy.get('[data-testid=submit-button]').click();
    
    // 6. Verificar éxito
    cy.get('[data-testid=success-message]')
      .should('be.visible')
      .and('contain', 'Usuario creado exitosamente');
    
    // 7. Verificar en lista
    cy.get('[data-testid=user-list]')
      .should('contain', 'Test User')
      .and('contain', testEmail)
      .and('contain', 'admin');
  });

  it('should edit an existing user', () => {
    // 1. Crear usuario de prueba
    cy.createTestUser().then((user) => {
      cy.visit('/users');
      
      // 2. Encontrar usuario en lista
      cy.get(`[data-testid=user-row-${user.id}]`).should('be.visible');
      
      // 3. Hacer clic en editar
      cy.get(`[data-testid=edit-user-${user.id}]`).click();
      
      // 4. Modificar datos
      cy.get('[data-testid=edit-name-input]')
        .clear()
        .type('Updated User Name');
      
      // 5. Guardar cambios
      cy.get('[data-testid=save-button]').click();
      
      // 6. Verificar cambios
      cy.get('[data-testid=success-message]')
        .should('contain', 'Usuario actualizado exitosamente');
      
      cy.get('[data-testid=user-list]')
        .should('contain', 'Updated User Name');
    });
  });

  it('should delete a user with confirmation', () => {
    // 1. Crear usuario de prueba
    cy.createTestUser().then((user) => {
      cy.visit('/users');
      
      // 2. Encontrar usuario
      cy.get(`[data-testid=user-row-${user.id}]`).should('be.visible');
      
      // 3. Hacer clic en eliminar
      cy.get(`[data-testid=delete-user-${user.id}]`).click();
      
      // 4. Verificar modal de confirmación
      cy.get('[data-testid=delete-confirmation-modal]')
        .should('be.visible')
        .and('contain', '¿Estás seguro de que quieres eliminar este usuario?');
      
      // 5. Confirmar eliminación
      cy.get('[data-testid=confirm-delete-button]').click();
      
      // 6. Verificar usuario eliminado
      cy.get(`[data-testid=user-row-${user.id}]`).should('not.exist');
      
      cy.get('[data-testid=success-message]')
        .should('contain', 'Usuario eliminado exitosamente');
    });
  });

  it('should handle form validation errors', () => {
    // 1. Abrir formulario
    cy.get('[data-testid=create-user-button]').click();
    
    // 2. Intentar enviar sin datos
    cy.get('[data-testid=submit-button]').click();
    
    // 3. Verificar errores de validación
    cy.get('[data-testid=name-error]')
      .should('be.visible')
      .and('contain', 'El nombre es requerido');
    
    cy.get('[data-testid=email-error]')
      .should('be.visible')
      .and('contain', 'El email es requerido');
    
    // 4. Llenar solo nombre
    cy.get('[data-testid=name-input]').type('Test User');
    cy.get('[data-testid=submit-button]').click();
    
    // 5. Verificar que solo queda error de email
    cy.get('[data-testid=name-error]').should('not.exist');
    cy.get('[data-testid=email-error]')
      .should('be.visible')
      .and('contain', 'El email es requerido');
  });

  it('should search and filter users', () => {
    // 1. Crear múltiples usuarios
    const users = [
      { name: 'Alice Admin', role: 'admin' },
      { name: 'Bob User', role: 'user' },
      { name: 'Charlie Manager', role: 'manager' }
    ];
    
    users.forEach(user => {
      cy.createTestUser(user);
    });
    
    cy.visit('/users');
    
    // 2. Buscar por nombre
    cy.get('[data-testid=search-input]').type('Alice');
    cy.get('[data-testid=user-list]')
      .should('contain', 'Alice Admin')
      .and('not.contain', 'Bob User')
      .and('not.contain', 'Charlie Manager');
    
    // 3. Filtrar por rol
    cy.get('[data-testid=role-filter]').select('admin');
    cy.get('[data-testid=user-list]')
      .should('contain', 'Alice Admin')
      .and('not.contain', 'Bob User');
    
    // 4. Limpiar filtros
    cy.get('[data-testid=clear-filters-button]').click();
    cy.get('[data-testid=user-list]')
      .should('contain', 'Alice Admin')
      .and('contain', 'Bob User')
      .and('contain', 'Charlie Manager');
  });
});
```

### 3. Testing de Performance

#### Tests de performance con React DevTools:
```jsx
// src/tests/performance/PerformanceTests.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Profiler } from 'react';
import UserList from '../../presentation/components/UserList';

describe('Performance Tests', () => {
  test('renderizado inicial es rápido', () => {
    const onRender = jest.fn();
    const users = Array.from({ length: 100 }, (_, i) => ({
      id: i.toString(),
      name: `User ${i}`,
      email: `user${i}@example.com`
    }));

    render(
      <Profiler id="UserList" onRender={onRender}>
        <UserList users={users} />
      </Profiler>
    );

    // Verificar que el renderizado inicial sea rápido
    const renderTime = onRender.mock.calls[0][1];
    expect(renderTime).toBeLessThan(100); // Menos de 100ms
  });

  test('re-renderizados son optimizados', () => {
    const onRender = jest.fn();
    const { rerender } = render(
      <Profiler id="UserList" onRender={onRender}>
        <UserList users={[{ id: '1', name: 'User 1', email: 'user1@example.com' }]} />
      </Profiler>
    );

    // Primer renderizado
    const firstRenderTime = onRender.mock.calls[0][1];
    
    // Re-renderizar con los mismos datos
    rerender(
      <Profiler id="UserList" onRender={onRender}>
        <UserList users={[{ id: '1', name: 'User 1', email: 'user1@example.com' }]} />
      </Profiler>
    );

    // El re-renderizado debería ser más rápido
    const secondRenderTime = onRender.mock.calls[1][1];
    expect(secondRenderTime).toBeLessThan(firstRenderTime);
  });

  test('virtualización funciona para listas grandes', () => {
    const largeUserList = Array.from({ length: 10000 }, (_, i) => ({
      id: i.toString(),
      name: `User ${i}`,
      email: `user${i}@example.com`
    }));

    const startTime = performance.now();
    render(<UserList users={largeUserList} />);
    const renderTime = performance.now() - startTime;

    // Con virtualización, el renderizado debería ser rápido
    expect(renderTime).toBeLessThan(500); // Menos de 500ms
    
    // Verificar que solo se rendericen elementos visibles
    const visibleItems = screen.getAllByTestId(/user-row/);
    expect(visibleItems.length).toBeLessThan(100); // Solo elementos visibles
  });
});
```

#### Tests de performance con Cypress:
```javascript
// cypress/e2e/performance.cy.js
describe('Performance Tests', () => {
  it('should load dashboard within performance budget', () => {
    cy.login();
    
    // Medir tiempo de carga
    cy.visit('/dashboard', {
      onBeforeLoad: (win) => {
        win.performance.mark('navigation-start');
      }
    });
    
    cy.get('[data-testid=dashboard-content]').should('be.visible');
    
    cy.window().then((win) => {
      win.performance.mark('navigation-end');
      win.performance.measure('dashboard-load', 'navigation-start', 'navigation-end');
      
      const measure = win.performance.getEntriesByName('dashboard-load')[0];
      expect(measure.duration).to.be.lessThan(3000); // Menos de 3 segundos
    });
  });

  it('should handle large data sets efficiently', () => {
    cy.login();
    
    // Crear muchos usuarios para probar rendimiento
    const createUsers = Array.from({ length: 100 }, (_, i) => 
      cy.createTestUser({ name: `User ${i}`, email: `user${i}@example.com` })
    );
    
    cy.wrap(createUsers).then(() => {
      const startTime = performance.now();
      
      cy.visit('/users');
      cy.get('[data-testid=user-list]').should('be.visible');
      
      cy.window().then((win) => {
        const loadTime = performance.now() - startTime;
        expect(loadTime).to.be.lessThan(2000); // Menos de 2 segundos
        
        // Verificar que la lista sea interactiva
        cy.get('[data-testid=search-input]').type('User 1');
        cy.get('[data-testid=user-list]').should('contain', 'User 1');
      });
    });
  });

  it('should maintain good performance during user interactions', () => {
    cy.login();
    cy.visit('/users');
    
    // Medir tiempo de respuesta de búsqueda
    cy.get('[data-testid=search-input]').then(($input) => {
      const startTime = performance.now();
      
      cy.wrap($input).type('test');
      
      cy.get('[data-testid=search-results]').should('be.visible').then(() => {
        const searchTime = performance.now() - startTime;
        expect(searchTime).to.be.lessThan(500); // Menos de 500ms
      });
    });
  });
});
```

### 4. Testing de APIs y Servicios

#### Tests de servicios con MSW:
```jsx
// src/tests/api/UserService.test.jsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { render, screen, waitFor } from '@testing-library/react';
import { UserService } from '../../infrastructure/services/UserService';

const server = setupServer(
  // GET /api/users
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: '1', name: 'John Doe', email: 'john@example.com' },
        { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
      ])
    );
  }),

  // POST /api/users
  rest.post('/api/users', (req, res, ctx) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
      return res(
        ctx.status(400),
        ctx.json({ error: 'Name and email are required' })
      );
    }
    
    return res(
      ctx.status(201),
      ctx.json({
        id: Date.now().toString(),
        name,
        email,
        role: 'user'
      })
    );
  }),

  // PUT /api/users/:id
  rest.put('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    const { name, email } = req.body;
    
    return res(
      ctx.json({
        id,
        name,
        email,
        role: 'user'
      })
    );
  }),

  // DELETE /api/users/:id
  rest.delete('/api/users/:id', (req, res, ctx) => {
    return res(ctx.status(204));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserService API Tests', () => {
  let userService;

  beforeEach(() => {
    userService = new UserService();
  });

  test('should fetch users successfully', async () => {
    const users = await userService.getUsers();
    
    expect(users).toHaveLength(2);
    expect(users[0]).toEqual({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    });
  });

  test('should create user successfully', async () => {
    const userData = {
      name: 'New User',
      email: 'newuser@example.com'
    };
    
    const createdUser = await userService.createUser(userData);
    
    expect(createdUser).toMatchObject({
      name: 'New User',
      email: 'newuser@example.com',
      role: 'user'
    });
    expect(createdUser.id).toBeDefined();
  });

  test('should handle validation errors', async () => {
    const invalidUserData = {
      name: '',
      email: ''
    };
    
    await expect(userService.createUser(invalidUserData))
      .rejects
      .toThrow('Name and email are required');
  });

  test('should update user successfully', async () => {
    const updates = {
      name: 'Updated Name',
      email: 'updated@example.com'
    };
    
    const updatedUser = await userService.updateUser('1', updates);
    
    expect(updatedUser).toMatchObject({
      id: '1',
      name: 'Updated Name',
      email: 'updated@example.com'
    });
  });

  test('should delete user successfully', async () => {
    await expect(userService.deleteUser('1')).resolves.not.toThrow();
  });

  test('should handle network errors gracefully', async () => {
    // Simular error de red
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res.networkError('Failed to connect');
      })
    );
    
    await expect(userService.getUsers()).rejects.toThrow('Failed to connect');
  });
});
```

### 5. Testing de Accesibilidad Avanzado

#### Tests de accesibilidad con jest-axe:
```jsx
// src/tests/accessibility/AccessibilityTests.test.jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import UserForm from '../../presentation/components/UserForm';

expect.extend(toHaveNoViolations);

describe('Accessibility Tests', () => {
  test('UserForm should meet accessibility standards', async () => {
    const { container } = render(<UserForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  test('should have proper ARIA labels', () => {
    render(<UserForm />);
    
    // Verificar labels asociados
    const nameInput = screen.getByLabelText('Nombre completo');
    const emailInput = screen.getByLabelText('Correo electrónico');
    
    expect(nameInput).toBeInTheDocument();
    expect(emailInput).toBeInTheDocument();
    
    // Verificar que los inputs tengan IDs únicos
    expect(nameInput).toHaveAttribute('id');
    expect(emailInput).toHaveAttribute('id');
    expect(nameInput.id).not.toBe(emailInput.id);
  });

  test('should handle keyboard navigation properly', async () => {
    const user = userEvent.setup();
    render(<UserForm />);
    
    // Navegación con Tab
    const nameInput = screen.getByLabelText('Nombre completo');
    const emailInput = screen.getByLabelText('Correo electrónico');
    const submitButton = screen.getByRole('button', { name: 'Crear Usuario' });
    
    await user.tab();
    expect(nameInput).toHaveFocus();
    
    await user.tab();
    expect(emailInput).toHaveFocus();
    
    await user.tab();
    expect(submitButton).toHaveFocus();
  });

  test('should announce errors to screen readers', async () => {
    const user = userEvent.setup();
    render(<UserForm />);
    
    // Intentar enviar sin datos
    const submitButton = screen.getByRole('button', { name: 'Crear Usuario' });
    await user.click(submitButton);
    
    // Verificar que los errores tengan aria-live
    const errorMessages = screen.getAllByRole('alert');
    expect(errorMessages).toHaveLength(2);
    
    // Verificar que los campos con error tengan aria-invalid
    const nameInput = screen.getByLabelText('Nombre completo');
    const emailInput = screen.getByLabelText('Correo electrónico');
    
    expect(nameInput).toHaveAttribute('aria-invalid', 'true');
    expect(emailInput).toHaveAttribute('aria-invalid', 'true');
  });

  test('should support high contrast mode', () => {
    render(<UserForm />);
    
    // Verificar que los elementos tengan suficiente contraste
    const submitButton = screen.getByRole('button', { name: 'Crear Usuario' });
    
    // Simular modo de alto contraste
    document.documentElement.style.setProperty('--color-primary', '#000000');
    document.documentElement.style.setProperty('--color-background', '#ffffff');
    
    // Verificar que el botón sea visible
    expect(submitButton).toBeVisible();
  });
});
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Testing de Integración Completo**
Implementa tests de integración para un flujo completo de gestión de productos con CRUD, búsqueda y filtros.

### **Ejercicio 2: E2E Testing con Cypress**
Crea tests E2E para una aplicación de e-commerce que incluya registro, login, búsqueda, carrito y checkout.

### **Ejercicio 3: Testing de Performance**
Implementa tests de performance para componentes que manejen listas grandes y operaciones costosas.

### **Ejercicio 4: Testing de APIs**
Crea tests completos para servicios de API usando MSW y testing de diferentes escenarios de error.

### **Ejercicio 5: Testing de Accesibilidad**
Implementa tests de accesibilidad para formularios complejos y componentes interactivos.

### **Ejercicio 6: Testing de Seguridad**
Crea tests que verifiquen validación de inputs, autenticación y autorización.

### **Ejercicio 7: Testing de Micro-Frontends**
Implementa tests E2E para aplicaciones con micro-frontends y comunicación entre módulos.

### **Ejercicio 8: Testing de Regresión**
Crea tests automatizados que detecten regresiones en funcionalidades críticas.

### **Ejercicio 9: Testing de CI/CD**
Configura pipelines de testing automatizado con diferentes entornos y reportes.

### **Ejercicio 10: Testing de Stress**
Implementa tests que verifiquen el comportamiento de la aplicación bajo carga.

## 🎯 Proyecto Integrador: Plataforma de E-learning

### **Descripción del Proyecto**
Construye una plataforma de e-learning completa con testing exhaustivo que incluya testing de integración, E2E, performance y accesibilidad.

### **Requisitos del Proyecto**
- ✅ Sistema de cursos y lecciones
- ✅ Gestión de usuarios y roles
- ✅ Sistema de progreso y evaluaciones
- ✅ Tests de integración para flujos completos
- ✅ Tests E2E con Cypress
- ✅ Tests de performance y accesibilidad
- ✅ Testing de APIs y servicios
- ✅ Coverage de testing > 95%
- ✅ Pipeline de CI/CD completo
- ✅ Reportes de testing automatizados

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Cuál es la diferencia entre testing unitario, de integración y E2E?
2. ¿Cómo se configura Cypress para testing E2E?
3. ¿Qué es MSW y cómo se usa para testing de APIs?
4. ¿Cómo se implementan tests de performance?
5. ¿Qué son los tests de accesibilidad y por qué son importantes?
6. ¿Cómo se integra testing en pipelines de CI/CD?
7. ¿Cuáles son las mejores prácticas para testing de aplicaciones grandes?

### **Criterios de Evaluación**
- ✅ Implementar testing de integración completo
- ✅ Configurar y usar Cypress para E2E
- ✅ Crear tests de performance
- ✅ Implementar testing de APIs
- ✅ Crear tests de accesibilidad avanzados
- ✅ Configurar testing en CI/CD
- ✅ Alcanzar coverage > 95%
- ✅ Crear reportes de testing

## 🔑 Conceptos Clave a Recordar

- **Testing de integración** verifica que múltiples componentes trabajen juntos
- **E2E testing** simula el comportamiento real del usuario
- **Cypress** es una herramienta moderna para testing E2E
- **MSW** permite mockear APIs para testing
- **Testing de performance** verifica que la app sea rápida
- **Testing de accesibilidad** asegura que la app sea usable para todos
- **CI/CD** automatiza el proceso de testing
- **Coverage** mide qué tan completo es el testing

## 🚀 Vista Previa del Siguiente Nivel

En el siguiente módulo aprenderás sobre performance y optimización avanzada, incluyendo bundle analysis, Core Web Vitals, y técnicas de optimización para aplicaciones de producción.

## 📚 Recursos Adicionales

- [Cypress Documentation](https://docs.cypress.io/)
- [MSW - Mock Service Worker](https://mswjs.io/)
- [Jest-Axe](https://github.com/nickcolley/jest-axe)
- [Testing Library Best Practices](https://testing-library.com/docs/guiding-principles)
- [Performance Testing](https://web.dev/performance-testing/)

## ✅ Checklist de Competencias

- [ ] Implemento testing de integración completo
- [ ] Configuro y uso Cypress para E2E testing
- [ ] Creo tests de performance y rendimiento
- [ ] Implemento testing de APIs y servicios
- [ ] Creo tests de accesibilidad avanzados
- [ ] Implemento testing de seguridad básico
- [ ] Configuro testing automatizado en CI/CD
- [ ] Creo estrategias de testing para aplicaciones grandes
- [ ] Implemento testing de micro-frontends
- [ ] Completo el proyecto integrador de la plataforma de e-learning

---

**¡Excelente progreso! Has dominado testing avanzado y E2E. Estás listo para performance y optimización.** 🎉
