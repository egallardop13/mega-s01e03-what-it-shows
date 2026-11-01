<script lang="ts">
  // How often intake's guess (meta.intakeTag) matched the cause the tenant directory gives.
  import reports from '#lib/data/reports-4471.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived.by(() => {
    const n: Record<string, number> = {};
    for (const r of merged) {
      const key = `${r.meta.intakeTag} → ${causeOf(r, tenantOf(r)) ?? 'unplaced'}`;
      n[key] = (n[key] ?? 0) + 1;
    }
    return Object.entries(n).sort((a, b) => b[1] - a[1]);
  });
  const wrong = $derived(merged.filter((r) => r.meta.intakeTag !== (causeOf(r, tenantOf(r)) ?? 'unplaced')).length);
  const exported = (reports as Report[]).length;
</script>

<p class="sum">
  <strong>{exported}</strong> rows in the export · <strong>{merged.length}</strong> reports once ids are deduplicated ·
  intake tag wrong on <strong>{wrong}</strong> of them
</p>

<table>
  <thead><tr><th>intake tagged → cause by tenant</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as [key, n] (key)}
      <tr class:wrong={key.split(' → ')[0] !== key.split(' → ')[1]}><td><code>{key}</code></td><td>{n}</td></tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1rem; }
  .wrong td { background: var(--warn-bg); }
</style>
