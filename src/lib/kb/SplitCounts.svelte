<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  // The desk read these as a one-day move, a third problem: they leave the group the tenant put them in.
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const groups = $derived(groupByCause(rest, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  const rows = $derived([
    { problem: 'Parser reads dates month first', cause: 'parser', n: count('parser') },
    { problem: 'Booking-v3 form hints MM/DD/YYYY', cause: 'hint', n: count('hint') },
    { problem: 'One-day shift (cause not yet found)', cause: 'one-day', n: moved.length },
  ]);
</script>

<p><strong>{merged.length}</strong> reports in #{ticket}, <strong>{rows.length}</strong> problems.</p>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as row (row.cause)}
      <tr><td>{row.problem}</td><td>{row.n}</td></tr>
    {/each}
  </tbody>
  <tfoot><tr><td>total</td><td>{rows.reduce((s, r) => s + r.n, 0)}</td></tr></tfoot>
</table>
