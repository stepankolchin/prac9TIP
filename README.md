## Практическое занятие №9. Реализация регистрации и входа пользователей. Хэширование паролей с bcrypt. Колчин Степан Сергеевич ЭФМО-02-25.

**Окружение**

- Go: go version go1.25.3 windows/amd64

- Docker: Docker Desktop (Docker version 28.5.1, build e180ab8)

- ОС: Windows 11 (с использованием WSL2 - MINGW64_NT-10.0-19045)

- Git: git version 2.51.0.windows.1

- GORM ORM: v1.31.1 

- GORM POSTGRES DRIVER: v1.6.0 

**Скриншоты**


`201 Created` - Регистрация

<img width="561" height="128" alt="image" src="https://github.com/user-attachments/assets/334a9099-45a1-42af-a3d2-fbeb0053435a" />


`200 OK` - Вход верный

<img width="563" height="129" alt="image" src="https://github.com/user-attachments/assets/b76cf247-e4c5-4227-b0b2-5aff3a634b02" />


`401 Unauthorized` - Вход неверный 

<img width="556" height="134" alt="image" src="https://github.com/user-attachments/assets/7636ddb1-dba4-41f9-91ec-f468f257ea5b" />


`409 Confict` - повторная регистрация

<img width="562" height="141" alt="image" src="https://github.com/user-attachments/assets/64edb49f-41c4-476a-9ec0-cd6d6a1076d2" />


**Фрагменты кода**

- Обработчик `Register` место, где вызывается `bcrypt.GenerateFromPassword`  
[auth.go](./internal/http/handlers/auth.go)

```GO
func (h *AuthHandler) Register(w http.ResponseWriter, r *http.Request) {
// ...

// bcrypt hash
    hash, err := bcrypt.GenerateFromPassword([]byte(in.Password), h.BcryptCost)
    if err != nil {
        writeErr(w, http.StatusInternalServerError, "hash_failed"); return
    }
```

- Обработчик `Login` место, где вызывается `bcrypt.CompareHashAndPassword`

[auth.go](./internal/http/handlers/auth.go)

```GO
func (h *AuthHandler) Login(w http.ResponseWriter, r *http.Request) {
//...

if bcrypt.CompareHashAndPassword([]byte(u.PasswordHash), []byte(in.Password)) != nil {
        writeErr(w, http.StatusUnauthorized, "invalid_credentials"); return
    }
```

**SQL/миграции**

- `AutoMigrate()` GORM

[user_repo.go](./internal/repo/user_repo.go)

```GO
func (r *UserRepo) AutoMigrate() error {
    return r.db.AutoMigrate(&core.User{})
}
```

[main.go](./cmd/api/main.go)

```GO
users := repo.NewUserRepo(db)
if err := users.AutoMigrate(); err != nil { 
    log.Fatal("migrate:", err) 
}
```

**Команды запуска**

- Созданиие БД на `Docker`
```bsh
docker run --name pz9-postgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=123 -e POSTGRES_DB=pz9 -p 5432:5432 -d postgres:18
```

- Запуск проекта от корневого `009-practice`
```bash
export DB_DSN="postgres://user:123@localhost:5432/pz9?sslmode=disable"
go run cmd/api/main.go
```

**Краткие выводы**

1. `Почему нельзя хранить пароли в открытом виде?`

Есть множество причин, почему нельзя хранить пароли в открытом виде:

Уточека БД - злоумышленники смогут получить доступ к паролям с одной БД

Использования одного и того же пароля у пользователя влечет за собой утечкой информации на других платформах

Базовая проверка безопасности - любой аудит сразу заметит подобную уязвимость

2. `Почему bcrypt`

Встроенная соль (salt) - защита от **rainbow table** атак

Адаптивная сложность через параметр `cost` - устойчивость к `brute-force`

Замедленное выполнение
