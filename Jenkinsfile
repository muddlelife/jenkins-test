pipeline {
    agent { label 'rack' }

    options {
        timestamps()
        skipDefaultCheckout(true)
        timeout(time: 5, unit: 'MINUTES')
    }

    stages {
        stage('Test credential masking') {
            steps {
                // Dummy credential test: Jenkins should mask the bound secret in logs.
                withCredentials([string(credentialsId: 'key', variable: 'API_KEY')]) {
                    sh(label: 'Test Secret text masking', script: '''#!/bin/sh
set +x
set -eu
printf 'API_KEY=%s\\n' "$API_KEY"
echo "$API_KEY"
''')
                }
            }
        }

        stage('Inspect agent environment') {
            steps {
                sh(label: 'Show agent and tool paths', script: '''#!/bin/sh
set -u

printf '\\n=== Agent ===\\n'
printf 'Node: %s\\n' "${NODE_NAME:-unknown}"
printf 'Workspace: %s\\n' "${WORKSPACE:-unknown}"
printf 'User: '; id -un
printf 'System: '; uname -a
printf 'Working directory: '; pwd

printf '\\n=== Search paths ===\\n'
printf 'PATH=%s\\n' "$PATH"
printf 'JAVA_HOME=%s\\n' "${JAVA_HOME:-not set}"
printf 'MAVEN_HOME=%s\\n' "${MAVEN_HOME:-not set}"
printf 'M2_HOME=%s\\n' "${M2_HOME:-not set}"
printf 'GOROOT=%s\\n' "${GOROOT:-not set}"
printf 'GOPATH=%s\\n' "${GOPATH:-not set}"
printf 'VIRTUAL_ENV=%s\\n' "${VIRTUAL_ENV:-not set}"

inspect_tool() {
    tool="$1"
    shift
    printf '\\n--- %s ---\\n' "$tool"
    if command -v "$tool" >/dev/null 2>&1; then
        printf 'Path: '
        command -v "$tool"
        "$tool" "$@" 2>&1 || printf 'WARNING: %s version check failed\\n' "$tool"
    else
        printf 'NOT FOUND in PATH: %s\\n' "$tool"
    fi
}

printf '\\n=== Tools ===\\n'
inspect_tool python --version
inspect_tool python3 --version
inspect_tool pip --version
inspect_tool pip3 --version
inspect_tool java -version
inspect_tool javac -version
inspect_tool go version
inspect_tool mvn --version
inspect_tool make --version
inspect_tool git --version
inspect_tool docker --version
inspect_tool node --version
inspect_tool npm --version

printf '\\n=== Inspection complete ===\\n'
''')
            }
        }
    }
}
