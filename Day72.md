# Day 72: Jenkins Parameterized Job with Multiple Parameter Types

## 📋 Task Overview

**Scenario:** A new DevOps Engineer has joined the Nautilus team and will be working on Jenkins automation. Before assigning complex tasks, the team wants to test basic parameterized build functionality. You need to create a simple job with multiple parameter types to demonstrate how parameters work in Jenkins.

**Given Requirements:**
- **Jenkins UI Access:** Click Jenkins button on top bar
- **Login Credentials:**
  - Username: `admin`
  - Password: `Adm!n321`
- **Job Configuration:**
  - Job Name: `parameterized-job`
  - Job Type: Freestyle project
  - **String Parameter:**
    - Name: `Stage`
    - Default Value: `Build`
  - **Choice Parameter:**
    - Name: `env`
    - Choices: `Development`, `Staging`, `Production`
  - **Build Step:** Execute shell command that echoes both parameter values
  - **Validation:** Build at least once with `env=Production`

**Your Mission:**
1. Log in to Jenkins
2. Create new freestyle job named `parameterized-job`
3. Add String parameter `Stage` with default value `Build`
4. Add Choice parameter `env` with three environment options
5. Configure shell command to echo both parameters
6. Build job with `Production` environment to verify functionality

---

## 🎯 Learning Objectives

By the end of this task, you will understand:
- **Multiple Parameter Types:** String vs Choice parameters
- **Parameter Default Values:** Setting sensible defaults
- **Choice Parameters:** Restricting input to predefined options
- **Parameter Usage:** Accessing parameters in build steps
- **Build Validation:** Testing with different parameter combinations
- **Best Practices:** Parameter naming and documentation

---

## 📖 Understanding Jenkins Parameters

### Parameter Types Comparison

**String Parameter:**
```
Type: Free-form text input
Input Method: Text box
Validation: None (accepts any text)
Use Case: Branch names, version numbers, custom values
Example: Stage = "Build", "Test", "Deploy" (user can type anything)
```

**Choice Parameter:**
```
Type: Predefined dropdown options
Input Method: Dropdown selection
Validation: Built-in (only allows defined choices)
Use Case: Environments, regions, boolean-like choices
Example: env = Development | Staging | Production (user selects from list)
```

**Comparison:**
```
┌─────────────────┬──────────────────┬─────────────────┐
│ Feature         │ String Parameter │ Choice Parameter│
├─────────────────┼──────────────────┼─────────────────┤
│ Input Type      │ Text field       │ Dropdown        │
│ Validation      │ None             │ Restricted list │
│ Default Value   │ Optional         │ First choice    │
│ User Flexibility│ High             │ Low (controlled)│
│ Use Case        │ Open-ended input │ Fixed options   │
└─────────────────┴──────────────────┴─────────────────┘
```

---

### When to Use Each Parameter Type

**Use String Parameter when:**
- ✅ Input values are unpredictable
- ✅ Users need flexibility to enter custom values
- ✅ Examples:
  - Git branch names (feature/user-login, bugfix/issue-123)
  - Version numbers (1.2.3, 2.0.0-beta)
  - Package names (nginx, vim, git)
  - Custom messages or descriptions

**Use Choice Parameter when:**
- ✅ Input values are known and limited
- ✅ You want to prevent typos or invalid values
- ✅ Examples:
  - Environments (Development, Staging, Production)
  - Regions (us-east-1, us-west-2, eu-west-1)
  - Actions (deploy, rollback, restart)
  - Boolean-like choices (yes/no, enabled/disabled)

---

### Parameter Best Practices

**Naming Conventions:**
```bash
Good Examples:
- Stage (clear, capitalized)
- env (short, lowercase, common abbreviation)
- BRANCH_NAME (uppercase with underscore)
- deploymentRegion (camelCase)

Bad Examples:
- s (too short, unclear)
- environment_name_for_deployment (too long)
- Env (inconsistent capitalization)
- stage-name (hyphens can cause issues in some contexts)
```

**Default Values:**
```
String Parameter:
- Provide sensible default if applicable
- Leave empty if no good default exists
- Example: Stage = "Build" (most common stage)

Choice Parameter:
- First option is automatically default
- Put most common choice first
- Example: env choices = "Development, Staging, Production"
  (Development is default as most builds are for dev)
```

---

## 🛠️ Task Implementation

### Step 1: Access Jenkins and Log In

**Click the "Jenkins" button on the top bar.**

**Or navigate to:**
```
http://<jenkins-server-ip>:8080
```

**Log in with admin credentials:**
- **Username:** `admin`
- **Password:** `Adm!n321`

**Expected:** Jenkins Dashboard appears

---

### Step 2: Create New Jenkins Job

**From Jenkins Dashboard:**

1. Click **New Item** (left sidebar)
2. **Enter job name:** `parameterized-job`
3. **Select:** Freestyle project
4. Click **OK**

**You'll be redirected to job configuration page.**

---

### Step 3: Configure String Parameter

**In the job configuration page:**

1. Scroll to **General** section
2. Check the box: **☑ This project is parameterized**

**The parameter section will appear below.**

---

**Add String Parameter:**

3. Click **Add Parameter** dropdown button
4. Select **String Parameter**

**Configure the String Parameter:**
```
┌──────────────────────────────────────────────┐
│  String Parameter                             │
├──────────────────────────────────────────────┤
│                                              │
│  Name:                                       │
│  [Stage_____________________________]        │
│                                              │
│  Default Value:                              │
│  [Build_____________________________]        │
│                                              │
│  Description:                                │
│  [Build stage to execute (e.g., Build,]     │
│  [Test, Deploy)_____________________]        │
│                                              │
│  ☐ Trim the string                          │
│                                              │
└──────────────────────────────────────────────┘
```

**Fill in:**
- **Name:** `Stage`
- **Default Value:** `Build`
- **Description:** `Build stage to execute (e.g., Build, Test, Deploy)`
- **Trim the string:** Check this box (optional but recommended)

**Important:** 
- Parameter name is `Stage` (capital S)
- Default value is `Build` (capital B)
- Case-sensitive - must match exactly in shell script

---

### Step 4: Configure Choice Parameter

**Still in the General section:**

1. Click **Add Parameter** dropdown button again
2. Select **Choice Parameter**

**Configure the Choice Parameter:**
```
┌──────────────────────────────────────────────┐
│  Choice Parameter                             │
├──────────────────────────────────────────────┤
│                                              │
│  Name:                                       │
│  [env_______________________________]        │
│                                              │
│  Choices:                                    │
│  [Development_______________________]        │
│  [Staging___________________________]        │
│  [Production________________________]        │
│  [__________________________________]        │
│                                              │
│  Description:                                │
│  [Target environment for deployment_]        │
│  [(Development/Staging/Production)__]        │
│                                              │
└──────────────────────────────────────────────┘
```

**Fill in:**
- **Name:** `env`
- **Choices:** (Enter one per line)
  ```
  Development
  Staging
  Production
  ```
- **Description:** `Target environment for deployment (Development/Staging/Production)`

**Important:**
- Parameter name is `env` (lowercase)
- Each choice on a separate line
- First choice (`Development`) becomes the default
- No quotes or commas between choices

---

**Your parameter configuration should now show:**
```
General Section:
☑ This project is parameterized

  String Parameter:
    Name: Stage
    Default Value: Build
    Description: Build stage to execute

  Choice Parameter:
    Name: env
    Choices: Development
             Staging
             Production
    Description: Target environment for deployment
```

---

### Step 5: Configure Build Step

Scroll down to the **Build** section.

1. Click **Add build step** dropdown
2. Select **Execute shell**

**Enter the following shell script:**

```bash
#!/bin/bash

echo "=========================================="
echo "Jenkins Parameterized Job Execution"
echo "=========================================="
echo ""
echo "Parameter Values:"
echo "  Stage: $Stage"
echo "  Environment: $env"
echo ""
echo "=========================================="
echo "Executing $Stage stage in $env environment"
echo "=========================================="
echo ""

# Simulate some work based on parameters
if [ "$Stage" == "Build" ]; then
    echo "📦 Building application..."
    echo "  - Compiling source code"
    echo "  - Running unit tests"
    echo "  - Creating artifacts"
elif [ "$Stage" == "Test" ]; then
    echo "🧪 Running tests..."
    echo "  - Integration tests"
    echo "  - Performance tests"
    echo "  - Security scans"
elif [ "$Stage" == "Deploy" ]; then
    echo "🚀 Deploying application..."
    echo "  - Preparing deployment package"
    echo "  - Uploading to $env"
    echo "  - Running health checks"
else
    echo "📋 Executing custom stage: $Stage"
fi

echo ""
echo "=========================================="
echo "✅ $Stage stage completed successfully in $env environment"
echo "=========================================="
```

**This script:**
- ✅ Echoes both parameter values (`$Stage` and `$env`)
- ✅ Provides visual formatting with separators
- ✅ Simulates different actions based on Stage parameter
- ✅ Shows both parameters in context
- ✅ Provides clear success message

---

**Simple version (minimal requirement):**

If you prefer a simpler script that just echoes the parameters:

```bash
#!/bin/bash
echo "Stage: $Stage"
echo "Environment: $env"
```

**This meets the requirement but provides less context.**

---

### Step 6: Save the Job Configuration

1. Scroll to the bottom of the page
2. Click **Save**

**You'll be redirected to the job page showing:**
- Job name: `parameterized-job`
- Build history (empty)
- Left sidebar with options

---

### Step 7: First Build - Test with Default Values

**Test the job with default parameter values:**

1. On the job page, click **Build with Parameters** (left sidebar)

**You'll see the parameter form:**
```
┌──────────────────────────────────────────────┐
│  Build with Parameters                        │
├──────────────────────────────────────────────┤
│                                              │
│  Stage                                       │
│  [Build_____________________________]        │
│  Build stage to execute (e.g., Build,        │
│  Test, Deploy)                               │
│                                              │
│  env                                         │
│  [Development ▼]                             │
│  Target environment for deployment           │
│                                              │
│              [Build]                         │
│                                              │
└──────────────────────────────────────────────┘
```

**Default values are already filled:**
- Stage: `Build`
- env: `Development` (first choice in dropdown)

2. Click **Build** (use default values for this test)

---

**Monitor the build:**

3. In **Build History** (left sidebar), you'll see **#1** appear
4. Click on the build number **#1**
5. Click **Console Output** to view logs

**Expected output:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/parameterized-job
[parameterized-job] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ echo ==========================================
==========================================
+ echo Jenkins Parameterized Job Execution
Jenkins Parameterized Job Execution
+ echo ==========================================
==========================================
+ echo
+ echo Parameter Values:
Parameter Values:
+ echo   Stage: Build
  Stage: Build
+ echo   Environment: Development
  Environment: Development
+ echo
+ echo ==========================================
==========================================
+ echo Executing Build stage in Development environment
Executing Build stage in Development environment
+ echo ==========================================
==========================================
+ echo
+ [ Build == Build ]
+ echo 📦 Building application...
📦 Building application...
+ echo   - Compiling source code
  - Compiling source code
+ echo   - Running unit tests
  - Running unit tests
+ echo   - Creating artifacts
  - Creating artifacts
+ echo
+ echo ==========================================
==========================================
+ echo ✅ Build stage completed successfully in Development environment
✅ Build stage completed successfully in Development environment
+ echo ==========================================
==========================================
Finished: SUCCESS
```

**Result:** Build #1 should succeed (blue ball) ✅

---

### Step 8: Build with Production Environment (Required)

**The task requires building at least once with `env=Production`:**

1. Click on the job name **parameterized-job** (breadcrumb at top)
2. Click **Build with Parameters**

**Modify parameters:**
```
┌──────────────────────────────────────────────┐
│  Build with Parameters                        │
├──────────────────────────────────────────────┤
│                                              │
│  Stage                                       │
│  [Build_____________________________]        │
│                                              │
│  env                                         │
│  [Production ▼]                              │
│                                              │
│              [Build]                         │
│                                              │
└──────────────────────────────────────────────┘
```

3. Keep Stage as: `Build`
4. Change env dropdown to: `Production`
5. Click **Build**

---

**Monitor Build #2:**

6. Click on **#2** in Build History
7. Click **Console Output**

**Expected output should show:**
```
Parameter Values:
  Stage: Build
  Environment: Production

Executing Build stage in Production environment

📦 Building application...
  - Compiling source code
  - Running unit tests
  - Creating artifacts

✅ Build stage completed successfully in Production environment

Finished: SUCCESS
```

**Result:** Build #2 should succeed with `env=Production` ✅

**This satisfies the requirement!**

---

### Step 9: Test with Different Parameter Combinations

**Test various combinations to verify flexibility:**

**Test 3: Test Stage + Staging Environment**
1. Build with Parameters
2. Stage: `Test`
3. env: `Staging`
4. Build

**Expected output:**
```
Parameter Values:
  Stage: Test
  Environment: Staging

Executing Test stage in Staging environment

🧪 Running tests...
  - Integration tests
  - Performance tests
  - Security scans

✅ Test stage completed successfully in Staging environment
```

---

**Test 4: Deploy Stage + Production Environment**
1. Build with Parameters
2. Stage: `Deploy`
3. env: `Production`
4. Build

**Expected output:**
```
Parameter Values:
  Stage: Deploy
  Environment: Production

Executing Deploy stage in Production environment

🚀 Deploying application...
  - Preparing deployment package
  - Uploading to Production
  - Running health checks

✅ Deploy stage completed successfully in Production environment
```

---

**Test 5: Custom Stage + Development**
1. Build with Parameters
2. Stage: `Validate` (custom value)
3. env: `Development`
4. Build

**Expected output:**
```
Parameter Values:
  Stage: Validate
  Environment: Development

Executing Validate stage in Development environment

📋 Executing custom stage: Validate

✅ Validate stage completed successfully in Development environment
```

**This demonstrates String parameter flexibility - users can enter any stage name!**

---

### Step 10: Verify Build History

**After multiple test runs, your build history should show:**

```
Build History:
#5  ● Custom: Validate + Development (5 sec) - SUCCESS
#4  ● Deploy + Production (6 sec) - SUCCESS
#3  ● Test + Staging (5 sec) - SUCCESS
#2  ● Build + Production (5 sec) - SUCCESS ✅ (Required)
#1  ● Build + Development (5 sec) - SUCCESS (Default)

Success Rate: 100% (5/5)
```

**All builds should show blue ball (success) ✅**

**Most important: Build #2 with Production environment is complete ✅**

---

### Step 11: Review Parameter Configuration

**Verify job is configured correctly:**

1. Go to job page: **parameterized-job**
2. Click **Configure** (left sidebar)
3. Verify **General** section shows:

```
☑ This project is parameterized

Parameters:
  1. String Parameter
     - Name: Stage
     - Default: Build
     - Description: Build stage to execute

  2. Choice Parameter
     - Name: env
     - Choices: Development, Staging, Production
     - Description: Target environment
```

4. Verify **Build** section shows shell script that echoes both parameters
5. Click **Save** (or Cancel if no changes)

---

## 📊 Understanding Build Parameters Page

**When you click "Build with Parameters", Jenkins displays:**

```
┌─────────────────────────────────────────────────┐
│  Build with Parameters                          │
├─────────────────────────────────────────────────┤
│                                                 │
│  String Parameter (Stage):                      │
│  ┌───────────────────────────────────────────┐ │
│  │ Build                                     │ │ ← Text input field
│  └───────────────────────────────────────────┘ │
│  Build stage to execute (e.g., Build,           │
│  Test, Deploy)                                  │
│                                                 │
│  Choice Parameter (env):                        │
│  ┌───────────────────────────────────────────┐ │
│  │ Development                            ▼  │ │ ← Dropdown menu
│  └───────────────────────────────────────────┘ │
│  Target environment for deployment              │
│                                                 │
│  [Build]  [Cancel]                              │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Interaction:**
- **Stage field:** Users can type any value (free-form)
- **env dropdown:** Users select from predefined list (Development, Staging, Production)
- **Build button:** Submits form and starts build with specified parameters
- **Cancel button:** Returns to job page without building

---

## 📊 Complete Shell Script with Enhanced Features

**Enhanced version with additional logging and validation:**

```bash
#!/bin/bash

#############################################
# Jenkins Parameterized Job
# Job Name: parameterized-job
# Parameters: Stage (String), env (Choice)
#############################################

# Color codes for better visibility (if terminal supports)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to print headers
print_header() {
    echo "=========================================="
    echo "$1"
    echo "=========================================="
}

# Function to print info
print_info() {
    echo "${BLUE}ℹ${NC}  $1"
}

# Function to print success
print_success() {
    echo "${GREEN}✓${NC}  $1"
}

# Start
print_header "Jenkins Parameterized Job Execution"
echo ""

# Display parameters
echo "📋 Build Parameters:"
echo "   Stage: ${GREEN}$Stage${NC}"
echo "   Environment: ${GREEN}$env${NC}"
echo ""

# Validate parameters (optional but good practice)
if [ -z "$Stage" ]; then
    echo "${RED}ERROR: Stage parameter is empty${NC}"
    exit 1
fi

if [ -z "$env" ]; then
    echo "${RED}ERROR: env parameter is empty${NC}"
    exit 1
fi

# Display timestamp
echo "🕐 Build Time: $(date '+%Y-%m-%d %H:%M:%S')"
echo "👤 Build User: ${BUILD_USER:-admin}"
echo "🏗️  Build Number: #${BUILD_NUMBER}"
echo ""

print_header "Executing $Stage Stage"

# Execute based on Stage parameter
case "$Stage" in
    "Build")
        print_info "Starting Build stage..."
        echo ""
        echo "   📦 Compiling source code"
        sleep 1
        echo "   🧪 Running unit tests"
        sleep 1
        echo "   📂 Creating artifacts"
        sleep 1
        echo "   📊 Generating build report"
        echo ""
        print_success "Build completed successfully"
        ;;
        
    "Test")
        print_info "Starting Test stage..."
        echo ""
        echo "   🧪 Running integration tests"
        sleep 1
        echo "   ⚡ Performance testing"
        sleep 1
        echo "   🔒 Security scanning"
        sleep 1
        echo "   📊 Generating test report"
        echo ""
        print_success "Tests passed successfully"
        ;;
        
    "Deploy")
        print_info "Starting Deploy stage..."
        echo ""
        echo "   📦 Preparing deployment package"
        sleep 1
        echo "   🚀 Deploying to $env environment"
        sleep 1
        echo "   🏥 Running health checks"
        sleep 1
        echo "   ✉️  Sending deployment notification"
        echo ""
        print_success "Deployment completed successfully"
        ;;
        
    *)
        print_info "Executing custom stage: $Stage"
        echo ""
        echo "   📋 Custom stage logic here"
        echo "   🔧 Processing custom commands"
        echo ""
        print_success "Custom stage completed"
        ;;
esac

echo ""
print_header "Environment: $env"

# Environment-specific information
case "$env" in
    "Development")
        echo "   🔧 Environment: Development"
        echo "   📍 Region: dev-us-east-1"
        echo "   🌐 URL: https://dev.example.com"
        echo "   👥 Audience: Developers"
        ;;
        
    "Staging")
        echo "   🎭 Environment: Staging"
        echo "   📍 Region: staging-us-east-1"
        echo "   🌐 URL: https://staging.example.com"
        echo "   👥 Audience: QA Team"
        ;;
        
    "Production")
        echo "   🚀 Environment: Production"
        echo "   📍 Region: prod-us-east-1"
        echo "   🌐 URL: https://www.example.com"
        echo "   👥 Audience: End Users"
        echo "   ⚠️  WARNING: Production deployment!"
        ;;
esac

echo ""
print_header "Summary"

echo "   Stage: $Stage"
echo "   Environment: $env"
echo "   Status: ${GREEN}SUCCESS${NC}"
echo "   Duration: ${BUILD_DURATION:-N/A}"
echo ""

print_success "Job completed successfully!"
echo ""
print_header "End of Build"
```

**This enhanced script provides:**
- ✅ Color-coded output (if terminal supports)
- ✅ Parameter validation
- ✅ Build metadata (time, user, build number)
- ✅ Case statement for different stages
- ✅ Environment-specific details
- ✅ Professional formatting
- ✅ Sleep commands to simulate work
- ✅ Comprehensive summary

---

## 🧪 Testing Scenarios

### Test Matrix

**Test all combinations systematically:**

```
┌───────────┬─────────────┬──────────┬────────────────┐
│ Test #    │ Stage       │ env      │ Expected       │
├───────────┼─────────────┼──────────┼────────────────┤
│ 1         │ Build       │ Dev      │ Build + Dev    │
│ 2         │ Build       │ Staging  │ Build + Stg    │
│ 3         │ Build       │ Prod     │ Build + Prod ✅│
│ 4         │ Test        │ Dev      │ Test + Dev     │
│ 5         │ Test        │ Staging  │ Test + Stg     │
│ 6         │ Test        │ Prod     │ Test + Prod    │
│ 7         │ Deploy      │ Dev      │ Deploy + Dev   │
│ 8         │ Deploy      │ Staging  │ Deploy + Stg   │
│ 9         │ Deploy      │ Prod     │ Deploy + Prod  │
│ 10        │ (Custom)    │ Any      │ Custom output  │
└───────────┴─────────────┴──────────┴────────────────┘
```

**Test #3 (Build + Production) is REQUIRED ✅**

---

### Verification Checklist

**For each build, verify:**

✅ **Parameter Display:**
- Console output shows: `Stage: <value>`
- Console output shows: `Environment: <value>`

✅ **Build Success:**
- Build completes with blue ball
- No errors in console output
- "Finished: SUCCESS" message appears

✅ **Parameter Passing:**
- Correct Stage value is used
- Correct env value is used
- Shell script receives both parameters

✅ **Build History:**
- Each build is numbered sequentially
- Parameters are visible in build history
- Builds can be differentiated by parameters

---

## 🐛 Common Issues and Solutions

### Issue 1: Parameters Not Showing in Build Form

**Symptoms:**
- Click "Build Now" instead of "Build with Parameters"
- No parameter form appears
- Job builds with default values automatically

**Diagnosis:**
- Job is not configured as parameterized
- "This project is parameterized" checkbox not checked

**Solution:**

```
1. Go to job configuration
2. Check "General" section
3. Ensure "This project is parameterized" is checked ☑
4. Add parameters if missing
5. Save job configuration
6. Refresh job page
7. "Build with Parameters" option should appear
```

---

### Issue 2: Parameter Values Not Appearing in Console Output

**Symptoms:**
```
Console output shows:
  Stage:
  Environment:
(Values are empty)
```

**Diagnosis:**
- Variable names in script don't match parameter names
- Case sensitivity issue
- Shell syntax error

**Solution:**

```bash
# Check parameter names match exactly
Parameter name in config: Stage (capital S)
Variable in script: $Stage (capital S)

Parameter name in config: env (lowercase)
Variable in script: $env (lowercase)

# Correct usage in shell:
echo "Stage: $Stage"
echo "Environment: $env"

# Wrong (will be empty):
echo "Stage: $stage"  # Wrong case
echo "Environment: $ENV"  # Wrong case
```

---

### Issue 3: Choice Parameter Shows All Options in One Line

**Symptoms:**
```
Choices field shows:
Development, Staging, Production
(All on one line with commas)
```

**Diagnosis:**
- Entered choices incorrectly
- Should be one per line, not comma-separated

**Solution:**

```
Wrong:
┌─────────────────────────────────────┐
│ Development, Staging, Production    │
└─────────────────────────────────────┘

Correct:
┌─────────────────────────────────────┐
│ Development                         │
│ Staging                             │
│ Production                          │
└─────────────────────────────────────┘

Steps:
1. Edit job configuration
2. Find Choice Parameter section
3. Clear Choices field
4. Enter each choice on separate line
5. Press Enter after each choice
6. Save
```

---

### Issue 4: Build Always Uses Default Values

**Symptoms:**
- Changed parameters in build form
- But console output shows default values
- Parameters not being passed to build

**Diagnosis:**
- Old build definition cached
- Browser cache issue
- Need to refresh

**Solution:**

```
1. Clear browser cache
2. Refresh Jenkins page (F5 or Ctrl+R)
3. Log out and log back in
4. Try building again with parameters
5. Verify console output shows correct values

If still not working:
1. Delete job
2. Recreate with same configuration
3. Test again
```

---

### Issue 5: Shell Script Syntax Error

**Symptoms:**
```
Console output shows:
/tmp/jenkins123.sh: line 10: syntax error near unexpected token `fi'
Build failed
```

**Diagnosis:**
- Shell script has syntax error
- Missing quotes, brackets, or keywords
- Incorrect if/case statement structure

**Solution:**

```bash
# Test script locally first
bash -n script.sh  # Check syntax without running

# Common issues:
# 1. Missing 'then' after if
if [ "$Stage" == "Build" ]  # Missing 'then'
    echo "Building"
fi

# Correct:
if [ "$Stage" == "Build" ]; then
    echo "Building"
fi

# 2. Missing 'esac' after case
case "$Stage" in
    "Build") echo "Building" ;;
# Missing 'esac'

# Correct:
case "$Stage" in
    "Build") echo "Building" ;;
esac

# 3. Quote variables to handle spaces
echo Value: $Stage  # May break if Stage has spaces
echo "Value: $Stage"  # Correct
```

---

### Issue 6: Cannot Find "Build with Parameters" Option

**Symptoms:**
- Only see "Build Now" in left sidebar
- No parameter form available
- Job configured with parameters but option missing

**Diagnosis:**
- Parameters not saved correctly
- Need to refresh page
- Jenkins UI cache issue

**Solution:**

```
1. Go to job configuration
2. Verify parameters are configured
3. Click "Apply" button (not just Save)
4. Click "Save"
5. Return to job page
6. Refresh browser (F5)
7. "Build with Parameters" should appear

If still missing:
1. Check Jenkins logs for errors
2. Restart Jenkins: Manage Jenkins → Reload Configuration
3. Re-add parameters and save again
```

---

## 📖 Key Takeaways

### Concepts Mastered Today:

1. ✅ **Multiple Parameter Types**
   - String parameters for free-form input
   - Choice parameters for restricted options
   - Combining different parameter types
   - Understanding when to use each type

2. ✅ **Parameter Configuration**
   - Setting parameter names and defaults
   - Adding descriptions for clarity
   - Configuring choice options
   - Best practices for parameter design

3. ✅ **Build Step Integration**
   - Accessing parameters in shell scripts
   - Using variables ($Stage, $env)
   - Conditional logic based on parameters
   - Echoing parameter values for verification

4. ✅ **Testing and Validation**
   - Building with different parameter combinations
   - Verifying parameter values in console output
   - Testing required combinations (Production)
   - Reviewing build history

5. ✅ **Job Usability**
   - Creating user-friendly parameter forms
   - Adding helpful descriptions
   - Setting sensible defaults
   - Professional output formatting

---

## 🎯 Real-World Applications

### Production Use Cases

**1. Environment-Specific Deployments**
```yaml
Job: deploy-application
Parameters:
  - Stage: String (Deploy, Rollback, Verify)
  - env: Choice (Dev, Staging, Prod)
  - version: String (1.2.3)

Use Case: Deploy different versions to different environments
```

**2. Multi-Region Deployments**
```yaml
Job: deploy-to-region
Parameters:
  - env: Choice (Development, Staging, Production)
  - region: Choice (us-east-1, us-west-2, eu-west-1)
  - app_name: String

Use Case: Deploy applications across multiple AWS regions
```

**3. Build and Test Pipeline**
```yaml
Job: ci-pipeline
Parameters:
  - Stage: Choice (Build, Test, Package, Publish)
  - branch: String (main, develop, feature/*)
  - run_tests: Boolean (true/false)

Use Case: Flexible CI pipeline with optional stages
```

**4. Database Operations**
```yaml
Job: database-operations
Parameters:
  - operation: Choice (backup, restore, migrate, cleanup)
  - env: Choice (Development, Staging, Production)
  - database_name: String

Use Case: Manage database operations across environments
```

---

### Enterprise Patterns

**Pattern 1: Environment Promotion Pipeline**
```
Job 1: Build (env=Development)
   ↓
Job 2: Test (env=Development)
   ↓
Job 3: Deploy (env=Staging)
   ↓
Job 4: Test (env=Staging)
   ↓
Job 5: Deploy (env=Production) ← Manual approval
```

**Pattern 2: Feature Branch Workflow**
```yaml
Job: feature-branch-build
Parameters:
  - branch: String (feature name)
  - stage: Choice (Build, Test, Deploy)
  - target_env: Choice (Dev, Feature-Env)

Workflow:
1. Developer enters branch name
2. Selects stage to execute
3. Chooses target environment
4. Jenkins builds and deploys
```

**Pattern 3: Microservices Deployment**
```yaml
Job: deploy-microservice
Parameters:
  - service: Choice (auth-service, user-service, payment-service)
  - env: Choice (Dev, Staging, Prod)
  - version: String
  - action: Choice (deploy, rollback, restart)

Use Case: Independent deployment of microservices
```

---

## 📚 Additional Resources

**Official Documentation:**
- [Jenkins Parameterized Builds](https://www.jenkins.io/doc/book/pipeline/syntax/#parameters)
- [Build Parameters Plugin](https://plugins.jenkins.io/build-parameters/)
- [Extended Choice Parameter Plugin](https://plugins.jenkins.io/extended-choice-parameter/)

**Parameter Types:**
- [String Parameter](https://www.jenkins.io/doc/book/pipeline/syntax/#string)
- [Choice Parameter](https://www.jenkins.io/doc/book/pipeline/syntax/#choice)
- [Boolean Parameter](https://www.jenkins.io/doc/book/pipeline/syntax/#booleanparam)

**Best Practices:**
- [Parameterized Build Best Practices](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#parameters)
- [Jenkins Job Design](https://www.jenkins.io/doc/book/pipeline/getting-started/)

**Advanced Topics:**
- [Active Choices Plugin](https://plugins.jenkins.io/uno-choice/) (Dynamic parameters)
- [Pipeline Parameter Reference](https://www.jenkins.io/doc/pipeline/steps/params/)

**Next Steps:**
- **Day 73:** Jenkins Pipeline as Code (Jenkinsfile)
- **Day 74:** Jenkins Shared Libraries
- **Day 75:** Jenkins Multi-Branch Pipeline
- **Day 76:** Jenkins + GitHub Webhook Integration

---

## ✅ Task Completion Checklist

**Job Creation:**
- [ ] Logged into Jenkins (admin / Adm!n321)
- [ ] Created new job named `parameterized-job`
- [ ] Selected Freestyle project type
- [ ] Job created successfully

**Parameter Configuration:**
- [ ] Checked "This project is parameterized"
- [ ] Added String Parameter:
  - [ ] Name set to `Stage`
  - [ ] Default value set to `Build`
  - [ ] Description added
- [ ] Added Choice Parameter:
  - [ ] Name set to `env`
  - [ ] Choices entered (one per line):
    - [ ] Development
    - [ ] Staging
    - [ ] Production
  - [ ] Description added

**Build Step Configuration:**
- [ ] Added "Execute shell" build step
- [ ] Script echoes Stage parameter: `echo "Stage: $Stage"`
- [ ] Script echoes env parameter: `echo "Environment: $env"`
- [ ] Script saved successfully

**Testing:**
- [ ] First build executed with default values
  - [ ] Stage: Build
  - [ ] env: Development
  - [ ] Build succeeded (blue ball)
- [ ] Console output shows both parameter values
- [ ] Build #2 executed with env=Production ✅ (REQUIRED)
  - [ ] Stage: Build
  - [ ] env: Production
  - [ ] Build succeeded (blue ball)
  - [ ] Console shows "Environment: Production"

**Additional Testing (Optional but Recommended):**
- [ ] Tested with Test stage
- [ ] Tested with Deploy stage
- [ ] Tested with Staging environment
- [ ] Tested with custom Stage value
- [ ] All tests passed successfully

**Build History Verification:**
- [ ] Build history shows multiple builds
- [ ] At least one build with env=Production
- [ ] Console outputs show correct parameter values
- [ ] All builds show SUCCESS status

**Documentation:**
- [ ] Screenshots captured of:
  - [ ] Job configuration page showing parameters
  - [ ] String parameter configuration
  - [ ] Choice parameter configuration
  - [ ] Build step with shell script
  - [ ] Build with Parameters form
  - [ ] Console output showing parameter values
  - [ ] Build history with Production build
- [ ] Or screen recording created (loom.com)

---

**🎉 Congratulations!** You've successfully created a parameterized Jenkins job with multiple parameter types! You've mastered:
- Configuring String parameters for flexible input
- Configuring Choice parameters for controlled options
- Combining different parameter types in one job
- Accessing parameters in build scripts
- Testing with different parameter combinations
- Building with Production environment as required

This is a fundamental skill for creating flexible, reusable Jenkins jobs that can handle multiple scenarios with a single job configuration!

**Day 72 Status:** ✅ Complete

**Next:** Day 73 - Jenkins Pipeline as Code with Jenkinsfile! 🚀
