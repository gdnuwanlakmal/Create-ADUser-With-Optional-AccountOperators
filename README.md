# Create ADUser With Optional AccountOperators
PowerShell script to create Active Directory users interactively with an optional Account Operators role for helpdesk scenarios.
# Create-ADUser-With-Optional-AccountOperators

This PowerShell script provides an interactive way to create users in Active Directory
and optionally add them to the **Account Operators** group for helpdesk use.

## Features
- Interactive prompts for:
  - First Name
  - Last Name
  - Username (SamAccountName)
  - Password (secure)
- Creates users in a predefined OU
- Optional **YES / NO** prompt to add the user to **Account Operators**
- Prevents duplicate user creation
- Uses Microsoft-supported Active Directory cmdlets

## Use Cases
- Creating normal domain users
- Creating helpdesk users who need to:
  - Reset passwords
  - Enable / disable accounts

## Requirements
- Windows Server or workstation with RSAT installed
- PowerShell 5.1+
- ActiveDirectory module
- Domain Admin privileges (to add Account Operators)

## How to Use
1. Clone or download this repository
2. Open PowerShell **as Administrator**
3. Run the script:

```powershell
.\Create-ADUser-With-Optional-AccountOperators.ps1

```shell
<#
.SYNOPSIS
Creates an Active Directory user with an option to add the user to Account Operators.

.DESCRIPTION
This script interactively creates a new Active Directory user in a specified OU.
After creation, it prompts whether the user should be added to the Account Operators
group for helpdesk password reset and account enable/disable tasks.

.AUTHOR
Nuwan Gamage

.VERSION
1.0

.CREATED
2025-12-16

.NOTES
Run as Domain Admin.
Designed for Microsoft Active Directory environments.
#>

Import-Module ActiveDirectory

# =====================================================
# CONFIGURATION
# =====================================================

# Domain UPN suffix
$DomainUPN = "domain.local"

# OU where users will be created
$UserOU = "Replace with OU - DistinguishedName"

# Built-in group for helpdesk operations
$AccountOperatorsGroup = "Account Operators"

Write-Host ""
Write-Host "=== ACTIVE DIRECTORY USER CREATION ===" -ForegroundColor Cyan
Write-Host ""

# =====================================================
# USER INPUT
# =====================================================

$FirstName = Read-Host "Enter First Name"
$LastName  = Read-Host "Enter Last Name"
$Sam       = Read-Host "Enter Username (SamAccountName)"
$Password  = Read-Host "Enter Password" -AsSecureString

$DisplayName = "$FirstName $LastName"
$UPN         = "$Sam@$DomainUPN"

# =====================================================
# CHECK IF USER EXISTS
# =====================================================

if (Get-ADUser -Filter "SamAccountName -eq '$Sam'" -ErrorAction SilentlyContinue) {
    Write-Host ""
    Write-Host "ERROR: User '$Sam' already exists. Aborting." -ForegroundColor Red
    exit
}

# =====================================================
# CREATE USER
# =====================================================

New-ADUser `
    -Name $DisplayName `
    -GivenName $FirstName `
    -Surname $LastName `
    -SamAccountName $Sam `
    -UserPrincipalName $UPN `
    -Path $UserOU `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true

Write-Host ""
Write-Host "User created successfully." -ForegroundColor Green

# =====================================================
# OPTIONAL ACCOUNT OPERATORS ROLE
# =====================================================

$AddToAO = Read-Host "Add this user to Account Operators? (YES / NO)"

if ($AddToAO -match '^(YES|Y)$') {

    Add-ADGroupMember $AccountOperatorsGroup $Sam
    Write-Host "User added to Account Operators." -ForegroundColor Yellow

} else {

    Write-Host "User NOT added to Account Operators (normal user)." -ForegroundColor Cyan
}

# =====================================================
# SUMMARY
# =====================================================

Write-Host ""
Write-Host "=== SUMMARY ===" -ForegroundColor Cyan
Write-Host "Name     : $DisplayName"
Write-Host "Username : $Sam"
Write-Host "OU       : $UserOU"

if ($AddToAO -match '^(YES|Y)$') {
    Write-Host "Role     : Helpdesk (Account Operators)" -ForegroundColor Yellow
} else {
    Write-Host "Role     : Normal User" -ForegroundColor Green
}

Write-Host ""
Write-Host "Script completed successfully." -ForegroundColor Green
```
