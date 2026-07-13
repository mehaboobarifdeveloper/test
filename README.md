```
import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test('homepage snapshot', async ({ page }) => {
  await page.goto('https://example.com');
  await percySnapshot(page, 'Homepage');
});

test('responsive snapshot', async ({ page }) => {
  await page.goto('https://example.com/pricing');
  await percySnapshot(page, 'Pricing Page', {
    widths: [375, 768, 1280],
  });
});

```
# macOS/Linux
export PERCY_TOKEN=<your_token>
# Windows PowerShell
$env:PERCY_TOKEN="<your_token>"

npx percy exec -- npx playwright test





========================
Latest
=======================
That's the same symptom as before: the `AggregateError` from `node:net` means the SDK tried to connect to `localhost:5338` and nothing was listening — the Percy server never came up (or died before tests started).

Run through this checklist in order:

**1. Clear any stale state:**

```powershell
Get-NetTCPConnection -LocalPort 5338 -ErrorAction SilentlyContinue | Select-Object OwningProcess
# if it returns a PID:
Stop-Process -Id <pid> -Force
Remove-Item C:\Users\hero\.percy\agent-5338.lock -ErrorAction SilentlyContinue
```

**2. Verify the browser override is valid** (a bad path makes Percy fail at launch, exactly like the hang):

```powershell
echo $env:PERCY_BROWSER_EXECUTABLE
Test-Path $env:PERCY_BROWSER_EXECUTABLE   # must be True; if False, fix it or unset:
Remove-Item Env:PERCY_BROWSER_EXECUTABLE
```

**3. Confirm token is still set** (new terminal = gone):

```powershell
echo $env:PERCY_TOKEN
```

**4. Rerun and watch the lines BEFORE the tests start:**

```powershell
npx percy exec --verbose -- npx playwright test
```

The verdict is always in the pre-test output. You must see `Percy has started!` before `Running "npx playwright test"`. If instead you see an error there (port in use, browser launch failure, skipping visual tests), that's the real problem — everything after ("Percy is not running") is just fallout. Paste those pre-test lines if it fails again.



import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test('homepage snapshot', async ({ page }) => {
  await page.goto('https://example.com');
  await percySnapshot(page, 'Homepage');
});

test('responsive snapshot', async ({ page }) => {
  await page.goto('https://example.com/pricing');
  await percySnapshot(page, 'Pricing Page', {
    widths: [375, 768, 1280],
  });
});import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test('homepage snapshot', async ({ page }) => {
  await page.goto('https://example.com');
  await percySnapshot(page, 'Homepage');
});

test('responsive snapshot', async ({ page }) => {
  await page.goto('https://example.com/pricing');
  await percySnapshot(page, 'Pricing Page', {
    widths: [375, 768, 1280],
  });
});
