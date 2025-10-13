# 🔐 Система Авторизации - Руководство

## Демо-аккаунты

Система предоставляет 4 роли с разными уровнями доступа:

### 👑 Администратор
- **Логин:** `admin`
- **Пароль:** `admin123`
- **Права:** Полный доступ ко всем разделам и функциям

### 💼 Менеджер
- **Логин:** `manager`
- **Пароль:** `manager123`
- **Права:** Заявки, клиенты, склад, техники, график, финансы, отчёты

### 🔧 Техник
- **Логин:** `tech`
- **Пароль:** `tech123`
- **Права:** Просмотр и работа с заявками, дашборд

### 📋 Администратор (Ресепшн)
- **Логин:** `reception`
- **Пароль:** `reception123`
- **Права:** Приём заявок, работа с клиентами, дашборд

---

## Разграничение доступа по разделам

| Раздел | Администратор | Менеджер | Техник | Ресепшн |
|--------|--------------|----------|--------|---------|
| Дашборд | ✅ | ✅ | ✅ | ✅ |
| Заявки | ✅ | ✅ | ✅ (просмотр) | ✅ |
| Клиенты | ✅ | ✅ | ❌ | ✅ |
| Инвентарь | ✅ | ✅ | ❌ | ❌ |
| Склад | ✅ | ✅ | ❌ | ❌ |
| Техники | ✅ | ✅ | ❌ | ❌ |
| График | ✅ | ✅ | ❌ | ❌ |
| Финансы | ✅ | ✅ | ❌ | ❌ |
| Отчёты | ✅ | ✅ | ❌ | ❌ |
| Настройки | ✅ | ❌ | ❌ | ❌ |

---

## Использование в коде

### 1. Защита маршрутов

```tsx
import ProtectedRoute from '@/components/ProtectedRoute';

// Защита всего маршрута
<Route 
  path="/admin" 
  element={
    <ProtectedRoute requiredRole="admin">
      <AdminPanel />
    </ProtectedRoute>
  } 
/>

// Защита по разделу
<Route 
  path="/finance" 
  element={
    <ProtectedRoute section="finance">
      <FinanceSection />
    </ProtectedRoute>
  } 
/>
```

### 2. Условный рендеринг компонентов

```tsx
import Permission from '@/components/Permission';

// Показать только админам
<Permission requiredRole="admin">
  <Button>Удалить всё</Button>
</Permission>

// Показать менеджерам и админам
<Permission requiredRole={['admin', 'manager']}>
  <Button>Экспорт данных</Button>
</Permission>

// С fallback
<Permission requiredRole="admin" fallback={<div>Доступ запрещён</div>}>
  <SensitiveData />
</Permission>
```

### 3. Использование хука useAuth

```tsx
import { useAuth } from '@/contexts/AuthContext';

function MyComponent() {
  const { user, hasPermission, canAccess, logout } = useAuth();

  // Проверка роли
  if (hasPermission('admin')) {
    // Код для админов
  }

  // Проверка доступа к разделу
  if (canAccess('finance')) {
    // Показать финансы
  }

  // Информация о пользователе
  console.log(user.fullName, user.role);

  return (
    <div>
      <h1>Привет, {user.fullName}!</h1>
      <Button onClick={logout}>Выйти</Button>
    </div>
  );
}
```

### 4. Использование хука usePermissions

```tsx
import { usePermissions } from '@/hooks/usePermissions';

function RepairsSection() {
  const { can, isAdmin, isTechnician } = usePermissions();

  return (
    <div>
      {can.createRepair() && (
        <Button>Создать заявку</Button>
      )}
      
      {can.deleteRepair() && (
        <Button variant="destructive">Удалить</Button>
      )}

      {isAdmin && <AdminControls />}
      {isTechnician && <TechnicianView />}
    </div>
  );
}
```

### 5. Программная навигация

```tsx
import { useAuth } from '@/contexts/AuthContext';
import { useNavigate } from 'react-router-dom';

function LoginButton() {
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleLogin = async () => {
    try {
      await login('admin', 'admin123');
      navigate('/');
    } catch (error) {
      console.error('Ошибка входа:', error);
    }
  };

  return <Button onClick={handleLogin}>Войти</Button>;
}
```

---

## Безопасность

### Сессия
- **Длительность:** 24 часа
- **Хранение:** localStorage (зашифрованно)
- **Автовыход:** При истечении сессии

### Проверки
- Все защищённые маршруты проверяются на клиенте
- При отсутствии прав — автоматический редирект
- Неавторизованные пользователи → `/login`

### Иерархия ролей
```
admin (4) > manager (3) > technician (2) > receptionist (1)
```

Роль с бóльшим числом имеет доступ ко всем возможностям нижестоящих ролей.

---

## API AuthService

```typescript
// Вход
await authService.login(username, password);

// Выход
authService.logout();

// Получить текущего пользователя
const user = authService.getCurrentUser();

// Проверить валидность сессии
const isValid = authService.isSessionValid();

// Получить список демо-аккаунтов
const users = authService.getMockUsers();
```

---

## Структура файлов

```
src/
├── contexts/
│   └── AuthContext.tsx          # Контекст авторизации
├── services/
│   └── authService.ts           # Сервис авторизации
├── components/
│   ├── ProtectedRoute.tsx       # HOC для защиты маршрутов
│   ├── Permission.tsx           # Компонент условного рендеринга
│   └── UserMenu.tsx             # Меню пользователя
├── hooks/
│   └── usePermissions.ts        # Хук для работы с правами
├── pages/
│   └── Login.tsx                # Страница входа
└── types/
    └── index.ts                 # Типы User, UserRole, AuthState
```

---

## Расширение системы

### Добавление новой роли

1. Добавить в `src/types/index.ts`:
```typescript
export type UserRole = 'admin' | 'manager' | 'technician' | 'receptionist' | 'newrole';
```

2. Обновить `roleHierarchy` в `AuthContext.tsx`:
```typescript
const roleHierarchy: Record<UserRole, number> = {
  admin: 5,
  newrole: 4,  // Новая роль
  manager: 3,
  technician: 2,
  receptionist: 1
};
```

3. Добавить права в `sectionPermissions`:
```typescript
const sectionPermissions: Record<string, UserRole[]> = {
  newsection: ['admin', 'newrole'],
  // ...
};
```

### Добавление нового раздела

Обновить `sectionPermissions` в `AuthContext.tsx`:
```typescript
const sectionPermissions: Record<string, UserRole[]> = {
  // ...existing
  analytics: ['admin', 'manager'],  // Новый раздел
};
```

---

## Troubleshooting

### Пользователь не может войти
- Проверьте правильность логина/пароля
- Откройте DevTools → Application → Local Storage
- Удалите ключ `auth_user` и попробуйте снова

### Раздел показывает "Доступ запрещён"
- Проверьте роль пользователя в профиле
- Убедитесь, что раздел добавлен в `sectionPermissions`
- Проверьте, что компонент обёрнут в `<ProtectedRoute>`

### Сессия истекает слишком быстро
Измените `SESSION_DURATION` в `authService.ts`:
```typescript
const SESSION_DURATION = 48 * 60 * 60 * 1000; // 48 часов
```

---

## Рекомендации

✅ **DO:**
- Всегда используйте `<ProtectedRoute>` для защиты маршрутов
- Проверяйте права перед важными действиями (удаление, изменение)
- Используйте `<Permission>` для условного отображения UI
- Логируйте действия пользователей для аудита

❌ **DON'T:**
- Не храните пароли в открытом виде
- Не полагайтесь только на клиентскую проверку (добавьте серверную)
- Не показывайте элементы, к которым нет доступа
- Не забывайте обрабатывать истечение сессии
