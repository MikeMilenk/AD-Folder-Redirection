# Folder Redirection (Active Directory) – Step-by-Step Guide

This guide demonstrates how to configure **Folder Redirection** in an Active Directory environment using **Group Policy (GPO)**. Instead of applying the policy to every domain user, access is controlled through a dedicated **security group**.

---
Table of Contents:


---

# 1. Create a Security Group

Open **Active Directory Users and Computers** and create a new security group. This group will be used later to assign permissions to users.
```Right-click on your domain > New > Group```

![Creating Security Group](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/0.png)

## 1.1 Group Name: ```Redirection Group```
![Creating Security Group](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/1.png)


This group will be used for **security filtering**, allowing Folder Redirection to apply only to selected users instead of everyone in the Organizational Unit (OU).

---

# 2. Create the Shared Folder

Go to Server Manager and navigate to:
```
→ File and Storage Services
→ Shares
→ Tasks
→ New Share...
```
![Create New Share](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/2.png)

Configure the share with the following settings:

## 2.1 Select Profile
```
→ SMB Share - Quick
→ Next
```

![Select Profile](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/3.png)


## 2.2 Share Location
```
→ Type a custom path:
→ Browse
```

![Share Location](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/4.png)

```
→ Select the drive (C: in my case)
→ New Folder
→ Select Folder
```
I named the shared folder ```FolderRedirect$```. The **`$`** at the end of the share name makes it a **hidden network share**. It won't appear when users browse the network, but it remains accessible to anyone who knows its UNC path and has the appropriate permissions.

![Create New Folder](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/5.png)
![Select New Folder](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/6.png)


## 2.3 Share Name

It displays both the local folder path and the network path to the shared folder

![Share Name](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/7.png)


## 2.4 Other Settings

```
→ Select "Enable access-based enumeration"
→ "Allow caching of share" is enabled by default
```
Access-Based Enumeration ensures users can only see folders they have permission to access.

![Other Settings](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/8.png)


## 2.5 Permissions

```
→ Customize Permissions...
→ Disable Inheritance
→ Convert inherited permissions into explicit permissions on this object.
```
Why did I disable inheritance? It allows me full manual control and prevents unwanted inherited permissions from parent folders.

![Disable Inheritance](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/9.png)

  ### 2.5.1 Remove the default permissions
  
  Delete the unnecessary ```Users``` entries. In my case, I removed all **Users** security principals to ensure that only explicitly assigned users and groups can access the share.

![Remove the Default Users](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/9.1.png)
![Apply Changes](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/9.2.png)

  ### 2.5.2 Add the Security Group
  
  ```
  → Add
  → Select a principal
  → Enter the name of the security group we created earlier (in my case, "Redirection Group")
  → Check Names (to ensure group is findable and was added)
  → OK
  ```

  Permissions Entry for ```<Security Group>```:
   
   → *Type: ```This Folder only```
   → *Applies to:* ```This Folder only```
   → Select *```Show advanced permissions```* to display the full list of available permissions.
   
In my case, I granted ```Read``` permissions along with ```Create folders / append data```. This allows users to create their own redirected folders while preventing them from modifying the root share. Depending on your environment and requirements, you may also choose to grant ```Write``` permissions. Ususally they do not require ```Full Control``` at the root level.
       
![Add the Security Group](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/10.png)
![Choose Permissions](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/10.1.png)

---

## 2.6 Confirmation

Verify the settings and complete share creation.

![Confirmation](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/11.png)
![Results](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/12.png)

Finish the wizard and verify the share appears under:

```
→ Server Manager
→ File and Storage Services
→ Shares
→ <Your Shared Folder>
```

## 2.7 Verify the UNC Path

Open the shared folder in File Explorer, then click the empty area of the address bar to reveal the full UNC path. Verify that it matches the path you will use for Folder Redirection. This path will be used later in the Group Policy.

![Open UNC Path](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/13.png)
![Verify UNC Path](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/13.1.png)

---

# 3. Create an Organizational Unit (OU)

In **Active Directory Users and Computers**, create a new Organizational Unit.

![Create OU](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/14.png)

**OU Name**: <RedirectFolderOU>

Using a dedicated OU simplifies testing and prevents the policy from affecting unintended users. Once OU created, it will appear in the *Active Directory Users and Computers* domain tree.
![UNC Path](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/15.png)

---

# 4. Create and Link a Group Policy Object (GPO)

Go to **Group Policy Management**:
```
→ Forests: (your domain)
→ Domains
→ <your domain>
→ <your new OU>
→ Right-click the new OU
→ Create a GPO in this domain, and Link it here...
```

![UNC Path](https://github.com/MikeMilenk/AD-Folder-Redirection/blob/0f82289443aceb63f52a304964cf521c93517d2f/Images/16.png)

Name the policy: ```Redirect Folder GPO```. After it is created, the new GPO will appear in the right pane of the **Group Policy Management** console.

---

# 5. Configure Folder Redirection

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
