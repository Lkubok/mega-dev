<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // The desk's own view: counted where they're filed (379 under parser), the oneDayMove reports are
  // footnoted and also listed as their own problem.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const problemOf = (r: Report, p: string) =>
    p === 'one-day' ? notes.oneDayMove.includes(r.id) : (causeOf(r, tenantOf(r)) ?? 'unplaced') === p;
  const rows = $derived(
    ['parser', 'hint', 'one-day', 'unplaced']
      .map((p) => {
        const rs = merged.filter((r) => problemOf(r, p));
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        return { p, n: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, first: days[0], last: days.at(-1) };
      })
      .filter((x) => x.n)
  );
</script>

<p><strong>{merged.length}</strong> reports in #{ticket} · <strong>{rows.length}</strong> problems (the one-day reports are also counted under parser)</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th></tr></thead>
  <tbody>
    {#each rows as x (x.p)}
      <tr><td><code>{x.p}</code></td><td>{x.n}</td><td>{x.tenants}</td><td>{x.first} → {x.last}</td></tr>
    {/each}
  </tbody>
</table>
