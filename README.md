# Folder Redirection (Active Directory) – Step-by-Step Guide

This guide demonstrates how to configure **Folder Redirection** in an Active Directory environment using **Group Policy (GPO)**. Instead of applying the policy to every domain user, access is controlled through a dedicated **security group**.

---
Table of Contents:


---

# 1. Create a Security Group

Open **Active Directory Users and Computers** and create a new security group.

**Group Name**

```
Redirection Group
```

This group will be used for **security filtering**, allowing Folder Redirection to apply only to selected users instead of everyone in the Organizational Unit (OU).

---

# 2. Create the Shared Folder

Navigate to:

```
Server Manager
→ File and Storage Services
→ Shares
→ Tasks
→ New Share
```

Configure the share with the following settings:

| Setting        | Value                 |
| -------------- | --------------------- |
| Profile        | SMB Share – Quick     |
| Share Location | Custom Path           |
| Folder Path    | `C:\Folder Redirect$` |

The **`$`** at the end of the folder name creates a **hidden network share**, preventing it from appearing during normal network browsing.

---

# 3. Configure Share Settings

Use the following share name:

```
FolderRedirect$
```

Copy the UNC path:

```text
\\DomainController\FolderRedirect$
```

Enable:

* ✅ Access-Based Enumeration (ABE)

### Why Enable ABE?

Access-Based Enumeration ensures users can only see folders they have permission to access.

Example:

* User **Mike** only sees:

```
\\DomainController\FolderRedirect$\Mike
```

* Other users' folders remain hidden.

---

# 4. Configure NTFS Permissions

Select **Customize Permissions**.

## Disable Inheritance

Choose:

```
Convert inherited permissions into explicit permissions
```

### Why?

* Preserves existing permissions
* Allows full manual control
* Prevents unwanted inherited permissions from parent folders

---

## Remove Default Permissions

Remove unnecessary **Users** entries.

### Why?

This prevents all domain users from accessing the root folder unless explicitly authorized.

---

## Add the Security Group

Add:

```
Redirection Group
```

Configure permissions:

* **Applies to:** This folder only
* Permissions:

  * ✅ Read
  * ✅ List Folder / Read Data

### Why?

Users only need access to the root folder so Windows can automatically create their personal redirected folders.

They do **not** require Full Control at the root level.

---

# 5. Complete Share Creation

Finish the wizard and verify the share appears under:

```
Server Manager
→ File and Storage Services
→ Shares
```

---

# 6. Verify the UNC Path

Open the shared folder in File Explorer and verify the UNC path:

```text
\\DomainController\FolderRedirect$
```

This path will be used later in the Group Policy.

---

# 7. Create an Organizational Unit (OU)

In **Active Directory Users and Computers**, create a new Organizational Unit.

**OU Name**

```
Redirect Folder OU
```

Using a dedicated OU simplifies testing and prevents the policy from affecting unintended users.

---

# 8. Create and Link a Group Policy Object

Right-click the new OU and select:

```
Create a GPO in this domain, and Link it here
```

Name the policy:

```
Folder Redirection Policy
```

---

# 9. Configure Folder Redirection

Navigate to:

```
User Configuration
→ Policies
→ Windows Settings
→ Folder Redirection
```

Configure **Desktop** and repeat the same process for **Documents**.

### Target Tab

Setting:

```
Basic – Redirect everyone's folder to the same location
```

Target Folder Location:

```
Create a folder for each user under the root path
```

Root Path:

```text
\\DomainController\FolderRedirect$
```

---

### Settings Tab

Enable:

* ✅ Redirect the folder back to the local user profile location when the policy is removed

This allows Windows to restore the original folder locations if the GPO is later removed.

---

# 10. Add Users

Add users to the policy using one of the following methods:

### Option 1

Add users to the **Redirection Group**.

### Option 2

Create new users directly inside:

```
Redirect Folder OU
```

### Option 3

Move existing users into the OU.

---

# 11. Apply the Policy

On the client computer, run:

```powershell
gpupdate /force
```

Optional verification:

```powershell
gpresult /r
```

---

# 12. Verify Folder Redirection

After the user signs in, Desktop and Documents should automatically redirect to:

```text
\\DomainController\FolderRedirect$\Username\Desktop
```

```text
\\DomainController\FolderRedirect$\Username\Documents
```

Expected behavior:

* Personal folders are created automatically.
* User data is stored on the server.
* Files remain available through Folder Redirection.
* Depending on Offline Files configuration, synchronization icons may appear.

---

# Key Concepts

| Feature                            | Purpose                                                                      |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| **Access-Based Enumeration (ABE)** | Hides folders users don't have permission to access.                         |
| **Converted NTFS Inheritance**     | Allows full manual control of permissions while preserving existing entries. |
| **Security Group Filtering**       | Applies Folder Redirection only to selected users.                           |
| **Folder Redirection**             | Stores user data on the server instead of the local computer.                |
