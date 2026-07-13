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
