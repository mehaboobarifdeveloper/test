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
web_dbf4940552c4b0ffad6844c791008ce1e1d9d9506e44392c8ec1913243c5e03c
