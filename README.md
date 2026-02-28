# 📘 MRPproject — Spring Boot Backend & Guide Complet

> **Projet** : Spring Boot + Angular + SQL Server + JWT + GraphQL  
> **Dernière mise à jour** : Février 2026

This module implements the backend API for the MRPVAL system using Spring Boot, GraphQL, JWT authentication and SQL Server. It is designed to work with the Angular frontend located in `../Angular/hikmaprj`.

---

## 🧩 Features

- GraphQL API with schema defined under `src/main/resources/graphql/schema.graphqls`
- JWT-based authentication with role support
- User management (create, update, activate/deactivate)
- CRUD operations for business entities (BOM, CDE, Stock, Pdp, etc.)
- Redis caching and SQL Server persistence
- Audit logging using aspect-oriented programming

---

## 🗂️ Structure du Projet

```
mrpval/
├── Springboot/
│   └── hikmaproject/        ← Backend Spring Boot (Java 17)
│       ├── src/main/resources/graphql/schema.graphqls
│       ├── src/main/resources/application.properties
│       └── src/main/java/com/example/hikmaproject/
│           ├── Config/
│           │   ├── SecurityConfig.java
│           │   └── WebConfig.java
│           ├── services/
│           │   ├── AuthService.java
│           │   └── UserService.java
│           └── Security/
│               ├── CustomUserDetailsService.java
│               └── UserPrincipal.java
└── Angular/
    └── hikmaprj/            ← Frontend Angular
```

---

## ⚙️ Prérequis

| Outil | Version |
|-------|---------|
| Java JDK | 17 |
| Maven | latest |
| Node.js / npm | npm 11+ |
| Angular CLI | `npm install -g @angular/cli` |
| SQL Server | localhost:1433 |
| Redis | localhost:6379 |

---

## 🚀 Lancer le Projet

### Backend Spring Boot

```bash
cd Springboot/hikmaproject
mvn clean package
mvn spring-boot:run
```

> Ou depuis IntelliJ : **Run HikmaprojectApplication**

Access GraphiQL playground at: `http://localhost:8080/graphiql`

### Frontend Angular

```bash
cd C:\Users\LENOVO\Desktop\mrpval\Angular\hikmaprj
npx ng serve
# ou si ng est dans le PATH :
ng serve
```

> **⚠️ IMPORTANT** : Toujours ouvrir un **nouveau terminal** après avoir installé Angular CLI !

### Building a JAR

```bash
mvn clean package
```

The runnable jar will be in `target/hikmaproject-0.0.1-SNAPSHOT.jar`.

### Running Tests

```bash
mvn test
```

---

## 🔧 Configuration

Edit `src/main/resources/application.properties` to adjust:

```properties
# Base de données
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=hikmapfeDB;trustServerCertificate=true
spring.datasource.username=imen
spring.datasource.password=imen123

# JWT
jwt.secret=Cv1yTh6A29GmvwumCLvok0IRKMcHV6eSfzTQT8cdLbLLj0K5Te3JWyr2U95J9it2
jwt.expiration=86400000  # 24h en millisecondes

# Redis
spring.redis.host=localhost
spring.redis.port=6379

# Mail
spring.mail.host=smtp.gmail.com
spring.mail.port=587
```

---

## 📄 Fichiers Importants

| Fichier | Rôle |
|---------|------|
| `WebConfig.java` | CORS & static resource configuration |
| `SecurityConfig.java` | Spring Security settings |
| `AuthService.java` | Authentication logic (JWT generation) |
| `UserService.java` | User CRUD and activation/deactivation |
| `application.properties` | Environment settings |
| `schema.graphqls` | GraphQL schema — doit rester synchronisé avec les DTOs Java |

---

## 👤 Créer un Utilisateur Admin

### Option A — Via GraphiQL

```graphql
mutation {
  createUser(input: {
    username: "admin",
    email: "admin@hikma.com",
    password: "admin123",
    roleIds: [1]
  }) {
    id
    username
    message
  }
}
```

### Option B — Directement en Base SQL (étape par étape)

**Étape 1 — Générer un Hash BCrypt**

Créer temporairement `TestHash.java` dans le projet :

```java
package com.example.hikmaproject;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class TestHash {
    public static void main(String[] args) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        System.out.println(encoder.encode("admin123"));
    }
}
```

Clic droit → **Run 'TestHash.main()'** → copier le hash affiché.  
**Supprimer le fichier après usage.**

**Étape 2 — Insérer en Base**

```sql
-- 1. Créer le rôle (si inexistant)
INSERT INTO [hikmapfeDB].[dbo].[Roles] (name) 
VALUES ('ROLE_ADMIN')

-- 2. Créer l'utilisateur (remplacer le hash par celui généré)
INSERT INTO [hikmapfeDB].[dbo].[users1] (active, email, password, username)
VALUES (
    1,
    'admin@hikma.com',
    '$2a$10$HASH_GENERE_ICI',
    'admin'
)

-- 3. Lier le rôle à l'utilisateur
INSERT INTO [hikmapfeDB].[dbo].[user_roles] (user_id, role_id)
VALUES (
    (SELECT id FROM [hikmapfeDB].[dbo].[users1] WHERE username = 'admin'),
    (SELECT id FROM [hikmapfeDB].[dbo].[Roles] WHERE name = 'ROLE_ADMIN')
)
```

**Étape 3 — Vérifier**

```sql
SELECT u.id, u.username, u.active, r.name as role
FROM [hikmapfeDB].[dbo].[users1] u
LEFT JOIN [hikmapfeDB].[dbo].[user_roles] ur ON u.id = ur.user_id
LEFT JOIN [hikmapfeDB].[dbo].[Roles] r ON ur.role_id = r.id
WHERE u.username = 'admin'
```

✅ Le résultat doit montrer : `active = 1` et `role = ROLE_ADMIN`

---

## ❗ Problèmes Fréquents & Solutions

---

### 1. 🔴 `ng` n'est pas reconnu comme commande

**Symptôme :**
```
'ng' n'est pas reconnu en tant que commande interne ou externe
```

**Solutions (dans l'ordre) :**

```bash
# Option A — le plus simple
npx ng serve

# Option B — vérifier et ajouter npm au PATH
npm config get prefix
# Ajouter le résultat dans : Paramètres Windows → Variables d'environnement → Path
# Redémarrer le terminal
```

---

### 2. 🔴 Port déjà utilisé (Spring Boot / JMX)

**Symptôme :**
```
java.net.BindException: Address already in use: bind
Port already in use: 54078
```

**Solution :**
```bash
# Trouver le PID
netstat -ano | findstr :54078

# Tuer le processus avec le VRAI PID trouvé
taskkill /PID [VRAI_PID] /F
```

Ou : **Task Manager → Détails → java.exe → Fin de tâche**

---

### 3. 🔴 "Account is deactivated. Please contact administrator."

**Cause :** Le champ `active = 0` en base de données.

**Solution :**
```sql
UPDATE [hikmapfeDB].[dbo].[users1] 
SET active = 1 
WHERE username = 'admin'
```

> Code concerné dans `AuthService.java` :
> ```java
> if (!user.isActive()) {
>     throw new RuntimeException("Account is deactivated...");
> }
> ```

---

### 4. 🔴 GraphQL Unauthorized (Login échoue)

**Causes possibles (dans l'ordre à vérifier) :**

1. Compte désactivé → voir problème #3
2. Mauvais mot de passe hashé en base → regénérer via `TestHash.java`
3. Rôle non lié à l'utilisateur

**Vérifier les rôles :**
```sql
SELECT u.username, u.active, r.name as role
FROM [hikmapfeDB].[dbo].[users1] u
LEFT JOIN [hikmapfeDB].[dbo].[user_roles] ur ON u.id = ur.user_id
LEFT JOIN [hikmapfeDB].[dbo].[Roles] r ON ur.role_id = r.id
WHERE u.username = 'admin'
```

Si `role = NULL` → lier le rôle :
```sql
INSERT INTO [hikmapfeDB].[dbo].[user_roles] (user_id, role_id)
VALUES (
    (SELECT id FROM [hikmapfeDB].[dbo].[users1] WHERE username = 'admin'),
    (SELECT id FROM [hikmapfeDB].[dbo].[Roles] WHERE name = 'ROLE_ADMIN')
)
```

---

## ✅ Checklist Avant de Lancer

- [ ] SQL Server démarré sur le port 1433
- [ ] Redis démarré sur le port 6379
- [ ] Aucune instance Java en cours sur le port utilisé
- [ ] L'utilisateur admin existe en base avec `active = 1`
- [ ] Le rôle est bien lié à l'utilisateur dans `user_roles`
- [ ] Ouvrir un nouveau terminal après installation de `@angular/cli`

---

## 🔐 Identifiants de Test

| Champ | Valeur |
|-------|--------|
| Username | `admin` |
| Password | `admin123` |
| Email | `admin@hikma.com` |

---

## 🚀 Déploiement

Deploy the JAR to any Java container or run directly. Ensure environment variables or properties override sensitive data (database credentials, JWT secret).

---

## 📝 Notes

- Keep GraphQL schema and corresponding Java DTOs synchronized. When adding new entities, update repository and resolver accordingly.
- BCrypt hashes are **one-way** — impossible to reverse. Always regenerate via `TestHash.java` if a password is lost.
- Redis **must be running** before starting Spring Boot, otherwise the application will crash on startup.

