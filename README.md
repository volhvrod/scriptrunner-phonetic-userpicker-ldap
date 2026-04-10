# 🚀 Custom LDAP Phonetic User Picker for Jira (ScriptRunner 10.7+)

**A powerful, production-ready custom picker for Jira Data Center that searches Active Directory users by phonetic name, restricted to a specific AD group.**

Built specifically for **ScriptRunner 10.7+ on Jira DC**, this script turns a standard Jira custom field into a smart, dynamic picker that:
- Searches LDAP in real-time as the user types
- Prioritizes phonetic display names (perfect for non-English or complex names)
- Only returns users who are members of your chosen security group
- Seamlessly converts LDAP users to real Jira `ApplicationUser` objects
- Provides beautiful rendering in issue view and edit screens

Perfect for helpdesk, HR, project management, or any team that needs a clean, fast user selector tied to your corporate directory.

---

## ✨ Features

- **Phonetic-first search** — Uses `msDS-PhoneticDisplayName` for superior matching as example
- **Group-restricted results** — Only users from the configured AD group appear
- **Real-time LDAP search** — Minimum 2 characters, lightning-fast results
- **Jira-native integration** — Returns proper `ApplicationUser` objects via UPN
- **Smart fallbacks** — Shows phonetic name if available, otherwise Jira `displayName`
- **Full picker lifecycle** — Supports search, option rendering, item loading, and issue view display
- **Production hardened** — Clean error handling, null-safety, and ScriptRunner best practices
- **Easy to customize** — Just change two constants at the top

---

## 📋 Requirements

| Requirement                  | Details |
|-----------------------------|---------|
| **Jira**                    | Data Center (Server not supported) |
| **ScriptRunner**            | 10.7 or higher |
| **LDAP Connection**         | Pre-configured ScriptRunner LDAP pool |
| **Active Directory**        | You can change `msDS-PhoneticDisplayName` to your localname atrr |
| **Permissions**             | ScriptRunner must be able to query the LDAP pool and read Jira users |

---

## ⚙️ Configuration

All settings are at the very top of the script — no other changes required.

```groovy
// ===================== Settings =====================
def ldapPoolName = 'Your_LDAP_Pool'                        // ← Name of your ScriptRunner LDAP connection pool
def groupDN = 'OU=ou,DC=domain,DC=net'                     // ← Full Distinguished Name of the target group
// =====================================================
```
---

## How to change settings
- Open the script in ScriptRunner → Script Console or your custom field configuration.
- Update ldapPoolName to match your LDAP pool name.
- Update groupDN to the exact DN of the group whose members should appear in the picker.
- Save and test.

---

## 📥 Installation & Setup

### Create the Custom Field
- Go to Jira Administration → Issues → Custom fields.
- Click Create custom field.
- Choose Select List (single choice) or ScriptRunner → Custom Field (depending on your version).
- Name it (e.g. "Phonetic User Picker" or "Assigned Specialist").

### Attach the Script
- In the custom field configuration, paste the entire script provided below into the appropriate ScriptRunner section (usually under "Picker Script" or "Script" tab for custom pickers).
- Configure Contexts & Screens
- Add the field to the relevant projects and issue types.
- Add it to Create, Edit, and View screens.

### Test
- Create a new issue.
- Start typing a phonetic name (at least 2 characters).
- You should see matching users from the specified AD group.

---

## 🔍 How the Script Works (Detailed Breakdown)

The script follows ScriptRunner’s official Custom Picker contract and implements all four required closures:
1. - fetchPhonetic (Helper)

```groovy
def fetchPhonetic = { String lookupValue -> ... }
```
- Looks up a user by userPrincipalName or sAMAccountName.
- Returns the msDS-PhoneticDisplayName attribute.
- Used when loading already-selected values.

---

2. -  search (Core Search Logic)
- Triggered every time the user types in the picker.
- Requires minimum 2 characters.
- Builds a precise LDAP filter:
```ldap
(&(objectCategory=person)(objectClass=user)(memberOf=GROUP_DN)(msDS-PhoneticDisplayName=*input*))
```
- Searches the entire subtree.
- Converts results to Jira ApplicationUser objects using Users.getByName().
- Returns a list of maps containing both the Jira user and phonetic name.

---

3. - toOption
- Converts each search result into a PickerOption object.
- Label = phonetic name (if exists) or Jira displayName.
- Uses the highlight closure provided by ScriptRunner to highlight matching text.
- Value = Jira username (for storage).

---

4. - getItemFromId
- Called when Jira loads an already-selected value (e.g. on issue view/edit).
- Fetches the Jira user + phonetic name again.

---

5. - renderItemViewHtml & renderItemTextOnlyValue
- Controls how the selected user appears on the issue view.
- Shows phonetic name if it exists, otherwise falls back to displayName.
- Clean, simple HTML/text output.

---

❤️ Made with ❤️ for the Jira community
Star this repository if it saved you time!
Contributions, issues, and feature requests are welcome.
Happy scripting! 🎉
