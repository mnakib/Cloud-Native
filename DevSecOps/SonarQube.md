## What is SonarQube?

SonarQube is a leading, open-source static code analysis platform used to continuously inspect and measure source code quality and security. It identifies bugs, security vulnerabilities, and code smells across 35+ programming languages, supporting CI/CD integration to ensure reliable, maintainable code before production.

Key features and benefits of SonarQube include:

- __Static Code Analysis:__ It automatically analyzes source code without executing it to identify bugs, security vulnerabilities (like SQL injection), and "code smells" (maintainability issues).

- __CI/CD Integration:__ SonarQube plugs into pipelines (Jenkins, GitHub Actions, GitLab, etc.) to automatically check code at every commit, branch, and pull request.

- __Quality Gates:__ It defines actionable, customizable rules (Quality Profiles) and thresholds (Quality Gates) to enforce coding standards and prevent poor-quality code from being merged.

- __Technical Debt Management:__ The tool provides detailed reports, tracking code duplication, complexity, test coverage, and the overall technical debt of a project.

- __Supported Languages:__ It works with over 35 languages including Java, Python, JavaScript, C#, C++, Go, and TypeScript.

- __Deployment Options:__ Available as a self-managed server (SonarQube Server) or a cloud-based solution (SonarQube Cloud).

SonarQube is developed by [SonarSource](https://www.sonarsource.com/products/sonarqube/) and acts as a "shift-left" tool, enabling developers to catch issues early in the development lifecycle.




## Phase 1: Prepare the Python Code

### Prepare the environment
```bash
# Create a working directory
mkdir sonar-demo && cd sonar-demo

# Create & activate a Python virtual environment
python3 -m venv sonarqube
source venv/bin/activate

# Install `pysonar` package
pip3 install pysonar
```

### Create the Python code

```bash
cat > app.py
```
```python
import os

# VULNERABILITY: Hardcoded sensitive information
API_KEY = "12345-ABCDE-SECRET-KEY"

def run_system_command(user_input):
    # VULNERABILITY: Command Injection (using os.system with raw input)
    os.system("echo " + user_input)

if __name__ == "__main__":
    print("Starting app...")
    run_system_command("Hello World")
```



## Phase 2: Run the SonarQube Container

```bash
# Pull the SonarQube image
podman pull sonarqube

# Run the container  
podman run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
    sonarqube
```

Breakdown of the flags:
-d: Runs the container in "detached" mode (background).

-p 9000:9000: Maps port 9000 of the container to your Mac’s port 9000.

-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true: Critical for Mac/Local dev. It prevents the embedded Elasticsearch from failing due to strict production-level system checks (like vm.max_map_count) that are hard to change on macOS.

Access it: Open your browser to `http://localhost:9000`.

Default Username: admin

Default Password: admin (You will be prompted to change the password immediately.)



## Phase 3 - Run a Project Analysis

### Create the Project

Open your SonarQube console on `http://localhost:9000`.

Go to Projects > Create Project > Local Project

Provide the required information:

Project display name: Python Demo Project
Project key: python-demo-project
Main branch name: main

Click on Next.

Set up new code for project: Follows the instance's default - Current default: Previous version
_In SonarQube and SonarCloud, the New Code Definition acts like a "quality filter" for your project. Instead of worrying about every error in a massive codebase built over years, it tells the system to only alert you about problems in the code you just wrote or changed. 
DEV Community
DEV Community
 +2
This follows the Clean as You Code methodology: if you keep every new change clean, the overall quality of your project naturally improves over time._

Click on Create project


### Analyze the Project

Analysis Method

How do you want to analyze your repository?: Locally

Provide a token: Generate > Continue

Run analysis on your project: Python

Install the Scanner for Python projects

```bash
pip install pysonar
```

Execute the Scanner

```bash
pysonar \
  --sonar-host-url=http://localhost:9000 \
  --sonar-token=sqp_8976d1e4010b8b0c9b03105832ac936541676328 \
  --sonar-project-key=Python-Demo-Project
```

If the command is run successfully, you will see something similar to below output.

```bash
INFO: More about the report processing at http://localhost:9000/api/ce/task?id=e5a73975-a061-427f-a49a-3b845a4faae7
INFO: Analysis total time: 11.779 s
INFO: SonarScanner Engine completed successfully
the scan report will
```

And the scan report will show up automatically on the console.



