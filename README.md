# SSO Proto

Protocol Buffers определения для SSO (Single Sign-On) сервиса аутентификации и авторизации.

## Описание

Проект содержит gRPC сервис для управления аутентификацией пользователей и проверки доступа к приложениям.

## Сервисы

### Auth Service

Сервис `Auth` предоставляет следующие методы:

- **Register** — регистрация нового пользователя
- **Login** — вход пользователя и получение токена доступа
- **Logout** — выход пользователя из приложения (по email и app_code)
- **Validate** — проверка валидности токена и доступа к приложению (возвращает `success`; поле `email` помечено как deprecated)
- **AllowAccess** — *deprecated*: используйте **Login** вместо этого метода
- **RevokeAccess** — *deprecated*: используйте **Logout** вместо этого метода
- **GrantAccess** — *deprecated*: используйте **AllowAccess** (который в свою очередь заменён на **Login**)

## Структура проекта

```
sso-proto/
├── proto/
│   └── sso/
│       └── sso.proto          # Определения protobuf
├── gen/
│   └── go/
│       └── sso/               # Сгенерированный Go код
├── go.mod                     # Go зависимости
├── go.sum                     # Контрольные суммы зависимостей
├── Taskfile.yaml              # Задачи для сборки
└── README.md
```

## Установка зависимостей

```bash
go mod download
```

## Генерация кода

Для генерации Go кода из proto файлов используйте Task:

```bash
task generate
# или
task gen
```

Эта команда выполнит:
```bash
protoc -I proto proto/sso/*.proto \
  --go_out=./gen/go/ \
  --go_opt=paths=source_relative \
  --go-grpc_out=./gen/go/ \
  --go-grpc_opt=paths=source_relative
```

### Требования

- [protoc](https://grpc.io/docs/protoc-installation/) — компилятор Protocol Buffers
- [protoc-gen-go](https://github.com/golang/protobuf/tree/main/protoc-gen-go) — плагин для генерации Go кода
- [protoc-gen-go-grpc](https://github.com/grpc/grpc-go) — плагин для генерации gRPC кода

Установка плагинов:
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## Использование

### Пример регистрации пользователя

```go
req := &ssov1.RegisterRequest{
    Email:    "user@example.com",
    Password: "secure_password",
}

resp, err := authClient.Register(ctx, req)
// resp.UserId — ID зарегистрированного пользователя
```

### Пример входа

```go
req := &ssov1.LoginRequest{
    Email:    "user@example.com",
    Password: "secure_password",
    AppCode:  "my-app",
}

resp, err := authClient.Login(ctx, req)
// resp.Token — токен доступа
```

### Пример выхода

```go
req := &ssov1.LogoutRequest{
    Email:   "user@example.com",
    AppCode: "my-app",
}

resp, err := authClient.Logout(ctx, req)
// resp.Success — true при успешном выходе
```

### Пример проверки токена

```go
req := &ssov1.ValidateTokenRequest{
    Token:   "user_token_here",
    AppCode: "my-app",
}

resp, err := authClient.Validate(ctx, req)
if err != nil {
    // Ошибка gRPC (токен невалиден или нет доступа)
    log.Fatal(err)
}
// resp.Success — true, если токен валиден и есть доступ к приложению
// resp.Email — deprecated, будет удалён; ориентируйтесь на success
```

### Пример разрешения доступа (deprecated)

> **Примечание:** метод `AllowAccess` помечен как deprecated. Используйте **Login** для входа и выдачи токена.

```go
req := &ssov1.AllowAccessRequest{
    Email:   "user@example.com",
    AppCode: "my-app",
}

resp, err := authClient.AllowAccess(ctx, req)
// resp.AppCode — код приложения
```

### Пример отзыва доступа (deprecated)

> **Примечание:** метод `RevokeAccess` помечен как deprecated. Используйте **Logout** для выхода пользователя.

```go
req := &ssov1.RevokeAccessRequest{
    Email:   "user@example.com",
    AppCode: "my-app",
}

resp, err := authClient.RevokeAccess(ctx, req)
// resp.AppCode — код приложения
```

## Миграция с app_id на app_code

Поле `app_id` (int32) помечено как `deprecated`. Используйте `app_code` (string) для новых реализаций.

**Старый подход (deprecated):**
```go
req := &ssov1.LoginRequest{
    AppId: 123, // deprecated
}
```

**Новый подход:**
```go
req := &ssov1.LoginRequest{
    AppCode: "my-app",
}
```

## Миграция с GrantAccess на AllowAccess и на Login

- Метод **GrantAccess** помечен как deprecated → используйте **AllowAccess**.
- Метод **AllowAccess** также помечен как deprecated → для выдачи доступа используйте **Login** (вход пользователя в приложение и получение токена).

**Рекомендуемый подход:**
```go
req := &ssov1.LoginRequest{
    Email:   "user@example.com",
    Password: "password",
    AppCode:  "my-app",
}
resp, err := authClient.Login(ctx, req)
// resp.Token — токен для доступа к приложению
```

## Миграция с RevokeAccess на Logout

Метод **RevokeAccess** помечен как deprecated. Используйте **Logout** для выхода пользователя из приложения.

**Рекомендуемый подход:**
```go
req := &ssov1.LogoutRequest{
    Email:   "user@example.com",
    AppCode: "my-app",
}
resp, err := authClient.Logout(ctx, req)
// resp.Success — результат операции
```

## ValidateTokenResponse: email deprecated

В ответе **ValidateTokenResponse** поле `email` помечено как deprecated и будет удалено. Используйте поле **success** (bool) для проверки валидности токена.

## Версионирование

Текущая версия: **v1**

Изменения обратно совместимы (backward compatible), поэтому версия остаётся v1.
