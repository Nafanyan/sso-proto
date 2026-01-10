# SSO Proto

Protocol Buffers определения для SSO (Single Sign-On) сервиса аутентификации и авторизации.

## Описание

Проект содержит gRPC сервис для управления аутентификацией пользователей и проверки доступа к приложениям.

## Сервисы

### Auth Service

Сервис `Auth` предоставляет следующие методы:

- **Register** — регистрация нового пользователя
- **Login** — вход пользователя и получение токена доступа
- **Validate** — проверка валидности токена и доступа к конкретному приложению (возвращает email пользователя, если токен валиден; иначе возвращает gRPC ошибку)

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
├── Taskfile.yaml             # Задачи для сборки
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
    AppCode:  "my-app", // Используйте app_code вместо app_id
}

resp, err := authClient.Login(ctx, req)
// resp.Token — токен доступа
```

### Пример проверки токена

```go
req := &ssov1.ValidateTokenRequest{
    Token:   "user_token_here",
    AppCode: "my-app",
}

resp, err := authClient.Validate(ctx, req)
if err != nil {
    // Токен невалиден или нет доступа к приложению
    log.Fatal(err)
}
// resp.Email — email пользователя (возвращается только если токен валиден)
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
    AppCode: "my-app", // рекомендуется
}
```

Старый код продолжит работать, но рекомендуется обновить на использование `app_code`.

## Версионирование

Текущая версия: **v1**

Изменения обратно совместимы (backward compatible), поэтому версия остается v1.