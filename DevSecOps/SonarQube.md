


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



