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
