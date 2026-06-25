# Azure IAM Notes 

---

## 1. What is IAM?

IAM stands for **Identity and Access Management**.

It controls **who can do what, and where** in Azure.

---

## 2. The Golden Formula

Every IAM assignment is just three things combined:

```
Principal  +  Role  +  Scope  =  Access
(WHO)         (WHAT)   (WHERE)
```

**Example:**
> Priya (user) + Contributor (role) + rg-data-dev (resource group) = She can create, edit, delete anything inside rg-data-dev

---

## 3. Principal — WHO is asking?

A principal is the identity that wants access. There are 4 types:

| Type | What it is | Example |
|---|---|---|
| User | A human with a login | priya@company.com |
| Group | A team of users | DataEngTeam |
| Managed Identity | An app/service with no password | ADF pipeline |
| Service Principal | An app with a client secret | CI/CD tool |

> **Tip:** Always assign roles to **Groups**, not individual users. When someone joins/leaves, just add/remove them from the group.

---

## 4. Role — WHAT can they do?

A role is a set of permissions. The 3 most important built-in roles:

| Role | Can create/edit/delete? | Can manage IAM? | Notes |
|---|---|---|---|
| Owner | Yes | Yes | Full control |
| Contributor | Yes | No | Most common for engineers |
| Reader | No | No | View only |

There are also **data-specific roles** like:
- `Storage Blob Data Contributor` — read/write blob data only
- `Key Vault Secrets User` — read secrets only

---

## 5. Scope — WHERE does it apply?

Scope is where the permission is active. There are 4 levels, from biggest to smallest:

```
Management Group  (optional, for large orgs)
    └── Subscription  (billing unit)
            └── Resource Group  (logical folder)
                    └── Resource  (one specific service)
```

**Key rule: permissions flow downward automatically.**

If you assign Contributor at the **Subscription** level → the person gets Contributor access to every Resource Group and every Resource inside that subscription.

If you assign Reader at one **Resource Group** → they can only view that group, nothing else.

---

## 6. Inheritance — How Permissions Flow Down

```
Subscription (Priya = Contributor assigned here)
    │
    ├── rg-ingestion  ← Priya = Contributor (inherited, automatic)
    │       ├── ADF Pipeline    ← Priya = Contributor (inherited)
    │       └── Storage Account ← Priya = Contributor (inherited)
    │
    └── rg-serving  ← Priya = Contributor (inherited, automatic)
            ├── Databricks      ← Priya = Contributor (inherited)
            └── SQL Database    ← Priya = Contributor (inherited)
```

> Assign **high** = access **everywhere** below.  
> Assign **low** = access to **only that one thing**.

---

## 7. Common Beginner Mistakes

### Mistake 1 — Too much access
Giving someone **Owner at Subscription** when they only need to run ADF pipelines.

**Fix:** Assign **Contributor at rg-ingestion only**.

### Mistake 2 — Assigning to individuals
Creating separate role assignments for Priya, Raj, and Meera each.

**Fix:** Create a group `DataEngTeam`, assign the role to the group once. Add/remove people from the group as needed.

---

## 8. Real Data Engineering Example

| Principal | Role | Scope | Result |
|---|---|---|---|
| DataEngTeam (group) | Contributor | rg-ingestion | Team can build/manage all pipelines |
| AnalystGroup (group) | Reader | rg-serving | Analysts can view dashboards, can't break anything |
| ADF Managed Identity | Storage Blob Data Contributor | Storage Account | ADF can read/write data — no password needed |

---

## 9. Quick Cheat Sheet

| Term | One-liner |
|---|---|
| Principal | WHO is getting access |
| Role | WHAT they are allowed to do |
| Scope | WHERE the permission applies |
| Owner | Full control, including changing others' access |
| Contributor | Create/edit/delete, but cannot touch IAM |
| Reader | View only, cannot change anything |
| Managed Identity | Best way to give apps access — no password |
| Inheritance | Permissions assigned high flow down automatically |

---

## 10. How to Assign a Role in the Azure Portal (Demo Steps)

1. Go to **Azure Portal → Resource Groups**
2. Click on your resource group (e.g. `rg-data-dev`)
3. In the left menu, click **Access control (IAM)**
4. Click **+ Add → Add role assignment**
5. Choose a **Role** (e.g. Contributor)
6. Choose **Members** → select a user or group
7. Click **Review + assign**

Done! That one action = one complete IAM assignment (Principal + Role + Scope).
