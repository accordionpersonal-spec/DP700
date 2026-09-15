# Azure + Microsoft Fabric Setup — Learning Guide

*A walkthrough of everything we set up, everything that broke, why it broke, and the concepts behind each fix. Written for someone brand new to Azure administration.*

---

## Part 1: The Big Picture (read this first)

Before any of the steps make sense, you need to understand that Microsoft's cloud has **two separate worlds** that talk to each other but have **separate permission systems**:

| World | What it is | Where you manage it |
|---|---|---|
| **Microsoft Entra ID** (formerly Azure AD) | The *identity* world — users, accounts, who you are | portal.azure.com → Entra ID |
| **Azure Subscription** | The *money* world — resources, billing, what you can create | portal.azure.com → Subscriptions |

**The single most important lesson from our whole debugging session:**

> Being an admin in the identity world (Entra) does **not** automatically give you rights in the money world (Subscription). They are two different locks with two different keys.

This one fact explains almost every error we hit. Keep it in mind as you read.

### Simple analogy

Think of it like an office building:

- **Tenant (Entra ID)** = the building itself. It has a directory of everyone who works there.
- **Users** = employees with ID badges.
- **Global Administrator** = the building manager. Can add/remove employees, change building rules.
- **Subscription** = the company credit card used to buy furniture (resources) for the building.
- **Subscription Owner/Contributor** = people authorized to swipe that credit card.

The building manager (Global Admin) can hire people all day, but if their name isn't on the credit card (subscription IAM), they can't buy a single chair. That's exactly what happened to us.

---

## Part 2: The Two Types of Microsoft Accounts

This caused our biggest confusion, so it gets its own section.

### Personal account (MSA — Microsoft Account)
- Example: `accordionpersonal@gmail.com`
- Any email signed up at microsoft.com. Used for Xbox, Outlook.com, personal OneDrive.
- **Can** create an Azure subscription (this is how the whole thing started).
- **Cannot** be used for many enterprise features — including creating a Fabric capacity. Remember the error: *"You cannot create a Microsoft capacity using a personal account."*

### Organizational account (Work/School account)
- Example: `shiva@accordionpersonalgmail.onmicrosoft.com`
- Lives *inside* an Entra tenant. The `.onmicrosoft.com` part is the tenant's default domain.
- This is what Fabric, Power BI, and most enterprise services expect.

### What happened in our case

1. You signed up for Azure with the **gmail** account → Azure auto-created a tenant called "Default Directory" and made the gmail account its Global Admin.
2. You created a second user **inside** that tenant: `shiva@...onmicrosoft.com` → this became your **org account**.
3. Fabric was signed in as the **org account**, but all the *power* (admin role, subscription ownership) still belonged to the **gmail** account.
4. Result: everything failed silently until we transferred rights to the org account.

**Lesson:** always know *which account* you're signed in as, and *which account owns what*. Half of Azure debugging is answering "who am I right now?"

---

## Part 3: Power BI vs Fabric — Licenses vs Capacities

Second biggest source of confusion. These are **two different things that both live at app.fabric.microsoft.com**:

### Licenses (per-user)
- Attached to a **person**. Examples: Free, Power BI Pro, Premium Per User (PPU).
- The **Power BI trial** ("59 days left" in your profile) is a *license* trial. It upgrades *you* as a user.
- Licenses let you *view and share* Power BI content. They do **not** give you compute to run Fabric items.

### Capacities (shared compute)
- A pool of compute power (CPUs, memory) that **workspaces** attach to.
- Fabric items — notebooks, lakehouses, pipelines, warehouses — **require a capacity** to exist. No capacity = "something went wrong" when creating a notebook (our very first error).
- Types:
  - **Fabric Trial capacity** — free 60 days, roughly F64-sized. Started via the "Start Fabric trial" button.
  - **F SKUs (F2–F2048)** — paid capacities created in Azure. What we ended up using.
  - **P SKUs** — old Power BI Premium capacities (being retired).

### The trap we fell into

The profile panel showed **"Power BI trial status: 59 days left"** and we assumed the Fabric trial was active. It wasn't. They are separate trials:

| | Power BI trial | Fabric trial |
|---|---|---|
| What it upgrades | Your user license | Gives you a compute capacity |
| Where it shows | Profile → "trial status" | Admin portal → Capacity settings → **Trial** tab |
| What it unlocks | Sharing reports, PPU features | Creating notebooks, lakehouses, pipelines |

**Lesson:** to verify a Fabric trial exists, don't trust the profile panel. Check **Admin portal → Capacity settings → Trial tab**. Empty tab = no trial, regardless of what buttons say.

---

## Part 4: The Debugging Journey (what broke and why)

This is the actual sequence we went through. Each step is a mini-lesson.

### Symptom 1: "Something went wrong" when creating a notebook
- **Cause:** the workspace had no Fabric capacity behind it. A workspace on Pro license mode can only hold Power BI items.
- **Concept:** every workspace has a **license mode** (Workspace settings → License info). Pro = Power BI only. Trial / Fabric capacity / Premium = Fabric items allowed.
- **Fix attempted:** switch workspace to Trial → but Trial wasn't available, because…

### Symptom 2: "Start Fabric trial" button did nothing (silent fail)
- Clicked activate, picked region, page refreshed, button still there.
- **Debugging principle used:** *verify state at the source, not the UI.* We checked Admin portal → Capacity settings → Trial tab → **empty** → confirmed the trial genuinely never got created (not just a stale button).
- **Root cause discovered along the way:** the Admin portal only showed 5 menu items (Capacity settings, Help + support…). A real admin sees ~20 (Tenant settings, Users, Audit logs…). **The stripped-down menu was the tell** that Fabric didn't consider shiva@ an admin.

### Symptom 3: shiva@ wasn't a Global Admin
- **Cause:** the gmail account created the tenant, so *it* was Global Admin. The shiva@ user was created later as a plain user.
- **Fix:** signed into portal.azure.com as gmail → Entra ID → Users → shiva@ → Assigned roles → add **Global Administrator**.
- **Verification:** Fabric Admin portal now showed the full menu including **Tenant settings**. 
- **Concept — Entra roles:** roles like Global Administrator, User Administrator, etc. control what a user can do *within the identity/tenant world*. Global Admin is the master key for the tenant.

### Symptom 4: Trial still wouldn't start even as admin
- Checked the relevant **tenant settings** (Fabric Admin portal → Tenant settings):
  - **"Users can create Fabric items"** → already Enabled ✔
  - **"Users can try Microsoft Fabric paid features"** → already Enabled ✔
- **Concept — tenant settings:** org-wide switches controlling what everyone in the tenant can do in Fabric. Changes take ~15 min to propagate (only matters when you *change* one).
- Trial activation remained broken (this happens on some fresh personal tenants — the trial provisioning path is flaky). So we **stopped fighting it and went around it**.

### Symptom 5: "You cannot create a Microsoft capacity using a personal account"
- Fallback plan: create a real **F2 capacity** in Azure using the $200 free credit.
- First attempt was as the **gmail** account → blocked, because capacity creation requires an **org account** (see Part 2).

### Symptom 6: shiva@ "does not have authorization to perform action ... over scope /subscriptions/..."
- Switched to shiva@ in Azure portal → subscription dropdown showed a red authorization error.
- **This is the Part 1 lesson in action:** shiva@ was now Global Admin (identity world) but had **zero role on the subscription** (money world).
- **Concept — Azure RBAC (Role-Based Access Control):** every subscription/resource group/resource has an **Access control (IAM)** blade where you assign roles:
  - **Owner** — full control *including* granting access to others. Assigning it now requires an extra "condition" step (that's the *"A condition is required for restricted delegation"* error we hit — Azure making you explicitly say whether this owner can delegate roles further).
  - **Contributor** — can create/manage all resources, but **cannot** grant access to others. No condition needed. Perfect for our case.
  - **Reader** — look but don't touch.
- **Fix:** as gmail → Subscriptions → Access control (IAM) → Add role assignment → **Contributor** → member: shiva@ → assign.
- **Gotcha:** roles are baked into your sign-in token. A page refresh isn't enough — you must **sign out and back in** to pick up a new role. (The error message even hints at this: "please refresh your credentials.")

### Resolution: created the F2 capacity
- As shiva@ (fresh sign-in): Create resource → Microsoft Fabric → subscription loaded clean →
  - Resource group: `rg-fabric` (a **resource group** is just a folder for related resources)
  - Region: **Central India** (match your tenant's home region — mismatched regions cause silent failures)
  - Size: **F2** — ⚠ the form defaults to **F64** which costs ~$8,000+/month at full rate. Always click "change size."
- After deployment: Fabric Admin portal → Capacity settings → **Fabric Capacity** tab → capacity listed ✔
- Workspace settings → License info → selected the F2 capacity → Apply → **notebook creation worked**. 🎉

---

## Part 5: Cost Control (do not skip this)

An F2 capacity bills **per second while running** (~$0.36/hour ≈ ~$260/month if left on 24/7). Your $200 credit dies in ~3 weeks if you forget it.

### Pause / Resume (your main lever)
- portal.azure.com → search "Fabric capacity" (or open `rg-fabric`) → click the capacity → **Pause** button on the Overview page.
- **Resume** from the same place before working.
- Paused = compute billing stops within ~a minute. OneLake **storage** still bills (pennies for learning data).
- Habit to build: **resume → work → pause.** Every session.

### Budgets & alerts (your safety net)
- portal.azure.com → Subscriptions → your subscription → **Budgets** → Create.
- Amount: e.g. $50/month, alert at 80% → you get an email long before anything hurts.

### Cost Analysis (your dashboard)
- Same subscription page → **Cost analysis** → daily spend, filterable by resource. Check weekly.

### Capacity Metrics (Fabric-side monitoring)
- In Fabric: Apps → search **"Microsoft Fabric Capacity Metrics"** → install.
- Shows CU (capacity unit) consumption per workspace/item, throttling, smoothing.
- On F2 solo learning you likely won't hit limits unless you hammer Spark — check it if notebooks start queuing.

---

## Part 6: Cheat Sheet

### "Who am I signed in as?" — always check first
- Azure portal / Fabric: click avatar top-right. Note the account **and** the directory/tenant.

### Where things live
| Task | Where |
|---|---|
| Add users, assign Entra roles | portal.azure.com → Entra ID → Users |
| Grant subscription access | portal.azure.com → Subscriptions → IAM |
| Create/pause/resume capacity | portal.azure.com → the Fabric capacity resource |
| Fabric org-wide switches | Fabric → Admin portal → Tenant settings |
| See trial capacities | Admin portal → Capacity settings → **Trial** tab |
| See paid F capacities | Admin portal → Capacity settings → **Fabric Capacity** tab |
| Attach capacity to workspace | Workspace settings → License info |
| Cost tracking | Subscriptions → Cost analysis / Budgets |

### Golden rules learned the hard way
1. **Entra Global Admin ≠ subscription access.** Two worlds, two permission grants.
2. **Personal accounts can't create Fabric capacities.** Use the org (.onmicrosoft.com) account.
3. **New role assigned? Sign out and back in.** Tokens don't refresh themselves.
4. **Power BI trial ≠ Fabric trial.** Verify in Capacity settings → Trial tab, not the profile panel.
5. **A stripped-down Admin portal menu means you're not really admin.**
6. **Region matters.** Match the tenant home region when creating capacities/trials.
7. **Check the SKU before clicking create.** F64 default will nuke your credit.
8. **Pause when done.** Every single time.
9. When the UI fails silently: **F12 → Network tab** → find the failed request → read the response body. The real error is always there.

### Glossary
- **Tenant** — your organization's identity boundary in Entra ID (the "building").
- **Entra ID** — Microsoft's identity service (formerly Azure Active Directory).
- **MSA** — personal Microsoft account (gmail/outlook.com signups).
- **Subscription** — billing container for Azure resources.
- **Resource group** — folder that groups related Azure resources.
- **RBAC / IAM** — role-based access control on subscriptions/resources (Owner, Contributor, Reader).
- **Capacity** — shared compute pool that Fabric workspaces run on.
- **F SKU** — Fabric capacity size tier (F2 smallest → F2048). Billed per second, pausable.
- **CU** — capacity unit, the compute currency inside a capacity.
- **License mode** — per-workspace setting deciding what items it can hold (Pro vs Trial vs Fabric capacity).
- **Tenant settings** — org-wide Fabric feature switches in the Admin portal.
