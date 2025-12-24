# Basic AWS IAM Labs for Beginners
## Easy-to-Understand Guide (No Prior Experience Needed)

---

## What is IAM? (In Simple Words)

**Think of it like a hotel key system:**
- **Root Account** = Master key (opens everything - don't use daily)
- **Users** = Different key cards for different people
- **Permissions/Policies** = What each key card can open
- **Groups** = Sets of similar key cards (all have same access)

**Real example:** In a company, you give:
- Receptionist: Access to door bell and visitor log (limited)
- Manager: Access to office, files, and reports (more access)
- Janitor: Access to storage closets only (very limited)

---

## Lab 1: Understanding Root Account vs Regular Users

### Why This Matters
Imagine you have 1 master key to your house. You wouldn't give it to everyone who visits. Same with AWS root account—it's too powerful to use daily.

### What You'll Learn
- What a root account is
- Why it's dangerous to share
- How to create a safer "user" account

### Step-by-Step Instructions

**Step 1: Sign In to Your AWS Account**
- Go to aws.amazon.com
- Click "Sign In"
- Use your email and password (this is your root account)
- You're now logged in as the account owner

**Step 2: Create a New User**
- In the top right, click your account name → "My Security Credentials"
- In the left menu, click "Users"
- Click "Create User" button
- Enter username: `my-first-user`
- Click "Create User"

**Step 3: Compare Root Account vs New User**
- Root account: Can do EVERYTHING (dangerous!)
- New user: Can't do anything yet (safe!)

**Step 4: Test the Difference**
- Click on your new user name
- Try clicking different services (EC2, S3, etc.)
- You'll see error messages because user has no permissions yet

### Real-World Analogy
| Aspect | Root Account | Regular User |
|--------|-------------|--------------|
| **Access** | Everything | Only what you give them |
| **Who Uses** | Only account owner (rarely) | Developers, employees, apps |
| **Safety** | Risky (if compromised, everything lost) | Safer (limited damage) |
| **Password** | Change it, then forget it | Give to person who needs access |

### Key Takeaway
✅ Create an IAM user for yourself for daily use  
✅ Lock away root account password  
✅ Never share root account credentials  

---

## Lab 2: Giving Permissions with Policies (Making Keys More Useful)

### Why This Matters
A key card that opens nothing is useless. We need to give users permission to do specific things.

### What You'll Learn
- How to give permissions (called "policies")
- Different permission levels
- How to attach permissions to users

### Step-by-Step Instructions

**Step 1: Go Back to Your User**
- In IAM dashboard, click "Users"
- Click on `my-first-user`

**Step 2: Give Permission to Use S3 (File Storage)**
- Click "Add Permissions" button
- Click "Attach Policies Directly"
- In the search box, type: `AmazonS3ReadOnlyAccess`
- Check the box next to "AmazonS3ReadOnlyAccess"
- Click "Next" → "Add Permissions"

**What Just Happened?**
Your user can now READ files from S3 (like viewing files in a shared folder) but CANNOT delete or change them.

**Step 3: Test the Permission**
- Sign out of root account (top right → Sign Out)
- Go to aws.amazon.com
- This time, sign in as:
  - Account ID: (see in IAM dashboard)
  - Username: `my-first-user`
  - Password: (you set this earlier)
- Go to S3 service
- You can now view buckets! (If none exist, that's normal)

**Step 4: Try Something You CAN'T Do**
- Try to delete an S3 bucket (if one exists)
- You'll get "Access Denied" error
- This is good! It means permissions are working

### Real-World Analogy

Imagine a library:
- **ReadOnly Permission** = Can check out and read books, but can't erase words in them
- **Full Access** = Can read, write, erase, and destroy books
- **No Permission** = Can't even enter the library

### Different Permission Levels (Simple Versions)

| Permission | Can Do | Cannot Do |
|-----------|--------|-----------|
| **ReadOnly** | View, list, read files | Delete, modify, create |
| **Write** | Read, create, modify files | Delete files |
| **Admin** | Read, create, modify, delete | Everything allowed |

### Key Takeaway
✅ Policies control what users can do  
✅ Start with read-only permissions  
✅ Only add more permissions when needed  

---

## Lab 3: Creating Groups (Easier Way to Manage Multiple Users)

### Why This Matters
Imagine you have 10 developers who all need the same S3 access. Do you give permission to each person one-by-one? That's tedious! Groups are the shortcut.

### What You'll Learn
- How to create a group
- How to add users to a group
- How groups automatically give permissions

### Step-by-Step Instructions

**Step 1: Create a Group**
- In IAM dashboard, click "User Groups" in left menu
- Click "Create Group"
- Name it: `Developers`
- Click "Create Group"

**Step 2: Add Permissions to the Group**
- You're now in the group page
- Click "Add Permissions" → "Attach Policies"
- Search for: `AmazonEC2FullAccess` (lets people control virtual computers)
- Check the box and click "Add Permissions"

**Step 3: Add a User to the Group**
- In the group page, scroll down to "Users"
- Click "Add Users"
- Select your `my-first-user`
- Click "Add Users"

**Step 4: Test It**
- Sign in as `my-first-user`
- Go to EC2 service
- You should now see EC2 options (previously you couldn't)
- This happened because you're in the "Developers" group!

**Step 5: Add More Users Later (Optional)**
If you create another user next week, just:
- Add them to the "Developers" group
- They automatically get EC2 permission
- No need to set it up again!

### Real-World Analogy

**Without Groups:**
```
Developer 1 → Give S3 permission
Developer 2 → Give S3 permission
Developer 3 → Give S3 permission
Developer 4 → Give S3 permission
...repeat 10 times
```

**With Groups:**
```
Create "Developers" group → Give S3 permission
Add all developers to group → Everyone gets permission automatically
```

### Visual Example

```
┌─────────────────────────────────┐
│     DEVELOPERS GROUP            │
│  (has EC2 Full Access policy)   │
└────────────────┬────────────────┘
                 │
        ┌────────┼────────┐
        │        │        │
     Alice     Bob     Charlie
    (gets EC2  (gets EC2  (gets EC2
     access)   access)    access)
```

### Key Takeaway
✅ Groups save time when managing multiple users  
✅ All group members get the same permissions  
✅ Easy to onboard new team members  

---

## Lab 4: Access Keys (How Apps Login to AWS)

### Why This Matters
Users login with username and password. But apps and scripts need a different way to access AWS. That's what Access Keys are for.

### What You'll Learn
- What Access Keys are
- How to create them (and keep them secret)
- How to use them in a simple script

### Step-by-Step Instructions

**Step 1: Create an Access Key**
- Sign in as `my-first-user`
- In the top right, click account name → "Security Credentials"
- Scroll down to "Access Keys"
- Click "Create Access Key"
- Choose "Command Line Interface (CLI)" → "I understand..."
- Click "Create Access Key"

**You'll see two things:**
- **Access Key ID** (like username) - it's OK if people see this
- **Secret Access Key** (like password) - KEEP THIS SECRET!

**Step 2: Download and Save Securely**
- Click "Download .csv file"
- Save it somewhere safe on your computer
- NEVER share this file or paste it in emails!

**Step 3: Why This is Needed**
Imagine your app needs to upload files to S3:
```
Your App → "Hey AWS, I'm my-first-user, let me upload this file"
AWS → "OK, who are you?"
Your App → Shows Access Key ID and Secret Access Key
AWS → "Great! You're allowed. Here you go!"
```

### Real-World Analogy

| Login Type | Used By | Example |
|-----------|---------|---------|
| **Username + Password** | People | You logging into AWS website |
| **Access Keys** | Applications/Scripts | Your Python script uploading to S3 |
| **MFA (Two-Factor)** | People (extra safety) | Code from phone to confirm login |

### ⚠️ Important Security Rules

**DO:**
- ✅ Treat Access Keys like passwords
- ✅ Rotate them every 90 days (create new ones)
- ✅ Delete old ones when not needed
- ✅ Use them only for the app that needs them

**DON'T:**
- ❌ Put them in email
- ❌ Upload them to GitHub/public sites
- ❌ Share with other people
- ❌ Use root account access keys

### Key Takeaway
✅ Access Keys let apps login to AWS  
✅ Keep them secret (like passwords)  
✅ Give apps only what they need  

---

## Lab 5: Simple Roles (For Apps and Services)

### Why This Matters
Instead of creating access keys for every app, we can use "Roles." Think of a role as a permission blueprint that apps use.

### What You'll Learn
- What roles are
- How to create a role
- How to use it with services like Lambda (serverless apps)

### Step-by-Step Instructions

**Step 1: Create a Role**
- In IAM dashboard, click "Roles" in left menu
- Click "Create Role"
- Under "Select trusted entity," choose "AWS Service"
- Search for and select "Lambda"
- Click "Next"

**What just happened?**
You told AWS: "This role is for Lambda apps"

**Step 2: Add Permissions to the Role**
- In the permissions page, search for: `AmazonS3ReadOnlyAccess`
- Check the box and click "Next"
- Name the role: `LambdaS3ReadRole`
- Click "Create Role"

**Step 3: Understand What We Built**

```
┌──────────────────────────────────────┐
│  LambdaS3ReadRole                    │
│  ├─ Can be used by: Lambda apps      │
│  └─ Permission: Read S3 files        │
└──────────────────────────────────────┘
```

**Step 4: Why This is Better Than Access Keys**

**Old Way (Using Access Keys):**
- Create access key for app
- Put key in app's config file
- If key leaks, anyone can use it forever
- Have to manually rotate keys

**New Way (Using Role):**
- Attach role to app
- AWS automatically handles login (no key to leak)
- AWS auto-rotates credentials every hour
- Much safer!

### Real-World Example

Imagine you hire a photographer for your event:

**Old Way (Access Keys):**
- Give photographer your house key
- They can enter whenever they want
- If key is lost, you have to change your locks

**New Way (Roles):**
- Give photographer a temporary badge
- Badge only works on the day of the event
- Badge automatically expires
- Much safer!

### Key Takeaway
✅ Roles are safer than access keys  
✅ Roles auto-rotate credentials  
✅ Perfect for Lambda, EC2, and other AWS services  

---

## Lab 6: Simple Permission Examples

### Why This Matters
Now let's look at real permission examples so you understand what they do.

### What You'll Learn
- How to read and write simple policies
- Common permission scenarios
- What "Allow" and "Deny" mean

### Example 1: Allow S3 Read-Only

This policy says: "User can view S3 buckets and files, but can't change or delete them"

```
Service: S3
Permission: GetObject (read files)
Permission: ListBucket (view file list)
Resources: Everything
```

**In Plain English:**
"You can look at files in S3 but not touch them"

### Example 2: Allow EC2 Start/Stop (Not Create/Delete)

This policy says: "User can turn EC2 computers on and off, but can't create or delete them"

**In Plain English:**
"You can start and stop servers but can't build new ones or destroy them"

**Why useful?**
- Developers can test their code (start/stop)
- They can't accidentally delete servers
- Saves money (you control who starts expensive resources)

### Example 3: Allow S3 Access Only to One Bucket

This policy says: "User can read from ONLY the 'company-files' bucket, not any other bucket"

**In Plain English:**
"You can access the company-files folder but not the secret-data folder"

**Why useful?**
- Developers access their project files
- They can't accidentally see other projects
- Security and privacy protection

### Example 4: Deny Deletion (Super Important)

This policy says: "No one is allowed to delete S3 files. Ever. Not even admin."

**In Plain English:**
"This is a safety rule: nobody can delete files, period"

**Why useful?**
- Prevents accidental data loss
- Protects against cyber attacks
- Compliance with rules (you must keep data)

### Permission Pyramid (Least to Most Access)

```
        ▲
        │        FULL ACCESS (Dangerous!)
        │       (Create, Read, Modify, Delete)
        │
        │       WRITE ACCESS (More Power)
        │       (Read + Modify, but not Delete)
        │
        │       READ ONLY (Safe)
        │       (Only view, no changes)
        │
        └─────────────────────────────
           START HERE (Least Dangerous)
```

### Key Takeaway
✅ Start with read-only permissions  
✅ Only give more access if needed  
✅ Use "Deny" for critical safety rules  

---

## Lab 7: Two-Factor Authentication (2FA/MFA)

### Why This Matters
Even strong passwords can be guessed or hacked. Two-factor authentication adds an extra lock.

### What You'll Learn
- What 2FA is
- How to set it up
- Why it's important for root account

### Simple Analogy

**One Lock (Password):**
```
Attacker guesses password → Enters your account
```

**Two Locks (Password + 2FA):**
```
Attacker guesses password → Asks for 6-digit code
                            Doesn't have your phone → Can't enter!
```

### Step-by-Step Instructions

**Step 1: Download an Authenticator App**
Choose one:
- Google Authenticator (free)
- Microsoft Authenticator (free)
- Authy (free)

Download to your phone.

**Step 2: Enable 2FA on Root Account**
- Sign in as root (your main account)
- Click on account name → "My Security Credentials"
- Click "Multi-factor Authentication (MFA)"
- Click "Activate MFA"
- Select "Authenticator app"
- Click "Next"

**Step 3: Connect Your Phone**
- Open the authenticator app on your phone
- Scan the QR code from AWS
- The app will show a 6-digit number
- Enter this number in AWS
- Click "Add MFA Device"

**Step 4: Test It**
- Sign out of AWS
- Sign back in with your password
- AWS will ask: "What's your 6-digit code?"
- Open authenticator app and enter the code
- You're in! ✅

### Why This Matters

**Without 2FA:**
- Attacker hacks your password → Full access ❌

**With 2FA:**
- Attacker hacks your password → Needs your phone ✅
- They don't have it → Can't get in ✅

### Key Takeaway
✅ Enable 2FA on root account FIRST  
✅ Enable 2FA on admin accounts  
✅ Keeps account safe from hacking  

---

## Lab 8: Reviewing What You Can Access (Audit)

### Why This Matters
After giving people permissions, you should check what they actually can do. This is called an "audit."

### What You'll Learn
- How to see what a user can do
- How to check if they have too much access
- How to clean up old permissions

### Step-by-Step Instructions

**Step 1: View a User's Permissions**
- In IAM dashboard, click "Users"
- Click on a user (e.g., `my-first-user`)
- Scroll down to "Permissions"
- You'll see all policies attached to this user

**Step 2: Check if They Have Too Much Access**
Ask yourself:
- "Does this person really need this permission?"
- "Can this be taken away?"
- "Is this creating a security risk?"

**Step 3: Remove Unused Permissions**
If a permission is not needed:
- Click the "X" next to the policy
- Click "Detach"
- The permission is removed

**Real Example:**
```
User: John (Developer)
Current Permissions:
├─ AmazonS3FullAccess ✅ (needed for his job)
├─ AmazonRDSAdminAccess ✅ (needed for database work)
├─ IAMFullAccess ❌ (NOT needed - too dangerous!)
└─ AmazonEC2FullAccess ✓ (probably OK)

Action: Remove IAMFullAccess
Reason: John shouldn't create new users (not his job)
```

### Step 4: Document What You Found

Write down:
- User name: John
- Permissions removed: IAMFullAccess
- Date: Today
- Reason: John doesn't manage IAM users

This creates a record for compliance and audits.

### Key Takeaway
✅ Regularly review user permissions  
✅ Remove permissions people don't need  
✅ Keep documentation of changes  

---

## Quick Reference: IAM Checklist for Beginners

### Security Checklist
- [ ] Created IAM user for daily work
- [ ] Root account password is strong and saved safely
- [ ] Enabled 2FA on root account
- [ ] No one shares root account credentials
- [ ] Access keys are kept secret
- [ ] Old access keys are deleted

### User Management Checklist
- [ ] Created groups for different roles
- [ ] Users are in appropriate groups
- [ ] Each group has only needed permissions
- [ ] New users get clear permission instructions
- [ ] Quarterly review of user permissions
- [ ] Deleted users from old employees

### Permission Checklist
- [ ] Users have minimum permissions needed (least privilege)
- [ ] Read-only access is used when possible
- [ ] Admin access is limited to few people
- [ ] Dangerous actions (like delete) require approval
- [ ] Policies are documented
- [ ] Regular audit of who has what access

---

## Common Mistakes to Avoid

❌ **Mistake 1:** Sharing root account credentials
- **Why bad:** If compromised, attacker has full control
- **Do this instead:** Give people IAM users with limited access

❌ **Mistake 2:** Making everyone an admin
- **Why bad:** One person's mistake = everyone can delete everything
- **Do this instead:** Give specific permissions only

❌ **Mistake 3:** Leaving access keys lying around
- **Why bad:** Keys in GitHub = hacker has your AWS access
- **Do this instead:** Use roles instead of keys when possible

❌ **Mistake 4:** Never reviewing permissions
- **Why bad:** People keep access they don't need anymore
- **Do this instead:** Review quarterly and remove old access

❌ **Mistake 5:** Not using 2FA
- **Why bad:** Password alone can be hacked
- **Do this instead:** Enable 2FA on all important accounts

---

## Real-World Scenarios

### Scenario 1: Hiring a New Developer

**Steps:**
1. Create IAM user for them
2. Add them to "Developers" group (they get EC2 + S3 read-only)
3. Create access keys for their laptop
4. Tell them the password (they change it on first login)
5. Enable 2FA on their account

**Time needed:** 10 minutes
**Result:** Developer has access, company is secure

---

### Scenario 2: Cleaning Up When Someone Leaves

**Steps:**
1. Remove user from all groups
2. Delete access keys
3. Delete login password
4. Delete user entirely (after waiting period)

**Time needed:** 5 minutes
**Result:** Ex-employee can't access anything

---

### Scenario 3: Developer Needs More Permissions

**Steps:**
1. Developer asks for EC2 instance deletion permission
2. You check if they really need it
3. If yes: Update "Developers" group policy
4. If no: Explain why they don't need it

**Time needed:** 5 minutes
**Result:** Transparent access control

---

## Summary: What You Learned

| Lab | Topic | Key Idea |
|-----|-------|----------|
| 1 | Root vs Users | Don't use root for daily work |
| 2 | Policies | Give people only what they need |
| 3 | Groups | Manage multiple users easily |
| 4 | Access Keys | Secure way for apps to login |
| 5 | Roles | Even safer than access keys |
| 6 | Permission Examples | Real-world permission scenarios |
| 7 | Two-Factor Auth | Add extra security layer |
| 8 | Audit | Check and cleanup regularly |

---

## Next Steps

After you complete these labs:

1. **Set up your AWS account properly:**
   - Secure root account
   - Create your user account
   - Enable 2FA
   - Set up billing alerts

2. **Practice with real services:**
   - Create an S3 bucket and give people access
   - Create an EC2 server and control who can stop it
   - Try different permission levels

3. **Build good habits:**
   - Review permissions monthly
   - Remove access when someone leaves
   - Document everything
   - Never share secrets

4. **Learn more (when ready):**
   - Advanced policies
   - Cross-account access
   - Audit logging
   - Automation

---

## Help! Something Went Wrong

**Q: I can't access something I should be able to access**
A: Check if your user has the right permissions attached. Ask your admin to check your groups.

**Q: I gave someone access but they can't login**
A: They might not have set a password yet. The account must have a password OR access keys to login.

**Q: I lost my 2FA device**
A: Contact AWS Support with proof of identity. They'll help you regain access.

**Q: Can I undo a permission change?**
A: Yes! Go back to the user/group and re-attach the policy. Changes happen instantly.

**Q: Is there a cheaper way to do this?**
A: IAM is completely free. You only pay for AWS services (EC2, S3, etc.), not for IAM itself.

---

## Contact & Support

- **AWS Support:** Create support case in AWS Console
- **AWS Forums:** Ask community (free)
- **AWS Documentation:** aws.amazon.com/iam/documentation
- **YouTube:** Search "AWS IAM tutorial" (many free videos)

---

**Remember:** Security is not about being perfect. It's about being **better than yesterday** and **safer than last month**. Start small, practice these labs, and build from there! 🚀
