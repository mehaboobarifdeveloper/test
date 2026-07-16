```
pipeline {
    agent {
        ecs {
            inheritFrom 'nvs-core'
            image 'artifactory'
        }
    }

    options {
        timeout(time: 15, unit: 'MINUTES')
        timestamps()
    }

    environment {
        // Same corporate proxy as the real pipeline - if this is missing/wrong, the
        // "Install Playwright browsers" stage below is where it'll show up.
        http_proxy  = "Y"
        https_proxy = "Y"
        HTTP_PROXY  = "Y"
        HTTPS_PROXY = "Y"
        NO_PROXY    = "N"
        no_proxy    = "${NO_PROXY}"
    }

    stages {
        stage('Agent info') {
            steps {
                sh '''
                echo "== Confirms the ECS agent came up at all =="
                uname -a || true
                cat /etc/os-release || true
                echo "-- pre-existing tooling on this image --"
                command -v node && node --version || echo "node: NOT present on core image"
                command -v npm  && npm --version  || echo "npm: NOT present on core image"
                command -v curl && curl --version | head -1 || echo "curl: NOT present"
                '''
            }
        }

        stage('Install Node.js (if missing)') {
            steps {
                sh '''
                if command -v node >/dev/null 2>&1; then
                  echo "Node already present, skipping install."
                  exit 0
                fi
                echo "Node not present - attempting install on top of core image."
                (command -v apt-get >/dev/null && apt-get update && apt-get install -y --no-install-recommends curl ca-certificates gnupg \
                    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs) \
                  || (command -v yum >/dev/null && yum install -y curl \
                    && curl -fsSL https://rpm.nodesource.com/setup_20.x | bash - && yum install -y nodejs) \
                  || { echo "FAIL: could not install Node.js on this agent/image - no apt-get or yum, or install failed."; exit 1; }
                node --version
                npm --version
                '''
            }
        }

        stage('Install Playwright + browser') {
            steps {
                sh '''
                mkdir -p pw-agent-smoketest && cd pw-agent-smoketest
                npm init -y >/dev/null
                echo "Installing @playwright/test - this needs npm registry access through the proxy."
                npm install --no-save @playwright/test
                echo "Installing the Chromium browser binary + OS deps - this is the step most\n                     likely to fail/hang if the proxy or NO_PROXY list is wrong for this agent."
                npx playwright install --with-deps chromium
                '''
            }
        }

        stage('Launch a real browser') {
            steps {
                sh '''
                cd pw-agent-smoketest
                cat > smoke.spec.js <<'EOF'
const { test, expect } = require('@playwright/test');
test('agent can launch a browser and load a page', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});
EOF
                npx playwright test smoke.spec.js
                '''
            }
        }
    }

    post {
        success {
            echo "RESULT: PASS - the ECS agent can install and run Playwright (Node install + proxy + browser launch all worked)."
        }
        failure {
            echo "RESULT: FAIL - check which stage above failed: Agent info (agent/image itself broken), Node install (image can't get Node), Install Playwright + browser (registry/proxy/CDN blocked), or Launch a real browser (browser launched but couldn't reach the network, or missing OS-level browser deps that --with-deps couldn't install)."
        }
    }
}
```
