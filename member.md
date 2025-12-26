# Module Member – Documentation complète

## 📌 Objectif

Le module Member gère l’ensemble du cycle de vie des utilisateurs :
création, connexion, permissions, mot de passe et activation.

Aucune inscription publique n’est autorisée.

---

## 🔐 Connexion

Conditions pour se connecter :

- Le compte existe
- Le compte est activé (`activated = true`)
- Le mot de passe est valide

Si `mustChangePassword = true`, l’utilisateur est redirigé
vers la création ou la modification de son mot de passe.

---

## 👥 Rôles et permissions

### ROLE_ADMINISTRATEUR
- Accès total
- Création, modification, suppression de tous les membres
- Attribution de tous les rôles
- Activation / désactivation des comptes

### ROLE_METTEUR_EN_SCENE
- Création de membres
- Accès aux membres de son théâtre
- Activation / désactivation
- Ne peut pas modifier les profils

### ROLE_MEMBER
- Accès à son espace personnel
- Modification de son profil
- Modification de son mot de passe

---

## ➕ Création d’un membre

La création est possible uniquement par :
- Administrateur
- Metteur en scène

État initial :
- activated = false
- mustChangePassword = true
- mot de passe provisoire

---

## 🔑 Mot de passe

### Premier mot de passe
- Controller : `MemberPasswordController`
- FormType : `FirstPasswordType`

Champs :
- Mot de passe provisoire
- Nouveau mot de passe
- Confirmation

Effets :
- Mot de passe enregistré
- activated = true
- mustChangePassword = false

### Modification du mot de passe
- Même controller
- Même formulaire
- Twig conditionnel

---

## 🛡️ Sécurité

### Voter : MemberVoter
- CREATE → admin, metteur en scène
- EDIT → uniquement son propre profil

### Subscriber
- `ForcePasswordChangeSubscriber`
- Redirection forcée si mustChangePassword = true

---

## 📊 Diagramme de flux – Connexion

## 📐 Diagramme UML – Module Member

```mermaid
classDiagram
    class Member {
        +int id
        +string email
        +string password
        +array roles
        +bool activated
        +bool mustChangePassword
    }

    class MemberVoter {
        +supports()
        +voteOnAttribute()
    }

    class MemberCrudController {
        +configureFields()
        +configureActions()
        +createIndexQueryBuilder()
    }

    class MemberPasswordController {
        +password()
    }

    class FirstPasswordType {
        +buildForm()
    }

    class ForcePasswordChangeSubscriber {
        +onKernelRequest()
    }

    Member --> MemberVoter : secured by
    Member --> MemberCrudController : managed by
    Member --> MemberPasswordController : password
    MemberPasswordController --> FirstPasswordType : uses
    ForcePasswordChangeSubscriber --> Member : checks

## 🔄 Diagramme de séquence – Login & mot de passe

```mermaid
sequenceDiagram
    participant User
    participant Security
    participant Subscriber
    participant PasswordController
    participant Database

    User->>Security: login
    Security->>Database: check credentials
    Database-->>Security: valid user

    Security->>Subscriber: kernel.request
    Subscriber->>Database: mustChangePassword ?

    alt mustChangePassword = true
        Subscriber-->>User: redirect to password form
        User->>PasswordController: submit new password
        PasswordController->>Database: update password
        PasswordController-->>User: redirect to space
    else mustChangePassword = false
        User-->>User: access space member
    end
